---
title: Simulacro 01 — Bases de Datos (modelo 2025)
type: simulacro
fecha: 2026-05-14
temas: [quórums, esquema-nosql, sharding, replicación]
---

# Simulacro 01 — Bases de Datos (modelo 2025)

**Tiempo orientativo: 25 minutos**
**Formato**: 3 preguntas de desarrollo. La parte práctica usa nombres distintos a los originales pero misma estructura.

---

## Pregunta 1

Un sistema de reservas de vuelos usa una base de datos distribuida con replicación entre 5 nodos (N=5). El equipo de ingeniería configura W=3 y R=1.

**a)** ¿Se garantiza que una lectura devuelve siempre la escritura más reciente? Justifica con la fórmula correspondiente.

**b)** ¿Se garantiza que no puede haber dos escrituras conflictivas simultáneas? Justifica.

**c)** ¿Qué tipo de consistencia ofrece este sistema según el modelo BASE? ¿Cambiaría algo si fuese un sistema bancario? Razona.

---

## Pregunta 2

Una startup decide migrar su aplicación de una base de datos relacional PostgreSQL a MongoDB. La tabla principal tiene 8 millones de filas y el equipo de desarrollo quiere añadir un nuevo campo `preferencias_usuario` (un objeto JSON con campos variables según el usuario).

**a)** Explica qué ventaja ofrece el modelo NoSQL frente al relacional para este cambio concreto. ¿Qué concepto resume esta característica?

**b)** Describe los pasos que debe seguir el equipo durante la migración, teniendo en cuenta que hay datos existentes en producción que no se pueden perder.

**c)** ¿Qué riesgo asume la aplicación al no haber un esquema fijo en la base de datos? ¿Quién pasa a ser responsable de la integridad de los datos?

---

## Pregunta 3

Dado el siguiente comando ejecutado durante la configuración de un cluster MongoDB:

```javascript
rs.initiate({
  _id: "rsShardAlpha",
  members: [
    { _id: 0, host: "192.168.1.10:27020" },
    { _id: 1, host: "192.168.1.11:27020" },
    { _id: 2, host: "192.168.1.12:27020" }
  ]
})
```

Y posteriormente:

```javascript
sh.shardCollection("tienda.productos", { categoria: "hashed" })
```

**a)** ¿Qué hace `rs.initiate` en este contexto? ¿Qué rol tiene este replica set dentro del cluster?

**b)** Explica qué significa el parámetro `{ categoria: "hashed" }` en `shardCollection`. ¿Qué ventaja tiene frente a `{ categoria: 1 }`? ¿Y qué desventaja?

**c)** Si `categoria` solo tuviese 3 valores posibles ("electrónica", "ropa", "alimentación"), ¿sería una buena shard key? ¿Por qué?

---

## Soluciones

> Completa el simulacro antes de leer esto.

<details>
<summary>Ver soluciones</summary>

### P1a) ¿Garantiza lectura consistente?
R + W = 1 + 3 = 4. N = 5. Como 4 < 5, **NO se cumple R + W > N** → no se garantiza leer la escritura más reciente → consistencia eventual.

### P1b) ¿Garantiza no conflictos de escritura?
W = 3 > N/2 = 2.5 → **SÍ**. Solo puede haber una escritura con mayoría al mismo tiempo, por lo que no pueden coexistir dos escrituras conflictivas con quórum.

### P1c) Consistencia y sistema bancario
Con R+W ≤ N el sistema ofrece **consistencia eventual** (modelo BASE). Para un sistema bancario esto **no sería aceptable**: una transferencia debe ser atómica y consistente (ACID). Habría que subir R para que R+W > N, o usar un sistema CP, sacrificando disponibilidad.

### P2a) Ventaja NoSQL y concepto
El modelo NoSQL no tiene esquema fijo en la BD — el **esquema está en la aplicación** (schema-on-read). Añadir `preferencias_usuario` no requiere `ALTER TABLE` ni migración bloqueante: basta con que los nuevos documentos incluyan el campo. Los documentos viejos sin él conviven sin problema.

### P2b) Pasos de migración legacy
1. Mantener el código antiguo funcionando (no romper producción)
2. Escribir los nuevos documentos ya con `preferencias_usuario`
3. Los documentos viejos siguen sin el campo — la app detecta su ausencia y usa un valor por defecto
4. Migrar los docs viejos gradualmente (background job) o al vuelo cuando se acceden
5. Cuando todos los docs tengan el campo, se puede simplificar el código de compatibilidad

### P2c) Riesgo e integridad
Sin esquema en la BD, la **aplicación** es responsable de la integridad. Riesgos: documentos con campos mal nombrados, tipos de dato incorrectos, campos obligatorios ausentes. Herramientas como validación de esquema en MongoDB (JSON Schema) o validación en capa de servicio mitigan esto.

### P3a) rs.initiate y rol
`rs.initiate` inicializa un **replica set** con 3 miembros. En el contexto del cluster, con `_id: "rsShardAlpha"` este replica set actúa como un **shard** (no como config server). Cada shard en MongoDB es internamente un replica set para garantizar disponibilidad dentro del shard.

### P3b) hashed vs ranged
`{ categoria: "hashed" }` aplica una función hash a `categoria` antes de asignar el documento a un shard. Ventaja frente a `{ categoria: 1 }` (ranged): **distribución uniforme**, evita hotspots. Desventaja: **malo para range queries** — documentos con valores de categoría cercanos acaban en shards distintos, por lo que una consulta `categoria >= X` necesita ir a todos los shards.

### P3c) ¿Es buena shard key con 3 valores?
**No**. Solo 3 valores posibles = **cardinalidad muy baja** = máximo 3 shards útiles. Si el cluster tiene más de 3 shards, algunos quedarán vacíos. Además, si una categoría concentra la mayoría de los documentos (ej: "alimentación" tiene el 80% del catálogo), habrá un hotspot en ese shard.

</details>
