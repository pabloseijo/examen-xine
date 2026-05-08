---
title: MongoDB
type: concepto
fuentes: [IntroMongoDB.pdf]
última_actualización: 2026-05-07
---

# MongoDB

## Qué es
Base de datos de documentos [[nosql]] líder del mercado. Almacena documentos BSON (Binary JSON) en colecciones. Diseñada para escalar horizontalmente.

## Modelo de datos
- **Documento**: objeto JSON/BSON con pares campo-valor. Valores pueden ser anidados (subdocumentos, arrays).
- **Colección**: agrupación de documentos (equivalente a tabla). Sin esquema fijo.
- **Base de datos**: agrupación de colecciones.

## Características clave
- Esquema dinámico (schema-less): no requiere declarar estructura
- Atomicidad a nivel de documento (operaciones CRUD sobre un doc son atómicas)
- Transacciones multi-documento disponibles desde v4.0
- Índices sobre campos de documentos anidados y arrays
- Pipeline de agregación potente

## Alta disponibilidad: Replica Set
- Primary (acepta escrituras) + Secondaries (replican) + opcionalmente Arbiter (vota sin datos)
- Replicación asíncrona; recuperación automática (heartbeat + elección)
- Ver [[replicacion]]

## Escalado horizontal: Sharding
- Cluster: mongos (router) + config servers + shards (cada shard es un replica set)
- Shard key determina distribución
- Hashed Sharding vs. Ranged Sharding
- Ver [[sharding]]

## Comandos esenciales
```javascript
// Consulta básica
db.peliculas.find({genero: "Action"}).sort({ingresos: -1}).limit(10)

// Aggregation
db.peliculas.aggregate([
  { $match: {ingresos: {$gt: 800000000}} },
  { $group: {_id: "$genero", total: {$sum: 1}} }
])

// Replica set
rs.initiate({_id:"rs0", members:[{_id:0,host:"s1:27017"},{_id:1,host:"s2:27017"}]})
rs.status()

// Sharding
sh.enableSharding("mi_bd")
sh.shardCollection("mi_bd.col", {id: "hashed"})
sh.addShard("rsX/host1:27017,host2:27017,host3:27017")
```

## Ver también
[[05-mongodb]], [[07-replicacion-sharding-mongodb]], [[sharding]], [[replicacion]], [[modelado-documentos]]
