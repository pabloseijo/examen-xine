---
title: Simulacro 04 — BD 10 preguntas (70% teoría / 30% práctica)
type: simulacro
fecha: 2026-05-15
temas: [CAP, ACID-BASE, replicación, sharding, modelado, esquema, MongoDB, Neo4J]
---

# Simulacro 04 — Bases de Datos (10 preguntas)

**Formato**: 7 teoría + 3 práctica (ratio del examen real según el profesor)
**Tiempo orientativo: 40 minutos**

---

## TEORÍA

**1.** ¿Qué diferencia hay entre escalar verticalmente y escalar horizontalmente? ¿Por qué las bases de datos relacionales tienen problemas con el escalado horizontal?

**2.** Explica las cuatro propiedades ACID. ¿Cuál de ellas es la más difícil de mantener en un sistema distribuido con varios nodos y por qué?

**3.** Un sistema de mensajería en tiempo real usa replicación maestro-esclavo. Un usuario envía un mensaje y medio segundo después lo busca en su historial — no lo ve. ¿Qué ha pasado? ¿Cómo se puede solucionar sin cambiar a peer-to-peer?

**4.** Explica qué es el hashed sharding y el ranged sharding. Una aplicación necesita hacer consultas del tipo `fecha >= "2025-01-01" AND fecha <= "2025-06-30"` frecuentemente. ¿Qué tipo de sharding conviene y por qué?

**5.** ¿Qué es el impedance mismatch? ¿Por qué es uno de los motivos principales para usar NoSQL?

**6.** Explica la diferencia entre replicación maestro-esclavo y peer-to-peer en términos de disponibilidad de escritura y tipo de conflictos que genera cada una.

**7.** ¿Qué es la ventana de inconsistencia en un sistema replicado? ¿De qué factores depende su duración? Pon un ejemplo de aplicación donde sea aceptable y otro donde no lo sea.

---

## PRÁCTICA

**8.** Explica qué hace cada parte de este comando:

```javascript
mongod --shardsvr --replSet rsShard2 --port 27021 --dbpath /data/shard2 --bind_ip localhost
```

¿Qué diferencia hay entre `--shardsvr` y `--configsvr`?

**9.** Dado este bloque Cypher:

```cypher
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE m.released > 2010
RETURN p.name, count(m) AS peliculas
ORDER BY peliculas DESC
LIMIT 3
```

Explica qué hace cada cláusula y qué devuelve la consulta completa.

**10.** Explica qué hace este pipeline y qué devuelve:

```javascript
db.empleados.aggregate([
  { $match: { departamento: "ingeniería" } },
  { $group: { _id: "$ciudad", media_salario: { $avg: "$salario" } } },
  { $sort: { media_salario: -1 } },
  { $limit: 3 }
])
```

---

## Soluciones

> Completa el simulacro antes de leer esto.

<details>
<summary>Ver soluciones</summary>

### P1 — Escalado vertical vs horizontal
**Vertical**: añadir más recursos a la misma máquina (más RAM, CPU más rápida). Tiene límite físico y es caro — llega un punto en que no hay máquina suficientemente potente o el coste es prohibitivo.

**Horizontal**: añadir más máquinas al cluster. Sin límite teórico, usa hardware commodity barato.

Los RDBMS tienen problemas con el escalado horizontal porque están diseñados para garantizar ACID en una única máquina. Distribuir transacciones entre nodos (coordinación de commits, locks distribuidos) es extremadamente costoso y degrada el rendimiento. NoSQL nace diseñado para el cluster desde el inicio, relajando consistencia a cambio de escalabilidad.

### P2 — ACID y sistema distribuido
- **Atomicity**: todo o nada — la transacción se completa entera o se deshace
- **Consistency**: la BD pasa de un estado válido a otro estado válido
- **Isolation**: las transacciones concurrentes no se interfieren
- **Durability**: una vez confirmada, la transacción persiste aunque haya fallos

La más difícil en sistemas distribuidos es **Isolation**: garantizar que transacciones concurrentes en nodos distintos no se interfieren requiere coordinación (locks distribuidos, protocolos de consenso como 2PC), lo que introduce latencia alta y reduce disponibilidad. Por eso NoSQL relaja el aislamiento y adopta consistencia eventual.

### P3 — Problema de lectura y solución
Ha ocurrido un problema de **read-your-writes**: el usuario escribió en el primario, luego leyó de un secundario que aún no se había sincronizado — la ventana de inconsistencia no había cerrado.

Solución sin cambiar a P2P: **sticky session** — el cliente siempre lee del mismo nodo secundario (o del primario). Como ese nodo tiene su escritura más reciente, la lectura siguiente sí la ve. En MongoDB se implementa con `readPreference: primaryPreferred` o con sesiones causales.

### P4 — Hashed vs Ranged y consultas de rango
**Hashed sharding**: aplica una función hash a la shard key → distribución uniforme entre shards. Malo para range queries porque el hash destruye el orden — hay que consultar todos los shards.

**Ranged sharding**: divide los valores por rangos → bueno para range queries porque todos los documentos de un rango están en el mismo shard o en pocos shards contiguos.

Para `fecha >= "2025-01-01" AND fecha <= "2025-06-30"` conviene **ranged sharding** sobre `fecha`: MongoDB sabe exactamente qué shards contienen ese rango y solo consulta esos.

Riesgo: si se usa timestamp como shard key es monotónica → hotspot en escrituras. Se puede mitigar con una shard key compuesta.

### P5 — Impedance mismatch
Las aplicaciones modernas trabajan con objetos, grafos y estructuras jerárquicas. El modelo relacional obliga a traducir todo a tablas planas. Esta traducción constante (mediante ORMs u código manual) es costosa, frágil y genera código boilerplate. NoSQL elimina esta fricción almacenando los datos en la misma forma en que la aplicación los usa (documentos JSON, grafos, pares clave-valor).

### P6 — Maestro-esclavo vs P2P
**Maestro-esclavo**:
- Disponibilidad de escritura: baja — solo el primario escribe → cuello de botella
- Conflictos: solo R-W (transitorio — desaparece cuando el secundario se sincroniza). No hay W-W.

**Peer-to-peer**:
- Disponibilidad de escritura: alta — cualquier nodo acepta escrituras → sin cuello de botella
- Conflictos: W-W (permanente — no se resuelve solo, requiere intervención de la aplicación)

### P7 — Ventana de inconsistencia
Período durante el cual distintos nodos pueden devolver valores distintos para el mismo dato, tras una escritura y antes de que se propague a todas las réplicas.

Depende de: velocidad de red, carga del sistema, frecuencia de sincronización entre nodos.

- **Aceptable**: contador de likes en una red social — no importa si dos usuarios ven valores distintos durante 1 segundo.
- **No aceptable**: saldo bancario — una transferencia debe ser visible inmediatamente en todos los nodos para evitar que el mismo dinero se gaste dos veces.

### P8 — --shardsvr vs --configsvr
El comando arranca un `mongod` como miembro del replica set `rsShard2` en el puerto 27021, actuando como **shard** del cluster (`--shardsvr`).

- `--shardsvr`: el nodo almacena **datos de usuario** — es uno de los shards del cluster
- `--configsvr`: el nodo almacena **metadatos del cluster** (qué chunks hay, en qué shard está cada uno) — es un Config Server

Son roles incompatibles: un nodo es shard o Config Server, nunca los dos.

### P9 — Consulta Cypher
- `MATCH (p:Person)-[:ACTED_IN]->(m:Movie)` — busca personas que tengan una relación ACTED_IN hacia una película
- `WHERE m.released > 2010` — filtra solo películas lanzadas después de 2010
- `RETURN p.name, count(m) AS peliculas` — devuelve el nombre de la persona y cuántas películas cumple la condición
- `ORDER BY peliculas DESC` — ordena de más a menos películas
- `LIMIT 3` — devuelve solo los 3 primeros

**Resultado**: los 3 actores que han actuado en más películas lanzadas después de 2010, ordenados de mayor a menor.

### P10 — Pipeline de agregación
1. `$match { departamento: "ingeniería" }` — filtra solo empleados de ingeniería
2. `$group { _id: "$ciudad", media_salario: { $avg: "$salario" } }` — agrupa por ciudad y calcula el salario medio de cada una
3. `$sort { media_salario: -1 }` — ordena de mayor a menor salario medio
4. `$limit: 3` — devuelve solo las 3 primeras

**Resultado**: las 3 ciudades con el salario medio más alto entre los empleados de ingeniería.

</details>
