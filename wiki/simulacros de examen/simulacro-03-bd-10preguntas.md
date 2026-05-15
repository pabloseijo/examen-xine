---
title: Simulacro 03 — BD 10 preguntas (70% teoría / 30% práctica)
type: simulacro
fecha: 2026-05-15
temas: [CAP, BASE, quórums, agregados, write-write, sharding, esquema, MongoDB, Neo4J]
---

# Simulacro 03 — Bases de Datos (10 preguntas)

**Formato**: 7 teoría + 3 práctica (ratio del examen real según el profesor)
**Tiempo orientativo: 40 minutos**

---

## TEORÍA

**1.** Define los tres componentes del Teorema CAP. ¿Por qué en un cluster real solo se puede elegir entre CP y AP, y nunca CA?

**2.** Explica la diferencia entre Soft State y Eventual Consistency dentro del modelo BASE. ¿Qué significa que el estado cambie sin que haya escrituras nuevas?

**3.** Un sistema tiene N=4, W=3, R=2. ¿Qué garantías ofrece? ¿Qué pasa si bajamos W a 1?

**4.** ¿Qué es un agregado en NoSQL? ¿Qué modelos usan el concepto de agregado y cuál no? ¿Por qué ese último no lo usa?

**5.** Explica el problema write-write en replicación peer-to-peer. ¿Qué es el Vector Stamp y qué hace exactamente — y qué NO hace?

**6.** ¿Qué tres criterios debe cumplir una buena shard key? Pon un ejemplo de shard key mala para cada criterio y explica qué problema genera.

**7.** Un desarrollador dice: "En NoSQL no hay esquema, así que no hay que preocuparse por la estructura de los datos." ¿Qué tiene de incorrecto esta afirmación? ¿Qué es schema-on-read?

---

## PRÁCTICA

**8.** Explica qué hace cada parte de este comando y para qué sirve dentro del cluster:

```javascript
rs.initiate({
  _id: "rsConfigServer",
  configsvr: true,
  members: [{ _id: 0, host: "localhost:27019" }]
})
```

**9.** Dado este comando Cypher:

```cypher
MERGE (p:Person { name: "Laura" })
ON CREATE SET p.born = 1995
RETURN p
```

¿Qué hace `MERGE`? ¿Qué diferencia hay con `CREATE`? ¿Cuándo se ejecuta `ON CREATE SET`?

**10.** Explica qué hace este pipeline de agregación de MongoDB y qué devuelve:

```javascript
db.pedidos.aggregate([
  { $match: { estado: "entregado" } },
  { $group: { _id: "$cliente_id", total: { $sum: "$importe" } } },
  { $sort: { total: -1 } },
  { $limit: 5 }
])
```

---

## Soluciones

> Completa el simulacro antes de leer esto.

<details>
<summary>Ver soluciones</summary>

### P1 — Teorema CAP
**C** — todos los nodos ven los mismos datos al mismo tiempo. **A** — cada petición recibida por un nodo que no falla DEBE ser respondida. **P** — el sistema funciona aunque haya una partición de red.

En un cluster real siempre existe riesgo de partición de red — no se puede eliminar físicamente. Por tanto P no es opcional, es una condición del entorno que el sistema debe asumir. Una vez P es obligatorio, ante una partición solo puedes elegir entre seguir respondiendo con datos posiblemente obsoletos (AP) o dejar de responder hasta resolver la partición (CP). CA solo es posible con un único servidor sin red entre nodos.

### P2 — Soft State vs Eventual Consistency
**Eventual Consistency**: si dejas de escribir, con el tiempo todos los nodos convergen al mismo valor. Describe el destino final.

**Soft State**: el estado puede cambiar aunque nadie esté escribiendo datos nuevos. Cuando escribes en el nodo A, los nodos B y C tienen el valor viejo y van a cambiar cuando reciban la propagación — sin que ningún cliente escriba nada nuevo. El sistema está "en movimiento" internamente por la propagación asíncrona ya en curso.

Soft State describe el proceso. Eventual Consistency describe el resultado.

### P3 — Quórums N=4, W=3, R=2
- **R + W = 5 > 4** → se garantiza leer la escritura más reciente ✅
- **W = 3 > N/2 = 2** → se evitan conflictos de escritura ✅

Si W baja a 1:
- **W = 1, no > N/2 = 2** → pueden coexistir dos escrituras con quórum simultáneamente → conflictos W-W posibles ❌
- **R + W = 3, no > 4** → no se garantiza leer la escritura más reciente → consistencia eventual ❌

### P4 — Agregado
Unidad natural de datos que se trata como un todo para dos cosas: manipulación atómica (se lee y escribe entero) y distribución (vive en un único nodo del cluster).

- Clave-valor → el par clave-valor es el agregado
- Documentos → el documento es el agregado
- Familia de columnas → la fila completa es el agregado
- **Grafos → no usa agregados**: las relaciones cruzan entidades libremente. Para responder una consulta de traversal hay que saltar entre nodos siguiendo aristas que pueden apuntar a cualquier entidad del grafo. No existe unidad aislable que pueda vivir en un solo nodo sin romper las relaciones.

### P5 — Write-write y Vector Stamp
En P2P todos los nodos aceptan escrituras. Si dos clientes escriben valores distintos sobre el mismo dato en nodos diferentes al mismo tiempo, hay un **conflicto W-W permanente** — no se resuelve solo.

**Vector Stamp**: array con un contador por nodo (`[n1:3, n2:5, n3:1]`). Cada escritura incrementa el contador del nodo que escribe. Para comparar: A es más reciente que B si todos los contadores de A ≥ B y al menos uno es estrictamente mayor. Si dos versiones tienen valores mayores en posiciones distintas → **conflicto detectado**.

**Lo que NO hace**: no resuelve el conflicto. Solo lo detecta. La resolución depende de la aplicación (last-write-wins, merge, exponer al usuario).

### P6 — Criterios shard key
1. **Alta cardinalidad**: muchos valores distintos → distribución uniforme. Ejemplo malo: campo booleano `activo` (solo 2 valores → máximo 2 shards útiles).
2. **Baja frecuencia de valores extremos**: ningún valor concentra demasiados documentos. Ejemplo malo: campo `pais` si el 90% de usuarios son de España → hotspot en el shard de España.
3. **No monotónica**: que no crezca siempre en la misma dirección. Ejemplo malo: `timestamp` o `ObjectId` → todos los documentos nuevos van siempre al mismo shard → hotspot en escrituras.

### P7 — Schema-on-read
La afirmación es incorrecta. En NoSQL no hay esquema en la base de datos, pero el esquema existe — vive en la aplicación. **Schema-on-read** significa que la BD acepta cualquier documento sin validar su estructura, y es la aplicación quien interpreta y valida los datos al leerlos.

El riesgo: nadie impide que entren documentos con campos mal nombrados, tipos incorrectos o campos obligatorios ausentes. La **integridad de los datos pasa a ser responsabilidad de la aplicación**, no del motor de BD.

### P8 — rs.initiate Config Server
- `rs.initiate({...})` — inicializa un replica set con la configuración indicada
- `_id: "rsConfigServer"` — nombre del replica set
- `configsvr: true` — indica que este replica set actúa como **Config Server**: almacena los metadatos del cluster (qué chunks existen, en qué shard está cada uno, qué shards hay)
- `members: [{ _id: 0, host: "localhost:27019" }]` — un único miembro en localhost:27019

### P9 — MERGE, CREATE, ON CREATE SET
**MERGE**: busca si ya existe un nodo `Person` con `name: "Laura"`. Si existe lo devuelve. Si no existe lo crea. Es un "busca o crea" — nunca genera duplicados.

**Diferencia con CREATE**: CREATE siempre crea un nodo nuevo, exista o no uno igual. Dos ejecuciones de CREATE producen dos nodos duplicados. Con MERGE siempre hay uno solo.

**ON CREATE SET**: solo se ejecuta cuando MERGE no encontró el nodo y tuvo que crearlo. Si el nodo ya existía, `p.born = 1995` no se ejecuta — el nodo existente no se modifica.

### P10 — Pipeline de agregación
1. `$match { estado: "entregado" }` — filtra solo los pedidos con estado "entregado"
2. `$group { _id: "$cliente_id", total: { $sum: "$importe" } }` — agrupa por cliente y suma todos sus importes
3. `$sort { total: -1 }` — ordena de mayor a menor total
4. `$limit: 5` — devuelve solo los 5 primeros

**Resultado**: los 5 clientes que más han gastado en pedidos entregados, ordenados de mayor a menor gasto.

</details>
