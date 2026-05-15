---
title: Simulacro 05 — BD 10 preguntas (70% teoría / 30% práctica)
type: simulacro
fecha: 2026-05-15
temas: [ACID-BASE, CAP ejemplos, durabilidad replicación, modelado grafos, Map-Reduce, schema-less, readPref, MongoDB, Neo4J]
---

# Simulacro 05 — Bases de Datos (10 preguntas)

**Formato**: 7 teoría + 3 práctica (ratio del examen real según el profesor)
**Tiempo orientativo: 40 minutos**

---

## TEORÍA

**1.** Compara ACID y BASE punto por punto. ¿Por qué los sistemas NoSQL adoptan BASE en lugar de ACID? ¿Es una decisión técnica o una decisión de diseño?

**2.** Clasifica estos sistemas según el Teorema CAP y justifica cada uno: MongoDB, Cassandra, PostgreSQL (sin cluster). ¿Qué implica para cada uno ante una partición de red?

**3.** En replicación, un sistema puede confirmar la escritura al cliente tras escribir en 1 nodo o tras escribir en N nodos. Explica el trade-off entre durabilidad, rendimiento y disponibilidad en cada caso.

**4.** ¿Cuándo usarías un modelo de grafos en lugar de un modelo de documentos? Da un ejemplo concreto donde el modelo de documentos falle y el de grafos resuelva el problema de forma natural.

**5.** Explica qué es Map-Reduce. ¿Qué hacen las fases Map, Shuffle y Reduce? ¿Para qué se usa en el contexto de bases de datos distribuidas?

**6.** Un equipo argumenta que usar NoSQL schema-less acelera el desarrollo porque "nunca hay que migrar nada". ¿Es correcto? Explica qué ventajas reales ofrece y qué complejidad introduce.

**7.** ¿Qué es el Optimistic Offline Lock? ¿Cómo se implementa con marcas de versión? ¿Qué pasa cuando se detecta un conflicto?

---

## PRÁCTICA

**8.** Explica qué hace cada parte de este bloque de comandos y en qué orden deben ejecutarse:

```javascript
db.productos.createIndex({ categoria: 1 })

sh.enableSharding("tienda")

sh.shardCollection("tienda.productos", { categoria: 1 })
```

¿Por qué `createIndex` debe ejecutarse antes de `shardCollection`?

**9.** Dado este comando:

```javascript
db.getMongo().setReadPref("secondaryPreferred")
```

¿Qué hace? ¿En qué situación es útil? ¿Qué riesgo introduce respecto a la consistencia?

**10.** Explica qué hace este pipeline y qué devuelve:

```javascript
db.peliculas.aggregate([
  { $match: { año: { $gte: 2000 } } },
  { $unwind: "$generos" },
  { $group: { _id: "$generos", total: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 5 }
])
```

¿Qué hace `$unwind` exactamente? ¿Qué pasaría si lo eliminaras?

---

## Soluciones

> Completa el simulacro antes de leer esto.

<details>
<summary>Ver soluciones</summary>

### P1 — ACID vs BASE
| Propiedad | ACID | BASE |
|-----------|------|------|
| Atomicity | Sí — todo o nada | No garantizada entre agregados |
| Consistency | Estado siempre válido | Consistencia eventual |
| Isolation | Transacciones aisladas | Sin aislamiento fuerte |
| Durability | Confirmada = persiste | Depende de W y réplicas |

NoSQL adopta BASE porque en un sistema distribuido mantener ACID requiere coordinación costosa (locks distribuidos, two-phase commit) que destruye la escalabilidad y disponibilidad. Es una decisión de diseño consciente: se sacrifica consistencia fuerte a cambio de escalabilidad horizontal y disponibilidad.

### P2 — CAP: MongoDB, Cassandra, PostgreSQL
- **MongoDB → CP**: ante una partición, el primario deja de aceptar escrituras si no puede confirmar con la mayoría del replica set. Prefiere ser consistente a estar disponible.
- **Cassandra → AP**: ante una partición, todos los nodos siguen respondiendo aunque puedan devolver datos obsoletos. Prefiere disponibilidad a consistencia.
- **PostgreSQL sin cluster → CA**: sistema centralizado (un solo nodo). Sin red entre nodos no hay riesgo de partición. Garantiza C y A pero no tolera particiones — si hay red, no es un sistema distribuido.

### P3 — Durabilidad vs rendimiento vs disponibilidad
**Confirmar tras 1 nodo**:
- Rendimiento: máximo (no hay espera)
- Durabilidad: baja (si el nodo falla antes de replicar, se pierde la escritura)
- Disponibilidad: alta (no depende de que otros nodos respondan)

**Confirmar tras N nodos**:
- Rendimiento: bajo (hay que esperar a que N nodos confirmen)
- Durabilidad: máxima (N copias existen antes de confirmar)
- Disponibilidad: baja (si alguna réplica falla, la escritura falla)

El trade-off es siempre el mismo: más durabilidad = menos rendimiento y disponibilidad.

### P4 — Grafos vs Documentos
Usar grafos cuando los datos son altamente relacionales y las consultas son traversals: "amigos de amigos", "ruta más corta entre A y B", "¿quién influye en quién?".

Ejemplo donde documentos fallan: red social donde necesitas encontrar todos los amigos de segundo grado de un usuario. En documentos tendrías que hacer múltiples consultas encadenadas (buscar amigos, luego buscar los amigos de cada amigo). En grafos es una sola consulta de traversal: `MATCH (u:User)-[:FRIEND*2]->(f:User)`.

### P5 — Map-Reduce
Modelo de programación para procesar grandes volúmenes de datos en paralelo.

- **Map**: se ejecuta independientemente sobre cada documento. Emite pares `(clave, valor)`.
- **Shuffle**: el framework agrupa automáticamente todos los pares por clave.
- **Reduce**: se ejecuta por cada clave única. Recibe todos los valores de esa clave y los agrega en un resultado.

En BD distribuidas se usa para cálculos analíticos sobre grandes datasets: construir índices invertidos, contar ocurrencias, calcular estadísticas. Permite paralelismo masivo porque cada tarea Map es independiente.

### P6 — Schema-less: ventajas reales y complejidad
La afirmación es parcialmente incorrecta. Las ventajas reales son:
- No hay `ALTER TABLE` bloqueante sobre millones de filas
- Migración incremental: nuevos documentos con nuevo formato, viejos con el antiguo, conviven
- Polimorfismo natural: documentos de la misma colección pueden tener estructura diferente

La complejidad que introduce:
- La integridad de datos recae en la aplicación
- El código debe manejar múltiples versiones del esquema durante la transición
- Consultas ad-hoc sobre datos sin estructura predecible son más difíciles

"Nunca hay que migrar nada" es falso — la migración existe, solo se desplaza de la BD a la aplicación.

### P7 — Optimistic Offline Lock
Estrategia para gestionar conflictos de escritura sin bloquear recursos durante la lectura.

Funcionamiento:
1. Al leer, se guarda la versión actual del dato (contador, timestamp, hash o UUID)
2. Al escribir, se comprueba que la versión del dato en la BD sigue siendo la misma que se leyó
3. Si nadie modificó el dato mientras tanto → escritura exitosa, se incrementa la versión
4. Si alguien lo modificó (versiones distintas) → conflicto detectado → se rechaza la escritura y se pide al cliente que reintente

Es "optimista" porque asume que los conflictos son raros — no bloquea, deja que ocurran y los detecta después. Contrario a la estrategia pesimista que bloquea antes de escribir.

### P8 — createIndex, enableSharding, shardCollection
Orden correcto:
1. `db.productos.createIndex({ categoria: 1 })` — crea un índice sobre el campo `categoria`
2. `sh.enableSharding("tienda")` — habilita el sharding en la base de datos "tienda"
3. `sh.shardCollection("tienda.productos", { categoria: 1 })` — parte la colección usando `categoria` como shard key

`createIndex` debe ejecutarse **antes** de `shardCollection` porque MongoDB requiere que la shard key esté indexada antes de poder particionar la colección. Sin el índice, `shardCollection` falla.

### P9 — setReadPref secondaryPreferred
Configura la preferencia de lectura del cliente a `secondaryPreferred`: el cliente lee de un secundario si hay alguno disponible, y solo cae al primario si no hay secundarios.

**Útil para**: distribuir la carga de lecturas entre secundarios, reduciendo la presión sobre el primario.

**Riesgo de consistencia**: los secundarios replican de forma asíncrona — pueden tener datos ligeramente obsoletos. Si lees de un secundario justo después de una escritura, puedes no ver tu propia escritura (problema read-your-writes).

### P10 — Pipeline con $unwind
1. `$match { año: { $gte: 2000 } }` — filtra películas del año 2000 en adelante
2. `$unwind "$generos"` — descompone el array `generos` de cada película en documentos separados, uno por género. Una película con 3 géneros se convierte en 3 documentos.
3. `$group { _id: "$generos", total: { $sum: 1 } }` — agrupa por género y cuenta cuántas películas tienen ese género
4. `$sort { total: -1 }` — ordena de más a menos
5. `$limit 5` — devuelve los 5 primeros

**Resultado**: los 5 géneros más frecuentes en películas desde el año 2000.

**Sin $unwind**: el `$group` recibiría el array entero como clave — `["acción", "thriller"]` sería una clave distinta de `["acción"]`. No se podrían contar géneros individuales.

</details>
