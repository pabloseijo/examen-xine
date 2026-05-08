---
title: Replicación y Sharding en MongoDB — Práctica
type: tema
fuentes: [Xestión de Información non Estruturada [G4012453] [2025_2026]_ Replicación y Sharding en MongoDB _ Campus Virtual.pdf, EntregaDia1..pdf, ComandosDia2.txt]
última_actualización: 2026-05-07
---

# Replicación y Sharding en MongoDB — Práctica

## Ejemplo 1: Cluster con Replicación (Replica Set)

### Arquitectura
3 contenedores Docker en la misma red, cada uno ejecuta un proceso mongod con el mismo identificador de replica set.

### Setup
```bash
# Crear red Docker
docker network create network-xine-grei

# Crear los tres contenedores (bash como comando para control manual)
docker run --name xine-grei-mongodb-rs0-1 -dti \
  -v c:\temp:/home/alumnobd/host-temp \
  --network network-xine-grei mongo:latest bash

docker run --name xine-grei-mongodb-rs0-2 -dti \
  --network network-xine-grei mongo:latest bash

docker run --name xine-grei-mongodb-rs0-3 -dti \
  --network network-xine-grei mongo:latest bash
```

### Arrancar mongod en cada nodo
```bash
# En cada contenedor:
docker exec -it xine-grei-mongodb-rs0-1 bash

mkdir /home/alumnobd/mongo
nohup mongod --replSet rs0 --port 27017 --bind_ip_all \
  --dbpath /home/alumnobd/mongo > /dev/null &
```

Opciones importantes:
- `--replSet rs0`: identificador del replica set (mismo en todos los nodos)
- `--bind_ip_all`: acepta conexiones desde cualquier IP
- `--dbpath`: directorio de datos
- `nohup ... &`: proceso en background, persiste si se cierra la terminal

### Inicializar el replica set (solo una vez, desde un nodo)
```javascript
// Conectar a mongosh en rs0-1
docker exec -it xine-grei-mongodb-rs0-1 mongosh

rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "xine-grei-mongodb-rs0-1:27017" },
    { _id: 1, host: "xine-grei-mongodb-rs0-2:27017" },
    { _id: 2, host: "xine-grei-mongodb-rs0-3:27017" }
  ]
})
```

### Verificar estado
```javascript
rs.status()   // estado de los miembros (PRIMARY/SECONDARY)
rs.conf()     // configuración del replica set
```

### Conectar al replica set (desde cualquier nodo)
```bash
mongosh "mongodb://rs0-1:27017,rs0-2:27017,rs0-3:27017/?replicaSet=rs0"
```

### Probar disponibilidad
```javascript
// Insertar datos de prueba
use xine
db.coleccionPrueba.insertOne({clave:1, valor:5})
for (var i=2; i<1000; i++) { db.coleccionPrueba.insertOne({clave:i, valor:i+5}) }
db.coleccionPrueba.find().count()

// Pausar un contenedor → la consulta sigue funcionando (quórum con 2/3)
// docker pause xine-grei-mongodb-rs0-2

// Pausar otro → puede que el primario pierda quórum
// → cambiar read preference si el primario cae
db.getMongo().setReadPref("primaryPreferred")

// Restaurar
db.getMongo().setReadPref("primary")
```

---

## Ejemplo 2: Cluster con Sharding (sin replicación de shards)

### Arquitectura simplificada
```
[broker container]: config server + mongos
[shard1 container]: shard 1 (rsSH1)
[shard2 container]: shard 2 (rsSH2)
```

### 1. Desplegar Config Server (en broker)
```bash
docker run --name xine-grei-mongodb-broker -dti \
  -p 27017:27017 --network network-xine-grei mongo:latest bash
docker exec -it xine-grei-mongodb-broker bash

mkdir /home/alumnobd/servconf
nohup mongod --configsvr --replSet rsCS --port 27018 --bind_ip_all \
  --dbpath /home/alumnobd/servconf > /dev/null &

mongosh --port 27018
rs.initiate({
  _id: "rsCS",
  configsvr: true,
  members: [{_id: 0, host: "xine-grei-mongodb-broker:27018"}]
})
```

### 2. Desplegar shards
```bash
# Shard 1
docker run --name xine-grei-mongodb-shard1 -dti --network network-xine-grei mongo:latest bash
docker exec -it xine-grei-mongodb-shard1 bash
mkdir /home/alumnobd/shard1
nohup mongod --shardsvr --replSet rsSH1 --port 27017 \
  --dbpath /home/alumnobd/shard1 --bind_ip_all > /dev/null &
mongosh --port 27017
rs.initiate({ _id: "rsSH1", members: [{_id: 0, host: "xine-grei-mongodb-shard1:27017"}] })

# Shard 2 (análogo con rsSH2 y shard2)
```

### 3. Arrancar mongos y añadir shards
```bash
docker exec -it xine-grei-mongodb-broker bash
nohup mongos --port 27017 \
  --configdb rsCS/xine-grei-mongodb-broker:27018 --bind_ip_all > /dev/null &

mongosh  # conecta al mongos en puerto 27017
sh.addShard("rsSH1/xine-grei-mongodb-shard1:27017")
sh.addShard("rsSH2/xine-grei-mongodb-shard2:27017")
```

### 4. Habilitar sharding y particionar colección
```javascript
use xine
sh.enableSharding("xine")

// Reducir tamaño de chunk para ver movimiento más rápido (solo para pruebas)
use config
db.settings.updateOne(
  { _id: "chunksize" },
  { $set: { _id: "chunksize", value: 1 } },
  { upsert: true }
)

// Crear índice y particionar
use xine
db.coleccionPrueba.insertOne({clave: 1, valor: "texto"})
db.coleccionPrueba.createIndex({clave: "hashed"})
sh.shardCollection("xine.coleccionPrueba", {clave: "hashed"})

// Insertar datos masivos para ver distribución
for (var i=2; i<100000; i++) {
  db.coleccionPrueba.insertOne({clave: i, valor: "texto suficientemente largo..."})
}

sh.status()  // ver distribución de chunks entre shards
```

---

## Ejemplo 3: Cluster con Sharding Y Replicación (ejercicio de evaluación)

### Arquitectura completa (3 contenedores × 4 procesos mongod + 1 mongos cada uno)

```
           mongo1            mongo2            mongo3
mongos:    27017             27017             27017
configsvr: 27019 (rsConf)   27019 (rsConf)   27019 (rsConf)
shard1:    27101 (rsShard1) 27101 (rsShard1) 27101 (rsShard1)
shard2:    27201 (rsShard2) 27201 (rsShard2) 27201 (rsShard2)
shard3:    27301 (rsShard3) 27301 (rsShard3) 27301 (rsShard3)
```

### Crear contenedores
```bash
docker network create mongo-cluster-net
docker run -d --name mongo1 --network mongo-cluster-net mongo:latest tail -f /dev/null
docker run -d --name mongo2 --network mongo-cluster-net mongo:latest tail -f /dev/null
docker run -d --name mongo3 --network mongo-cluster-net mongo:latest tail -f /dev/null
```

`tail -f /dev/null`: mantiene el contenedor vivo sin arrancar mongod automáticamente.

### Crear directorios en cada contenedor
```bash
docker exec -it mongo1 bash
mkdir -p /data/configdb /data/shard1 /data/shard2 /data/shard3 /data/logs
exit
# Repetir para mongo2 y mongo3
```

### Arrancar Config Server (rsConfServer) en los 3 contenedores
```bash
docker exec mongo1 mongod --configsvr --replSet rsConfServer --port 27019 \
  --dbpath /data/configdb --bind_ip_all --fork --logpath /data/logs/configsvr.log
docker exec mongo2 mongod --configsvr --replSet rsConfServer --port 27019 \
  --dbpath /data/configdb --bind_ip_all --fork --logpath /data/logs/configsvr.log
docker exec mongo3 mongod --configsvr --replSet rsConfServer --port 27019 \
  --dbpath /data/configdb --bind_ip_all --fork --logpath /data/logs/configsvr.log

# Inicializar el replica set del config server
docker exec -it mongo1 mongosh --port 27019
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

### Arrancar Shards (mismo patrón para shard1/shard2/shard3)
```bash
# Shard1 en los 3 contenedores (puerto 27101)
docker exec mongo1 mongod --shardsvr --replSet rsShard1 --port 27101 \
  --dbpath /data/shard1 --bind_ip_all --fork --logpath /data/logs/shard1.log
docker exec mongo2 mongod --shardsvr --replSet rsShard1 --port 27101 \
  --dbpath /data/shard1 --bind_ip_all --fork --logpath /data/logs/shard1.log
docker exec mongo3 mongod --shardsvr --replSet rsShard1 --port 27101 \
  --dbpath /data/shard1 --bind_ip_all --fork --logpath /data/logs/shard1.log

docker exec -it mongo1 mongosh --port 27101
rs.initiate({
  _id: "rsShard1",
  members: [
    { _id: 0, host: "mongo1:27101" },
    { _id: 1, host: "mongo2:27101" },
    { _id: 2, host: "mongo3:27101" }
  ]
})
# Repetir para rsShard2 (puerto 27201) y rsShard3 (puerto 27301)
```

### Arrancar mongos y registrar shards
```bash
docker exec mongo1 mongos --configdb rsConfServer/mongo1:27019,mongo2:27019,mongo3:27019 \
  --bind_ip_all --port 27017 --fork --logpath /data/logs/mongos.log
# Repetir para mongo2 y mongo3

docker exec -it mongo1 mongosh --port 27017
sh.addShard("rsShard1/mongo1:27101,mongo2:27101,mongo3:27101")
sh.addShard("rsShard2/mongo1:27201,mongo2:27201,mongo3:27201")
sh.addShard("rsShard3/mongo1:27301,mongo2:27301,mongo3:27301")
sh.status()
```

---

## Prueba de disponibilidad (cuántas máquinas pueden caer)

Con factor de replicación N=3 y quórum W=2, R=2:
- **1 máquina caída**: el cluster sigue funcionando (quórum con 2/3 nodos)
- **2 máquinas caídas**: el cluster pierde quórum → no puede aceptar escrituras ni leer de forma consistente

---

## Importar y particionar datos de películas
```javascript
use xine
sh.enableSharding("xine")

// Crear índice hash sobre id
db.peliculas.createIndex({ id: "hashed" })
sh.shardCollection("xine.peliculas", { id: "hashed" })

// Importar desde JSON exportado de PostgreSQL
// docker cp peliculas_mongo.json mongo1:/tmp/
// docker exec -it mongo1 mongoimport --port 27017 --db xine --collection peliculas --drop --file /tmp/peliculas_mongo.json
```

Ver también: [[sharding]], [[replicacion]], [[mongodb]], [[05-mongodb]]
