---
layout: default
title: SOAR/UI
---

# SOAR Incident Management UI: Real-Time Dashboard

## Visión General

Consola unificada donde analistas y supervisores visualizan, entienden y actúan sobre incidentes que procesa el SOAR, con autenticación corporativa y auditoría de cada acción.

---

## El Problema

Sin una UI, los problemas que se enfrentaban:

* Sin control de eventos procesados
* Agregar reglas de parseo para otros clientes
* Agregar objetos a la base de datos para una cliente
* Manipulación de algunos workflows
* Actuar sobre ellos (crear, consultar, actualizar, cerrar)

_Analistas y QA no tenían visión de los incidentes procesados por el SOAR._

---

## Solución

Componentes principales:

1. *Dashboard*
    * Visualización de incidentes (running, stopped, completed)
    * Filtros avanzados (severidad, estado, tiempo)
    * Actualización automática
2. *CRUD de objetos*
    * Interfaz intuitiva para agregar, actualizar, eliminar información de los objetos
3. *Workflow views*
    * Visualización de todos los workflows y sus resultados
    * Interacción con algunos workflows (confirmación o denegación de otra acción)
    * Botones para tomar decisiones: Close, Reassign, Override Severity, Escalate
    * Cada acción registra quién, cuándo, por qué
    * Impacto en tiempo real en el SOAR (ticket se crea, severidad se asigna)
4. *Login*
    * Ingreso para usuarios del dominio
    * Toda acción queda registrada por id, nombre y correo

![ui](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhDB3i7plCHbG_995UIwg4xhyxFkFsA4p3RvHkw06DzdncfAm76a10fI4-VUH-pKmGrylDETR53VbKce2gg-CFqNoVxN64yScBKtL3vZuoAvxc98xzY2FfZob3m52-cxhIlGnvPDrZQZDJiCgc3DLsdStXDWEkdj1_x5NHRaUTRC7QoeWSyfzpKTKbQE5pB/s320/Diagramas%20(1).png)

---

## Diseño

### Next.js + Typescript (migrado desde JavaScript)

La primera versión se escribió en JavaScript, al crecer el proyecto se migró a TypeScript porque necesitaba un tipado fuerte que obligara a seguir estructuras estándar para comunicarse con la API.

* Migración de JS a TS hecha sobre un proyecto en marcha, no desde cero
* Next.js como framework de la aplicación (App Router, Pages Router, autenticación)

### GraphQL

Todas las operaciones CRUD sobre incidentes pasan por GraphQL. Esto fue lo más complejo del proyecto: la UI tenía que comunicarse con un backend con su propio esquema, y había que mantener sincronizados los tipos de TypeScript con ese esquema.

* Consultas para listar y detallar incidentes
* Mutaciones para crear, actualizar y cerrar

### Autenticación con NextAuth + Microsoft Entra ID

El login usa NextAuth conectado a Entra ID, por lo que cualquier usuario del SOC accede con su cuenta corporativa.

* Sin contraseñas propias que administrar
* El acceso se alinea con el ciclo de vida de la cuenta corporativa: si alguien sale del equipo, pierde acceso
* El acceso se limitó por grupo de Entra, para que solo sea accesible a usuarios que realmente lo requieren

![auth](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEju2uj7z1XDSzxFrijkYZgv-PiI870DC6mv2D43jz08gw41V1cGW7K9_SL9C5Z3us-29jXCNEQoOe0FHJHHy2hwxqFpAAGnIQWzLmW17Oe4SDzKoAnvpA-N1C799ptiudAmSGSp5TTgbHmY7XvCj6aCW2vLf8qetNgAhwiUSVz8ZdgMKN6WL6MZF8nZUbme/s1280/Diagramas.png)

---

## Auditoría de cada acción

Como cada usuario entra con su identidad de Entra ID, cada acción queda asociada a una persona real, quién hizo qué y cuándo, perimitiendo tener trazabilidad.

---

## Desafíos técnicos

1. *Integrar la UI con GraphQL*: El esquema del SOAR definía la forma de los datos y la interfaz tenía que adaptarse a él, el schema especifica un tipo especifico de valor y requerido, y armar el json desde el frontend ajustaba los datos de diferente forma.

2. *Migrar de JavaScript a TypeScript a mitad de proyecto*: Había que tipar componentes y respuestas existentes sin romper lo que ya funcionaba. Comenzar por la migración de pequeños componentes (lists) hasta terminar con la primera capa de la interfaz (login).

3. *Trazabilidad de punta a punta*: Conectar la identidad de Entra ID con cada operación GraphQL para que la auditoría fuera completa y no dependiera de que el analista "declarara" quién era.

4. *Visualización en tiempo real*: Uno de los propósitos de esta interfaz era poder ver los incidentes en todos sus estados, además de tener un Quickstart de un evento que requiera intervención.

---

## Impacto

* Acceso unificado de todo el SOC con credenciales corporativas
* Auditoría completa de acciones sobre incidentes
* Interfaz única para gestionar los incidentes que antes se revisaban por partes
* Dashboard con métricas reales, teniendo un 95% de incidentes completados, 4% en espera de intervención por parte de analistas y 1% de incidentes en ejecución

---

## Stack

* *Aplicación*: Next.js, TypeScript, Tailwind CSS, shadcn/ui
* *Identidad*: NextAuth
* *Despliegue*: Kubernetes (MicroK8s), Skaffold

## [Atrás](./)