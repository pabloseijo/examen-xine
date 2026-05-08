---
title: Ejercicio de Evaluación MongoDB
type: tema
fuentes: [EntregaDia1..pdf, ComandosDia2.txt, Ejercicio de evaluación de MongoDB _ Campus Virtual.pdf, Replicación y Sharding en MongoDB _ Campus Virtual.pdf]
última_actualización: 2026-05-07
---

# Ejercicio de Evaluación MongoDB

## Arquitectura del cluster

El ejercicio usa **3 máquinas** (contenedores Docker), cada una ejecutando:
- 1 proceso `mongos` (router)
- 1 proceso `mongod --configsvr`
- 3 procesos `mongod --shardsvr` (uno por shard)

Total: **3 × 5 = 15 procesos** por cluster completo.

| Componente | Replica Set | Puertos |
|------------|-------------|---------|
| Config Servers | `rsConfServer` | 27019 |
| Shard 1 | `rsShard1` | 27101 |
| Shard 2 | `rsShard2` | 27201 |
| Shard 3 | `rsShard3` | 27301 |
| Mongos | — | 27017 |

## Tarea 1: Iniciar los procesos mongod

### Config Server (en cada máquina)
```bash
mongod --configsvr --replSet rsConfServer \
  --port 27019 --dbpath /data/configdb \
  --bind_ip_all --fork --logpath /data/logs/configsvr.log
```

### Shard 1 (en cada máquina)
```bash
mongod --shardsvr --replSet rsShard1 \
  --port 27101 --dbpath /data/shard1 \
  --bind_ip_all --fork --logpath /data/logs/shard1.log
```

(Repetir para Shard 2 en puerto 27201, Shard 3 en 27301)

## Tarea 2: Inicializar los Replica Sets

### Config Server
```js
// Conectar a mongo1:27019
rs.initiate({
  _id: "rsConfServer",
  configsvr: true,
  members: [
    { _id: 0, host: "mongo1:27019" },
    { _id: 1, host: "mongo2:27019" },
    { _id: 2, host: "mongo3:27019" }
  ]
})
```

### Shard 1
```js
// Conectar a mongo1:27101
rs.initiate({
  _id: "rsShard1",
  members: [
    { _id: 0, host: "mongo1:27101" },
    { _id: 1, host: "mongo2:27101" },
    { _id: 2, host: "mongo3:27101" }
  ]
})
```

(Repetir para rsShard2 en 27201 y rsShard3 en 27301)

## Tarea 3: Iniciar mongos y configurar el sharding

```bash
# Iniciar mongos en cada máquina
mongos --configdb rsConfServer/mongo1:27019,mongo2:27019,mongo3:27019 \
  --bind_ip_all --port 27017 \
  --fork --logpath /data/logs/mongos.log
```

```js
// Conectar a mongos (puerto 27017) y añadir shards
sh.addShard("rsShard1/mongo1:27101,mongo2:27101,mongo3:27101")
sh.addShard("rsShard2/mongo1:27201,mongo2:27201,mongo3:27201")
sh.addShard("rsShard3/mongo1:27301,mongo2:27301,mongo3:27301")
```

## Tarea 4: Importar datos

```bash
# Importar fichero JSON de películas al mongos
mongoimport --host mongo1:27017 \
  --db xine --collection peliculas \
  --file /data/peliculas.json --jsonArray
```

## Tarea 5: Indexar y habilitar sharding

```js
// Habilitar sharding en la base de datos
sh.enableSharding("xine")

// Crear índice hashed en _id (necesario antes de shardCollection)
db.peliculas.createIndex({ _id: "hashed" })

// Shardear la colección usando hashed sharding sobre _id
sh.shardCollection("xine.peliculas", { _id: "hashed" })
```

## Tarea 6: Verificar distribución de chunks

```js
sh.status()
// Muestra distribución de chunks entre shards
// Con datos suficientes el balanceador moverá chunks automáticamente

// Para acelerar el balanceo, reducir el tamaño de chunk:
use config
db.settings.updateOne(
  { _id: "chunksize" },
  { $set: { value: 1 } },  // 1 MB
  { upsert: true }
)
```

## Tarea 7: Consultas básicas

```js
// Contar documentos totales
db.peliculas.countDocuments()

// Buscar películas de un género concreto
db.peliculas.find({ genres: "Comedy" }).count()

// Películas donde aparece Penélope Cruz
db.peliculas.find({ cast: "Penélope Cruz" })
  .projection({ titulo: 1, year: 1 })
```

## Tarea 8: Consulta con conversión de fecha

```js
// Películas estrenadas después de 2010, usando $dateFromString
db.peliculas.aggregate([
  {
    $addFields: {
      fechaEstreno: {
        $dateFromString: { dateString: "$released", format: "%Y-%m-%d" }
      }
    }
  },
  { $match: { fechaEstreno: { $gt: new Date("2010-01-01") } } },
  { $project: { title: 1, year: 1, released: 1 } }
])
```

## Tarea 9: Agregación por género

```js
// Número de películas por género (genres es un array)
db.peliculas.aggregate([
  { $unwind: "$genres" },
  { $group: { _id: "$genres", total: { $sum: 1 } } },
  { $sort: { total: -1 } }
])
```

## Tarea 10: Top 10 directores por duración media

```js
db.peliculas.aggregate([
  { $unwind: "$directors" },
  {
    $group: {
      _id: "$directors",
      duracion_media: { $avg: "$runtime" },
      num_peliculas: { $sum: 1 }
    }
  },
  { $match: { num_peliculas: { $gte: 5 } } },  // al menos 5 películas
  { $sort: { duracion_media: -1 } },
  { $limit: 10 },
  { $project: { director: "$_id", duracion_media: 1, num_peliculas: 1, _id: 0 } }
])
```

## Tarea 11: Prueba de disponibilidad

### Apagar 1 máquina (debe seguir funcionando)
```bash
# Desde fuera, detener la máquina mongo3
# El cluster sigue funcionando: cada replica set tiene mayoría (2/3)
# El mongos de mongo1 y mongo2 siguen respondiendo
```

```js
rs.status()  // Verificar que los miembros restantes son PRIMARY/SECONDARY
```

### Apagar 2 máquinas (puede dejar de funcionar)
```bash
# Detener mongo2 y mongo3
# RESULTADO: los replica sets pierden mayoría (1/3) → no pueden elegir PRIMARY
# El cluster se vuelve READ-ONLY o completamente inaccesible
```

```js
// Solo se puede leer en modo de lectura débil:
db.getMongo().setReadPref("secondaryPreferred")
```

La tolerancia a fallos depende de tener mayoría de nodos activos en cada replica set.

## Resumen de comandos esenciales

| Acción | Comando |
|--------|---------|
| Estado del cluster | `sh.status()` |
| Estado de un RS | `rs.status()` |
| Añadir shard | `sh.addShard("rsNombre/host:puerto")` |
| Shardear colección | `sh.shardCollection("db.col", {campo:"hashed"})` |
| Importar datos | `mongoimport --host ... --db ... --collection ... --file ...` |
| Reducir chunk | `db.settings.updateOne({_id:"chunksize"},{$set:{value:1}},{upsert:true})` |

## Ver también
[[05-mongodb]], [[07-replicacion-sharding-mongodb]], [[sharding]], [[replicacion]], [[mongodb]]
