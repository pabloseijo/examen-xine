---
title: MongoDB — Comandos del Cluster (Práctica)
type: concepto
fuentes: [ComandosDia2.txt, EntregaDia1.pdf]
última_actualización: 2026-05-14
---

# MongoDB — Comandos del Cluster

> El profesor confirmó que los comandos del examen son los de las prácticas. Saber leer un comando y explicar qué hace cada parámetro.

---

## FASE 1: Arrancar los nodos

```javascript
// Config Server
mongod --configsvr --replSet rsConfServer --port 27019
       --dbpath /data/configdb --bind_ip_all
       --fork --logpath /data/logs/configsvr.log

// Shard de datos
mongod --shardsvrr --replSet rsShard1 --port 27101
       --dbpath /data/shard1 --bind_ip_all
       --fork --logpath /data/logs/shard1.log
```

| Parámetro | Qué hace |
|-----------|---------|
| `--configsvr` | Este mongod es Config Server — guarda metadatos del cluster, no datos reales |
| `--shardsvrr` | Este mongod es nodo de datos de un shard |
| `--replSet X` | Pertenece al replica set X (debe coincidir con el nombre en rs.initiate) |
| `--port X` | Puerto en el que escucha |
| `--dbpath /ruta` | Carpeta donde guarda los datos en disco |
| `--bind_ip_all` | Acepta conexiones desde cualquier IP (necesario en Docker) |
| `--fork` | Arranca en segundo plano, no bloquea la terminal |
| `--logpath /ruta` | Fichero donde escribe los logs |

> **Diferencia clave**: `--configsvr` = guarda metadatos del cluster. `--shardsvrr` = guarda datos reales de la colección.

---

## FASE 2: Inicializar los replica sets

```javascript
// Config Server (lleva configsvr: true)
rs.initiate({
  _id: "rsConfServer",
  configsvr: true,
  members: [
    { _id: 0, host: "mongo1:27019" },
    { _id: 1, host: "mongo2:27019" },
    { _id: 2, host: "mongo3:27019" }
  ]
})

// Shard de datos (sin configsvr)
rs.initiate({
  _id: "rsShard1",
  members: [
    { _id: 0, host: "mongo1:27101" },
    { _id: 1, host: "mongo2:27101" },
    { _id: 2, host: "mongo3:27101" }
  ]
})
```

- Se ejecuta dentro de mongosh conectado al puerto correspondiente
- `_id` debe coincidir con el `--replSet` del mongod
- `configsvr: true` solo en el Config Server, nunca en los shards
- Uno de los tres nodos se elige PRIMARY automáticamente, los otros SECONDARY
- Se repite para rsShard2 (puerto 27201) y rsShard3 (puerto 27301)

---

## FASE 3: Verificar estado

```javascript
rs.status()   // estado del replica set: quién es PRIMARY/SECONDARY, health de cada nodo
sh.status()   // estado del cluster: shards registrados, colecciones particionadas, distribución de chunks
```

En `rs.status()` lo importante:
- `stateStr: 'PRIMARY'` → acepta escrituras
- `stateStr: 'SECONDARY'` → replica datos, solo lecturas
- `health: 1` → nodo activo; `health: 0` → nodo caído

---

## FASE 4: Arrancar el router (mongos)

```javascript
mongos --configdb rsConfServer/mongo1:27019,mongo2:27019,mongo3:27019
       --bind_ip_all --port 27017
       --fork --logpath /data/logs/mongos.log
```

- `mongos` no es mongod — no guarda datos, solo redirige queries al shard correcto
- `--configdb rsConfServer/...` → dónde está el Config Server (necesita los metadatos para saber a qué shard enviar cada query)
- El cliente siempre se conecta al puerto de mongos, nunca directamente a los shards

---

## FASE 5: Configurar el cluster

```javascript
// Registrar los shards en el cluster (conectado al mongos)
sh.addShard("rsShard1/mongo1:27101,mongo2:27101,mongo3:27101")
sh.addShard("rsShard2/mongo1:27201,mongo2:27201,mongo3:27201")
sh.addShard("rsShard3/mongo1:27301,mongo2:27301,mongo3:27301")

// Activar sharding en la base de datos
sh.enableSharding("xine")

// Crear índice hashed (OBLIGATORIO antes de shardear)
db.peliculas.createIndex({ id: "hashed" })

// Shardear la colección
sh.shardCollection("xine.peliculas", { id: "hashed" })
```

- `sh.addShard` → formato `nombreRS/host:puerto,...` — registra el replica set como shard del cluster
- `sh.enableSharding` → activa sharding en la BD; sin esto las colecciones no se pueden particionar
- `createIndex` debe ejecutarse **antes** de `shardCollection` — MongoDB exige que exista el índice sobre la shard key
- `sh.shardCollection` convierte la colección en sharded; `id: "hashed"` = distribución uniforme por hash del campo id

---

## FASE 6: Preferencia de lectura

```javascript
db.getMongo().setReadPref('primaryPreferred')
```

| Valor | Comportamiento |
|-------|---------------|
| `primary` | Solo del primario — consistencia total |
| `primaryPreferred` | Del primario si está disponible, si no del secundario |
| `secondary` | Solo de secundarios — puede dar datos obsoletos |
| `nearest` | Del nodo con menor latencia de red |

---

## FASE 7: Queries

```javascript
// Contar documentos que cumplen condición
db.peliculas.countDocuments({ presupuesto: { $gt: 500000 } })

// Buscar con filtro, proyección y ordenación
db.peliculas.find(
  { "generos.name": "Action", "reparto.name": "Penélope Cruz" },  // filtro
  { _id: 0, titulo: 1, titulo_original: 1 }                       // proyección
).sort({ fecha_emision: 1 })

// Actualizar muchos documentos
db.peliculas.updateMany(
  { fecha_emision: { $type: "string", $ne: "" } },
  [{ $set: { fecha_emision: { $dateFromString: { dateString: "$fecha_emision" } } } }]
)
```

- Proyección: `1` = incluir campo, `0` = excluir. No se pueden mezclar 1 y 0 salvo con `_id`
- `"generos.name"` → acceso a campo dentro de un array de subdocumentos
- `sort(1)` = ascendente, `sort(-1)` = descendente
- `$type: "string"` → filtra documentos donde el campo es de tipo string
- `$dateFromString` → convierte string a tipo Date

---

## FASE 8: Aggregation pipeline

El pipeline es una cadena de etapas — la salida de cada una es la entrada de la siguiente.

```javascript
db.peliculas.aggregate([

  { $match: { ingresos: { $gt: 0 } } },
  // Filtra documentos — equivale a WHERE en SQL
  // Solo pasan los que tienen ingresos > 0

  { $unwind: "$generos" },
  // generos es un array → lo descompone: una fila por elemento
  // Una película con 3 géneros se convierte en 3 documentos

  { $match: { "personal.job": "Director" } },
  // $match puede aparecer varias veces en el pipeline

  { $group: {
      _id: "$generos.name",                        // campo por el que agrupar
      numero_peliculas: { $sum: 1 },               // contar
      presupuesto_total: { $sum: "$presupuesto" }, // sumar
      duracion_media: { $avg: "$duracion" }        // media
  }},
  // Equivale a GROUP BY en SQL

  { $project: {
      _id: 0,
      genero: "$_id",                              // renombrar _id a genero
      numero_peliculas: 1,                         // incluir campo
      diferencia: { $subtract: ["$ingresos_totales", "$presupuesto_total"] }
      // calcular campo nuevo
  }},
  // Reformatea la salida: incluir/excluir/calcular campos

  { $sort: { diferencia: -1 } },
  // Ordenar por diferencia descendente

  { $limit: 10 }
  // Solo los 10 primeros resultados

])
```

### Resumen de etapas del pipeline

| Etapa | Equivalente SQL | Qué hace |
|-------|----------------|---------|
| `$match` | WHERE | Filtra documentos |
| `$unwind` | — | Descompone array → una fila por elemento |
| `$group` | GROUP BY | Agrupa y calcula acumulados (`$sum`, `$avg`, `$count`) |
| `$project` | SELECT | Elige campos y calcula nuevos |
| `$sort` | ORDER BY | Ordena resultados |
| `$limit` | LIMIT | Limita el número de resultados |

---

## En el examen

El examen pone comandos reales de la práctica (con nombres distintos pero misma estructura) y pide explicar qué hace cada uno. Lo importante:

1. Distinguir `--configsvr` vs `--shardsvrr`
2. Saber que `createIndex` va **antes** de `shardCollection`
3. Saber qué hace cada etapa del pipeline de agregación
4. Saber el significado de cada read preference

## Relación con otros conceptos

- [[sharding]] — conceptos teóricos de particionado
- [[replicacion]] — conceptos teóricos de replicación
- [[mongodb]] — arquitectura general
