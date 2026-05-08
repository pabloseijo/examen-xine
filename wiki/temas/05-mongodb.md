---
title: MongoDB — Introducción
type: tema
fuentes: [IntroMongoDB.pdf, Xestión de Información non Estruturada [G4012453] [2025_2026]_ Introducción a MongoDB _ Campus Virtual.pdf]
última_actualización: 2026-05-07
---

# MongoDB — Introducción

## Qué es MongoDB

MongoDB es una **base de datos de documentos** NoSQL de código abierto. Almacena datos como documentos JSON (internamente BSON — Binary JSON). Es la BD NoSQL más utilizada en la industria.

---

## Modelo de datos

### Documentos
Un documento es un objeto JSON con pares `campo: valor`. Los valores pueden ser:
- Tipos primitivos: string, número, booleano, null, fecha
- **Subdocumentos** (objetos anidados)
- **Arrays** (listas de valores o subdocumentos)

```json
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "titulo": "El Padrino",
  "año": 1972,
  "director": {
    "nombre": "Francis Ford Coppola",
    "nacionalidad": "USA"
  },
  "generos": ["Crime", "Drama"],
  "reparto": [
    {"actor": "Marlon Brando", "personaje": "Vito Corleone", "orden": 0},
    {"actor": "Al Pacino", "personaje": "Michael Corleone", "orden": 1}
  ]
}
```

### Colecciones
- Agrupación de documentos (equivalente a tabla en relacional)
- **Sin esquema fijo**: cada documento puede tener campos distintos (polimorfismo)
- Una colección puede mezclar documentos con estructuras diferentes

### Tipos especiales
- **Vistas** (`View`): colecciones de solo lectura basadas en una agregación
- **Vistas materializadas** (`$merge`, `$out`): resultado de agregación guardado como colección

---

## Ventajas del modelo de documentos

### 1. Correspondencia con estructuras OO
Los documentos JSON se mapean directamente a objetos en lenguajes OO (Java, Python, JavaScript). Reduce el impedance mismatch.

### 2. Reducción de JOINs
Al embeber datos relacionados en un único documento, se eliminan los JOINs costosos. Una sola lectura devuelve todos los datos necesarios.

### 3. Esquema dinámico (polimorfismo fluido)
Sin necesidad de declarar esquema. Útil para:
- Evolución rápida del modelo de datos
- Documentos con campos opcionales
- Datos heterogéneos en la misma colección

### 4. Alto rendimiento
- El modelo anidado reduce el número de E/S (una sola lectura en lugar de múltiples JOINs)
- Índices sobre campos de documentos anidados y arrays
- Soporte para índices compuestos, de texto completo, geoespaciales

---

## Tipos de consultas

- **CRUD**: `find()`, `insertOne()`, `updateOne()`, `deleteOne()` y variantes Many
- **Agregación**: pipeline `$match`, `$group`, `$project`, `$sort`, `$limit`, `$unwind`, `$lookup`
- **Texto completo**: índices de texto para búsqueda full-text
- **Geoespaciales**: `$near`, `$geoWithin` para consultas de proximidad

---

## Modelado en MongoDB

### Regla fundamental
El diseño del modelo depende de **cómo se accede a los datos**, no de cómo están relacionados lógicamente.

### Validación con JSON Schema (desde v3.6)
MongoDB permite opcionalmente definir reglas de validación:
```javascript
db.createCollection("peliculas", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["titulo", "año"],
      properties: {
        titulo: { bsonType: "string" },
        año: { bsonType: "int", minimum: 1888 }
      }
    }
  }
});
```

### Datos anidados (embedded)
```json
// Toda la información en un solo documento
{
  "_id": "orden-001",
  "cliente": {"nombre": "Ana", "email": "ana@ex.com"},
  "productos": [
    {"nombre": "Laptop", "precio": 999}
  ]
}
```

### Referencias entre documentos
```json
// Orden con referencia al ID del cliente
{
  "_id": "orden-001",
  "cliente_id": ObjectId("abc123"),
  "productos": [...]
}
```

### Atomicidad
- **Nivel de documento**: las operaciones CRUD sobre un solo documento son atómicas
- **Multi-documento**: desde MongoDB 4.0 hay soporte para transacciones ACID multi-documento (en replica sets), desde 4.2 en clusters con sharding

---

## Replica Sets

### Arquitectura
```
Replica Set:
  [PRIMARY]   ← acepta todas las escrituras
  [SECONDARY] ← replica desde el primario
  [SECONDARY] ← replica desde el primario
  [ARBITER]   ← vota en elecciones, SIN datos
```

- **Primario**: único nodo que acepta escrituras
- **Secundarios**: replican de forma **asíncrona** del primario; pueden servir lecturas (configurable)
- **Árbitro (Arbiter)**: participa en votaciones para elegir nuevo primario, pero NO almacena datos. Útil para tener quórum de votación sin coste de almacenamiento.

### Configurar con rs.initiate()
```javascript
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "server1:27017" },
    { _id: 1, host: "server2:27017" },
    { _id: 2, host: "server3:27017" }
  ]
})
```

### Replicación asíncrona
Las escrituras se confirman al cliente tras escribir en el primario. Los secundarios replican después. Puede haber un pequeño lag.

### Recuperación automática
1. Secundarios envían **heartbeats** al primario cada 2 segundos
2. Si el primario no responde en 10 segundos → se inicia **elección**
3. Los secundarios votan → se elige nuevo primario
4. El proceso es automático y transparente para la aplicación (con drivers oficiales)

### Read preference
- `primary` (por defecto): todas las lecturas van al primario
- `primaryPreferred`: al primario si está disponible, si no a un secundario
- `secondary`: siempre a un secundario (posible lectura de datos obsoletos)
- `nearest`: al nodo con menor latencia

---

## Sharding en MongoDB

### Componentes del cluster
```
Cliente → [mongos router] → [Config Servers (rsConfServer)]
                         → [Shard 1 (rsShard1: Primary+Secondaries)]
                         → [Shard 2 (rsShard2: Primary+Secondaries)]
                         → [Shard 3 (rsShard3: Primary+Secondaries)]
```

| Componente | Función |
|------------|---------|
| **mongos** | Router de consultas. No almacena datos. Redirige al shard correcto. |
| **Config Servers** | Almacenan metadatos del cluster (mapeo chunk→shard). Deben ser un replica set. |
| **Shards** | Almacenan los datos reales. Cada shard es un replica set. |

### Shard Key
- Campo (o campos) del documento que determina en qué shard se almacena
- Debe tener un índice (simple o compuesto)
- **Crítica**: una mala shard key puede arruinar el rendimiento

### Chunks
- Los datos de una colección se dividen en **chunks** (rangos de valores de la shard key)
- Tamaño por defecto: 128MB
- El **balanceador** automático mueve chunks entre shards para mantener equilibrio
- Se puede configurar: `db.settings.updateOne({_id:"chunksize"}, {$set:{value:1}})`

### Habilitar sharding y particionar colección
```javascript
sh.enableSharding("mi_base_de_datos")
db.coleccion.createIndex({campo: "hashed"})  // o 1 para ranged
sh.shardCollection("mi_bd.coleccion", {campo: "hashed"})
sh.status()  // ver estado del cluster
```

---

## Tipos de sharding

### Hashed Sharding
```javascript
sh.shardCollection("xine.peliculas", { id: "hashed" })
```
- El valor de la shard key se pasa por una función hash
- Distribución muy uniforme de datos entre shards
- **Malo para consultas de rango**: los rangos de shard key quedan distribuidos aleatoriamente → scatter-gather
- Ideal para: distribución uniforme, sin consultas de rango

### Ranged Sharding
```javascript
sh.shardCollection("xine.peliculas", { fecha: 1 })
```
- Los chunks son rangos contiguos de la shard key
- **Bueno para consultas de rango**: una query de rango va a pocos shards
- **Mal elección de clave → hotspot**: si la clave es monotónica (e.g., fecha de inserción), todos los datos nuevos van al mismo shard

---

## Criterios de elección de Shard Key

### 1. Cardinalidad
Número de valores distintos posibles de la shard key.
- **Alta cardinalidad**: muchos valores distintos → puede haber muchos chunks → distribución fina
- **Baja cardinalidad**: pocos valores distintos (e.g., `genero` con 5 valores) → máximo 5 shards posibles, no escala

### 2. Frecuencia
Distribución de los valores de la shard key en los datos.
- **Distribución uniforme**: ideal, todos los shards reciben carga similar
- **Distribución sesgada**: algunos valores muy frecuentes → shards sobrecargados (hotspot)

### 3. Monotonicidad
¿Los valores de la shard key siempre crecen (o decrecen) con el tiempo?
- **Clave monotónica** (e.g., timestamp, ObjectId por defecto): todos los nuevos datos van al mismo shard extremo → hotspot de escritura
- **Solución**: usar hashed sharding sobre una clave monotónica

---

## Zonas (Zones)

Permiten asignar rangos de shard key a shards específicos:
```javascript
sh.addShardToZone("shard1", "zona-europe")
sh.updateZoneKeyRange("xine.peliculas", {pais: "ES"}, {pais: "ES￿"}, "zona-europe")
```
- Mejoran **localidad geográfica**: datos de usuarios europeos en shards en Europa
- Útiles para **cumplimiento normativo** (GDPR: datos en Europa)
- Permiten **tiered storage**: datos calientes en shards con SSD, fríos en HDD

---

## Comandos MongoDB esenciales

```javascript
// CRUD básico
db.peliculas.insertOne({titulo: "Avatar", año: 2009})
db.peliculas.find({genero: "Action"}).sort({ingresos: -1}).limit(10)
db.peliculas.updateMany({}, {$set: {activo: true}})
db.peliculas.deleteOne({_id: ObjectId("...")})

// Aggregation pipeline
db.peliculas.aggregate([
  { $match: { ingresos: { $gt: 800000000 } } },
  { $group: { _id: "$genero", total: { $sum: 1 }, media_presupuesto: { $avg: "$presupuesto" } } },
  { $sort: { total: -1 } }
])

// Convertir fecha string a Date
db.peliculas.updateMany({}, [{
  $set: { fecha_emision: { $dateFromString: { dateString: "$fecha_emision" } } }
}])

// Replica set
rs.status()
rs.conf()

// Sharding
sh.status()
sh.enableSharding("mi_bd")
sh.shardCollection("mi_bd.coleccion", {campo: "hashed"})
sh.addShard("rsNombre/host1:puerto,host2:puerto,host3:puerto")
```

Ver también: [[replicacion]], [[sharding]], [[nosql]], [[07-replicacion-sharding-mongodb]]
