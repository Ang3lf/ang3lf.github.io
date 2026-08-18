---
layout: default
title: Load Balancer on AWS
---

# Workflow SOAR de correo electrónico a incidente: arquitectura y diseño

## VISION GENERAL

El proyecto surge de la necesidad de transformar un proceso de gestión de alertas altamente dependiente de intervención manual en un flujo estructurado, escalable y orientado a la automatización.

La solución fue diseñada para procesar las alertas de forma progresiva, separando cada etapa del flujo y permitiendo validar, normalizar, analizar y clasificar la información antes de generar el ticket correspondiente.

![vision_general](/assets/images/public/capa1.svg)

> *La arquitectura presentada ha sido simplificada para preservar la confidencialidad de la infraestructura y lógica interna.*

---

## PROBLEMA

El procesamiento de alertas requería una importante intervención manual, especialmente durante las primeras etapas de revisión y clasificación.

Entre los principales desafíos se encontraban:

- Diferentes estructuras y formatos de información provenientes de las alertas.
- Necesidad de validar y normalizar los datos antes de continuar con el análisis.
- Procesamiento de grandes volúmenes de alertas.
- Dependencia de procesos manuales para casos no contemplados.
- Dificultad para priorizar los incidentes según su impacto.
- Necesidad de mantener el flujo operativo incluso cuando alguno de sus componentes presentara problemas.

---

## DESAFÍOS — PRE-SOAR

Antes de implementar la solución, el procesamiento de las alertas presentaba varios puntos de fricción:

- **Procesamiento manual:** una parte importante de las alertas requería intervención humana desde etapas tempranas.
- **Información inconsistente:** las alertas podían presentar estructuras diferentes o información que no cumplía con los formatos esperados.
- **Acoplamiento entre procesos:** un problema en una etapa podía afectar el procesamiento de las siguientes.
- **Priorización limitada:** no todas las alertas tenían el mismo nivel de criticidad, pero requerían un mecanismo consistente para determinar su prioridad.
- **Escalabilidad:** el aumento en el volumen de alertas incrementaba directamente la carga sobre los analistas.

El objetivo no era simplemente automatizar tareas, sino **crear un flujo capaz de absorber el volumen de trabajo, controlar los casos excepcionales y entregar información útil al analista.**

---

## SOLUCIÓN: ARQUITECTURA EN CAPAS

La solución se estructuró en diferentes capas, donde cada una tenía una responsabilidad específica y se comunicaba con la siguiente mediante un flujo desacoplado.

### CAPA 1 — INGESTA

Responsable de recibir las alertas provenientes de las diferentes fuentes y conservar la información original antes de iniciar su procesamiento.

Esta separación permitió establecer un punto inicial controlado para el flujo y evitar que la recepción de nuevas alertas dependiera directamente de la capacidad de procesamiento de las etapas posteriores.

---

### CAPA 2 — PARSING Y NORMALIZACIÓN

En esta etapa se procesaba la información recibida para identificar su estructura, validar los datos disponibles y transformarlos a un formato utilizable por las siguientes capas.

*Lógica*:
1. ¿Existe regla de parseo para este formato SIEM?
    * *Sí*: Extraer los campos y normalizar
    * *No*: Marcar como "inválido", mandar a revisión manual

![cap2](/assets/images/public/capa2.svg)

---

### CAPA 3 — ANÁLISIS Y CORRELACIÓN

Una vez normalizada la información, se realizaban diferentes validaciones y análisis sobre los datos disponibles.

El objetivo era obtener mayor contexto sobre cada alerta mediante la comparación de sus atributos y elementos relacionados, generando la información necesaria para determinar su nivel de criticidad.

* *Enrichment Service*: Busca este IP en threat intel, ¿es conocido como malicioso?
* *Correlation Engine*: ¿Este patrón ocurrió antes? ¿En la misma ventana temporal?
* *Pattern Matcher*: ¿Coincide con ataques conocidos?

_Una sola alerta es ruido. Correlacionar campos + time + threatintel = inteligencia._

---

### CAPA 4 — SEVERIDAD

A partir de los resultados obtenidos durante el análisis, se determinaba el nivel de severidad correspondiente.

Esta clasificación permitía diferenciar los eventos según su criticidad y establecer el tratamiento adecuado para cada caso, evitando que todas las alertas fueran procesadas con la misma prioridad.

![capa4](/assets/images/public/capa4.svg)

---

### CAPA 5 — TICKET

Finalmente, se generaba el ticket incorporando el análisis realizado y la severidad asignada.

El incidente quedaba disponible para la "**confirmación del analista**" o alertar directamente si así se necesitaba, manteniendo la intervención humana como punto de validación final cuando fuera necesario.

Dependiendo del score:

* *CRITICAL* → Inmediato + escalación automática
* *HIGH* → Cola de análisis
* *MEDIUM* → Atención no inmediata
* *LOW* → Posible falso positivo

Cada ticket contiene:

* Campos normalizados
* Threat intel adjunta
* Conclusiones del análisis automático
* Histórico de correlaciones

---

## Arquitectura Técnica Completa

```text
┌──────────────────────┐
│       ALERTAS        │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│  1. INGESTA          │
│  Recepción y         │
│  almacenamiento      │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│  2. PARSING          │
│  Y NORMALIZACIÓN     │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│  3. ANÁLISIS         │
│  Y CORRELACIÓN       │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│  4. SEVERIDAD        │
│  Clasificación       │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│  5. TICKET           │
│  Análisis +          │
│  Severidad           │
│  ↓                   │
│  Confirmación        │
│  del analista        │
└──────────────────────┘
```

## Decisiones de Diseño Clave

| Decision | Razon |
| :--- | :--- |
| Colas entre capas | Desacoplamiento. Si Analysis service cae, Parsing sigue procesando |
| Microservicios | Escalado independiente. Si hay spike de alerts, escalar Parsing sin tocar Analysis |
| Config-driven parsing | Agregar nuevas reglas sin redeployar código |
| Revision manual | Las máquinas fallan. Un análista revisa casos edge |
| Manejo de severidad dinamico | Recursos humanos finitos. Priorizar lo crítico |

## Impacto Medible

![final](/assets/images/public/Diagramas.svg)

## Lecciones Aprendidas
1. *La validación temprana es crítica*: Detectar datos corruptos en parsing ahorra 5 capas de debugging.
2. *Las colas son oro*: Un queue lento es mejor que un servicio caído.
3. *Configuración y Código*: Permitir a analistas agregar reglas reduce dependencia en desarrollo y agregación.
4. *Segmentación*: Si un pod/servicio/microservicio cae no repercute en los demás.