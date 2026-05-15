---
title: NoSQL — Distribución
type: tema
fuentes: [NoSQL2.Distribucion.pdf]
última_actualización: 2026-05-07
---

# NoSQL — Distribución

## Las dos técnicas fundamentales: Sharding y Replicación

> Son **técnicas ortogonales**: se pueden aplicar independientemente o combinadas.

- **Sharding (particionado)**: dividir los datos entre varios nodos. Cada nodo tiene un subconjunto diferente.
- **Replicación**: copiar los mismos datos en varios nodos. Cada nodo tiene una copia de los datos.

---

## Un solo servidor

Válido solo cuando el modelo de datos lo exige (principalmente **grafos**: las relaciones cruzadas entre todos los nodos hacen imposible particionarlos). Para el resto de modelos, distribuir es la opción preferida.

---

## Sharding (Particionado)

### Qué mejora el sharding
- **Escalabilidad horizontal**: más datos → más shards. La capacidad escala linealmente.
- **Lecturas y escrituras**: al distribuir los datos, las operaciones se distribuyen entre nodos. Tanto lecturas como escrituras mejoran.
- **Localidad de datos**: los datos que se acceden juntos están en el mismo shard.

### Qué NO mejora el sharding
- **Disponibilidad**: si un shard falla, los datos de ese shard son inaccesibles. El sharding solo divide datos, no los replica. Para disponibilidad hay que combinar con replicación.

### Gestión del sharding
- Muchas BD NoSQL gestionan el sharding automáticamente (MongoDB con balanceador automático)
- La **shard key** determina qué shard almacena cada documento/fila
- Elegir bien la shard key es crítico para el rendimiento

### Tipos de sharding (ver [[mongodb]] para detalles)
- **Hashed Sharding**: hash de la clave → distribución uniforme. Malo para rangos.
- **Ranged Sharding**: rangos de clave → bueno para rangos de consulta. Riesgo de hotspots.

---

## Replicación maestro-esclavo (Master-Slave / Primary-Secondary)

### Arquitectura
- **Un nodo primario (maestro)**: acepta todas las escrituras
- **N nodos secundarios (esclavos)**: replican desde el primario, pueden servir lecturas

### Ventajas
- Escalado de lecturas: redirigir lecturas a secundarios
- Alta disponibilidad: si el primario cae, un secundario puede asumir el rol

### Desventajas y problemas
1. **Cuello de botella en escrituras**: el primario es el único nodo que acepta escrituras. Las escrituras intensivas saturan el primario.
2. **Inconsistencia de lectura**: si se lee de un secundario antes de que se haya sincronizado con el primario, se puede leer un dato obsoleto (replicación asíncrona).
3. **Read-your-writes**: si un cliente escribe en el primario y luego lee de un secundario, puede no ver su propia escritura.

### Conflictos en maestro-esclavo
- Los conflictos **escritura-lectura** son **transitorios**: cuando el secundario se ponga al día, desaparecen.
- No hay conflictos escritura-escritura (solo el primario acepta escrituras).

---

## Replicación peer-to-peer (P2P)

### Arquitectura
- Todos los nodos son iguales: todos pueden aceptar lecturas y escrituras
- No hay distinción entre primario y secundario
- Cada escritura se propaga a todos los nodos (o quórum)

### Ventajas
- Sin cuello de botella en escrituras
- Mayor disponibilidad: cualquier nodo puede aceptar operaciones

### Desventaja crítica: conflictos escritura-escritura
Si dos clientes escriben simultáneamente el mismo dato en nodos diferentes, se produce un **conflicto escritura-escritura**. Este conflicto es **permanente** (no transitorio como en maestro-esclavo) y hay que resolverlo.

Estrategias de resolución:
- **Last Write Wins**: la escritura más reciente gana (riesgo de pérdida de datos)
- **Merge**: combinar ambas versiones (depende del dominio)
- **Exponer al cliente**: dejar que la aplicación decida (e.g., Amazon Dynamo con "shopping cart")
- [[vector-stamp]]: detectar conflictos con vectores de versión

---

## Combinación: Sharding + Replicación

La arquitectura más robusta combina ambas técnicas:
- Sharding para escalar capacidad y throughput
- Replicación dentro de cada shard para disponibilidad y tolerancia a fallos

**MongoDB** usa exactamente esta combinación: cada shard es un **replica set** (primario + secundarios).

```
Cluster MongoDB:
  Shard 1: [Primary | Secondary | Secondary]
  Shard 2: [Primary | Secondary | Secondary]
  Shard 3: [Primary | Secondary | Secondary]
  Config Servers: [rsConfServer]
  Mongos routers: [mongos, mongos, mongos]
```

---

## Map-Reduce

### Paradigma
Map-Reduce es un modelo de programación para procesar grandes volúmenes de datos en paralelo sobre un cluster.

### Fase Map
- Se ejecuta **independientemente** sobre cada **agregado** (documento, fila, par clave-valor)
- Entrada: un agregado
- Salida: zero o más pares `(clave, valor)`
- Las funciones map son completamente independientes entre sí → paralelismo masivo

```javascript
// Ejemplo: contar géneros de películas
function map(pelicula) {
  for (genero of pelicula.generos) {
    emit(genero, 1);
  }
}
```

### Fase Shuffle
- El framework agrupa automáticamente todos los pares `(clave, valor)` por clave
- Cada clave va a un único reducer
- Permite que los reducers trabajen en paralelo sobre claves distintas

### Fase Reduce
- Se ejecuta **por cada clave** única
- Entrada: una clave + lista de todos los valores con esa clave
- Salida: un valor agregado

```javascript
function reduce(genero, valores) {
  return valores.reduce((acc, v) => acc + v, 0);
}
// genero "Action" → reduce([1,1,1,1,1]) → 5
```

### Combiner (optimización)
- Un combiner es un **reducer que se aplica localmente** en el nodo map, antes de enviar datos por red
- Reduce la cantidad de datos transferidos durante el shuffle
- Solo se puede usar cuando la función reduce es **conmutativa y asociativa** (no todas las operaciones son combinables)
- Ejemplo: suma (combinable), media aritmética (NO combinable directamente)

### Pipeline de Map-Reduce
Para cálculos complejos que requieren múltiples pasadas:
```
Datos → MapReduce1 → resultado1 → MapReduce2 → resultado2 → ...
```

### Map-Reduce incremental
Para procesar nuevos datos sin reprocesar todo el dataset. Hay dos casos según si el reducer es combinable:

**Caso 1 — Reducer combinable (función aditiva)**: la fase Map es fácil de ejecutar incrementalmente porque cada tarea Map es independiente. Si además el reducer es combinable (conmutativo y asociativo, como sumas o conteos), basta con ejecutar Map solo sobre los datos nuevos y combinar el resultado parcial con el resultado anterior.

**Caso 2 — Reducer no combinable (red de dependencias)**: cuando el resultado de un Reduce depende de muchos resultados Map intermedios (ejemplo: pagerank, joins complejos), no se puede actualizar incrementalmente de forma sencilla. Hay que mantener un grafo de dependencias entre tareas y recalcular solo las partes del grafo que dependen de los datos que cambiaron. Esto es mucho más complejo y costoso.

- Importante para sistemas en tiempo real o con actualizaciones frecuentes
- La elección del modelo incremental depende de si la función de reducción es o no combinable

### Map-Reduce para indexado distribuido
El caso de uso canónico de Map-Reduce en RI:
- **Map**: para cada par `(docID, texto)`, tokenizar y emitir `(término, docID)`
- **Shuffle**: agrupar por término
- **Reduce**: para cada término, construir la posting list `término → [docID1, docID2, ...]`

Ver también: [[sharding]], [[replicacion]], [[nosql]], [[mongodb]]
