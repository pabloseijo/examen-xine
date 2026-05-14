---
title: Neo4J y Cypher — Práctica completa
type: concepto
fuentes: [Neo4J.txt, Ejercicio Neo4J.pdf]
última_actualización: 2026-05-14
---

## Estructura del grafo

**Nodos**:

- `(p:Pelicula)` — id, titulo, presupuesto, fechaEmision, ingresos, duracion
- `(p:Persona)` — id, nombre

**Relaciones**:

- `(persona)-[:ACTUO_EN {personaje, orden}]->(pelicula)`
- `(persona)-[:TRABAJO_EN {trabajo, departamento}]->(pelicula)`
- `(persona)-[:DIRIGE]->(pelicula)` — creada en la consulta 1 a partir de TRABAJO_EN donde trabajo="Director"

---

## Paso 0 — Constraints e índices

Se crean **antes** de importar datos. Los constraints garantizan unicidad; los índices aceleran las búsquedas por título/nombre.

```cypher
CREATE CONSTRAINT idPelicula FOR (p:Pelicula) REQUIRE p.id IS UNIQUE;
CREATE CONSTRAINT idPersona  FOR (p:Persona)  REQUIRE p.id IS UNIQUE;

CREATE INDEX FOR (p:Pelicula) ON (p.titulo);
CREATE INDEX FOR (p:Persona)  ON (p.nombre);
```

- `REQUIRE p.id IS UNIQUE` → evita duplicados al hacer `CREATE` en las importaciones
- El índice sobre `titulo` y `nombre` acelera los `MATCH` por esas propiedades en las consultas

---

## Importaciones con JDBC

Estructura general de cada importación:

```cypher
WITH "jdbc:postgresql://xine-grei-pgsql/xine?user=alumnobd&password=pwalumnobd" AS url
CALL apoc.load.jdbc(url, "SELECT ...") YIELD row
MATCH/CREATE ...
```

### 1. Importar 2000 películas

```cypher
WITH "jdbc:postgresql://xine-grei-pgsql/xine?user=alumnobd&password=pwalumnobd" AS url
CALL apoc.load.jdbc(url,
  "select id, titulo, presupuesto, fecha_emision, ingresos, duracion
   from peliculas
   order by ingresos desc, id
   limit 2000") YIELD row
CREATE (p:Pelicula {
  id:          row.id,
  titulo:      row.titulo,
  presupuesto: row.presupuesto,
  fechaEmision: row.fecha_emision,
  ingresos:    row.ingresos,
  duracion:    row.duracion
});
```

### 2. Importar personas (reparto + personal, sin duplicados)

La SQL une reparto y personal con `UNION ALL` y elimina duplicados con `DISTINCT`. Usa `json_array_elements` para descomponer los arrays JSON de la tabla `creditos`.

```cypher
WITH "jdbc:postgresql://xine-grei-pgsql/xine?user=alumnobd&password=pwalumnobd" AS url
CALL apoc.load.jdbc(url,
  "select distinct id,
          first_value(nombre) over (partition by id order by nombre) as nombre
   from (
     select cast(e->>'id' as integer) as id,
            e->>'name' as nombre
     from creditos, json_array_elements(personal) as p(e)
     where pelicula in (
       select id from peliculas order by ingresos desc, id limit 2000
     )
     union all
     select cast(e->>'id' as integer) as id,
            e->>'name' as nombre
     from creditos, json_array_elements(reparto) as r(e)
     where pelicula in (
       select id from peliculas order by ingresos desc, id limit 2000
     )
   ) as t") YIELD row
CREATE (p:Persona { id: row.id, nombre: row.nombre });
```

### 3. Crear relaciones ACTUO_EN

```cypher
WITH "jdbc:postgresql://xine-grei-pgsql/xine?user=alumnobd&password=pwalumnobd" AS url
CALL apoc.load.jdbc(url,
  "select cast(e->>'id' as integer) as persona,
          e->>'character' as personaje,
          cast(e->>'order' as integer) as orden,
          pelicula
   from creditos, json_array_elements(reparto) as r(e)
   where pelicula in (
     select id from peliculas order by ingresos desc, id limit 2000
   )") YIELD row
MATCH (pelicula:Pelicula {id: row.pelicula})
MATCH (persona:Persona   {id: row.persona})
CREATE (persona)-[:ACTUO_EN { personaje: row.personaje, orden: row.orden }]->(pelicula);
```

### 4. Crear relaciones TRABAJO_EN

```cypher
WITH "jdbc:postgresql://xine-grei-pgsql/xine?user=alumnobd&password=pwalumnobd" AS url
CALL apoc.load.jdbc(url,
  "select cast(e->>'id' as integer) as persona,
          e->>'job' as trabajo,
          e->>'department' as departamento,
          pelicula
   from creditos, json_array_elements(personal) as p(e)
   where pelicula in (
     select id from peliculas order by ingresos desc, id limit 2000
   )") YIELD row
MATCH (pelicula:Pelicula {id: row.pelicula})
MATCH (persona:Persona   {id: row.persona})
CREATE (persona)-[:TRABAJO_EN { trabajo: row.trabajo, departamento: row.departamento }]->(pelicula);
```

---

## Consultas de la práctica

### 1. CREATE relación DIRIGE a partir de TRABAJO_EN

```cypher
MATCH (p:Persona)-[t:TRABAJO_EN]->(pelicula:Pelicula)
WHERE t.trabajo = "Director"
CREATE (p)-[:DIRIGE]->(pelicula);
```

- Busca todas las relaciones TRABAJO_EN donde trabajo = "Director"
- Crea una nueva relación DIRIGE directamente en el grafo (más directa para consultar)

---

### 2. MATCH básico con filtro en relación

```cypher
MATCH (persona:Persona)-[r:ACTUO_EN]->(pelicula:Pelicula {titulo:"Star Wars"})
RETURN r.orden AS orden,
       r.personaje AS personaje,
       persona.nombre AS actor
ORDER BY r.orden;
```

- `[r:ACTUO_EN]` captura la relación en la variable r para acceder a sus propiedades
- `{titulo: "Star Wars"}` filtra el nodo por propiedad directamente en el MATCH

---

### 3. Calcular campo nuevo y ordenar

```cypher
MATCH (p:Pelicula)
RETURN p.titulo      AS titulo,
       p.presupuesto AS presupuesto,
       p.ingresos    AS ingresos,
       (p.ingresos - p.presupuesto) AS beneficio
ORDER BY beneficio DESC
LIMIT 10;
```

- Se pueden calcular campos directamente en el RETURN
- `LIMIT` siempre va al final

---

### 4. UNION — combinar dos consultas

```cypher
MATCH (p:Persona {nombre:"Quentin Tarantino"})-[:ACTUO_EN]->(pelicula:Pelicula)
RETURN pelicula.titulo       AS titulo,
       pelicula.fechaEmision AS fechaEmision,
       pelicula.presupuesto  AS presupuesto,
       pelicula.ingresos     AS ingresos,
       "actuo"               AS participacion
UNION
MATCH (p:Persona {nombre:"Quentin Tarantino"})-[:DIRIGE]->(pelicula:Pelicula)
RETURN pelicula.titulo       AS titulo,
       pelicula.fechaEmision AS fechaEmision,
       pelicula.presupuesto  AS presupuesto,
       pelicula.ingresos     AS ingresos,
       "dirigio"             AS participacion
ORDER BY fechaEmision;
```

- `UNION` elimina duplicados (como en SQL). `UNION ALL` mantiene duplicados
- Ambas partes deben devolver las **mismas columnas con los mismos nombres**
- El `ORDER BY` final se aplica al resultado combinado

---

### 5. collect() y count() — agrupar

```cypher
MATCH (persona:Persona)-[t:TRABAJO_EN]->(pelicula:Pelicula {titulo:"The Godfather"})
RETURN t.departamento AS departamento,
       count(*)       AS numPersonas,
       collect({nombre: persona.nombre, trabajo: t.trabajo}) AS personal
ORDER BY numPersonas DESC, departamento;
```

- `count(*)` cuenta filas del grupo
- `collect({campo: valor})` agrupa objetos en una lista (equivale a array de mapas)

---

### 6. WITH + doble OPTIONAL MATCH

```cypher
MATCH (steven:Persona {nombre:"Steven Spielberg"})-[t:TRABAJO_EN]->(pelicula:Pelicula)
WITH pelicula, t.trabajo AS trabajo
ORDER BY pelicula.ingresos DESC
LIMIT 1
OPTIONAL MATCH (actor:Persona)-[:ACTUO_EN]->(pelicula)
WITH pelicula, trabajo, count(DISTINCT actor) AS numActores
OPTIONAL MATCH (trabajador:Persona)-[:TRABAJO_EN]->(pelicula)
WHERE NOT (trabajador)-[:ACTUO_EN]->(pelicula)
RETURN pelicula.titulo    AS titulo,
       trabajo,
       pelicula.presupuesto AS presupuesto,
       pelicula.ingresos    AS ingresos,
       numActores,
       count(DISTINCT trabajador) AS numTrabajadores;
```

- `WITH ... LIMIT 1` selecciona solo la película con más ingresos y la pasa al siguiente paso
- Primer `OPTIONAL MATCH` cuenta actores (devuelve null si no hay, no elimina el nodo)
- `WITH` intermedio "congela" numActores antes del segundo OPTIONAL MATCH
- Segundo `OPTIONAL MATCH` con `WHERE NOT (patrón)` excluye a quienes también estén en el reparto

---

### 7. Traversal de dos saltos

```cypher
MATCH (:Persona {nombre:"Marlon Brando"})-[:ACTUO_EN]->(:Pelicula)<-[:DIRIGE]-(director:Persona)
MATCH (director)-[:DIRIGE]->(:Pelicula)<-[:ACTUO_EN]-(actor:Persona)
RETURN DISTINCT actor.nombre AS nombre
ORDER BY nombre;
```

- Primer MATCH: directores que dirigieron a Marlon Brando
- Segundo MATCH: actores dirigidos por esos directores (dos saltos)
- `DISTINCT` elimina duplicados en el resultado

---

### 8. WITH como HAVING — filtrar sobre agregaciones

```cypher
MATCH (director:Persona)-[:DIRIGE]->(pelicula:Pelicula)
WITH pelicula,
     collect(DISTINCT director.nombre) AS directores,
     count(DISTINCT director)          AS numDirectores
WHERE numDirectores > 1
RETURN pelicula.titulo AS titulo,
       numDirectores,
       directores
ORDER BY numDirectores DESC, titulo;
```

- `WITH` es como un "pipe": pasa el resultado parcial al siguiente paso
- `WHERE` después de `WITH` filtra sobre valores calculados (equivale a `HAVING` en SQL)
- Solo las variables del `WITH` están disponibles después

---

### 9. CALL — subconsulta por fila

```cypher
MATCH (persona:Persona)
CALL {
  WITH persona
  MATCH (persona)-[:ACTUO_EN]->(pelicula:Pelicula)
  RETURN pelicula, "reparto" AS rol
  UNION ALL
  WITH persona
  MATCH (persona)-[t:TRABAJO_EN]->(pelicula:Pelicula)
  RETURN pelicula, t.trabajo AS rol
}
WITH persona, pelicula, collect(rol) AS roles, count(*) AS numRoles
RETURN persona.id     AS id,
       persona.nombre AS nombre,
       pelicula.titulo AS titulo,
       numRoles,
       roles
ORDER BY numRoles DESC, nombre, titulo
LIMIT 10;
```

- `CALL { ... }` ejecuta una subconsulta **por cada fila** del contexto exterior
- `WITH persona` dentro del CALL pasa la variable exterior a la subconsulta
- `UNION ALL` dentro del CALL mantiene duplicados (una persona puede tener varios trabajos en la misma película)

---

### 10. UNION para grafo de colaboradores

```cypher
MATCH (qt:Persona {nombre:"Quentin Tarantino"})-[:DIRIGE]->(:Pelicula)<-[:ACTUO_EN]-(actor1:Persona)
RETURN DISTINCT actor1.nombre AS nombre
UNION
MATCH (qt:Persona {nombre:"Quentin Tarantino"})-[:DIRIGE]->(:Pelicula)<-[:ACTUO_EN]-(actorDirigido:Persona)
MATCH (actorDirigido)-[:ACTUO_EN]->(:Pelicula)<-[:ACTUO_EN]-(actor2:Persona)
WHERE actor2 <> actorDirigido
RETURN DISTINCT actor2.nombre AS nombre
ORDER BY nombre;
```

- Primera parte: actores que han actuado en películas **dirigidas** por Tarantino
- Segunda parte: actores que han actuado junto a alguien dirigido por Tarantino (colaboradores de segundo grado)
- `WHERE actor2 <> actorDirigido` evita que la persona se devuelva a sí misma
- `UNION` (sin ALL) elimina duplicados entre las dos mitades

---

### MERGE — lo que cayó en el examen 2025

```cypher
MERGE (m:Movie { title:"Cloud Atlas" })
ON CREATE SET m.released = 2012
RETURN m;
```

| Parte | Qué hace |
|-------|----------|
| `MERGE (m:Movie { title:"Cloud Atlas" })` | Busca si existe un nodo Movie con title="Cloud Atlas". Si existe, lo devuelve. Si no existe, lo crea. |
| `ON CREATE SET m.released = 2012` | Solo se ejecuta si el nodo **se acaba de crear**. Si ya existía, esta línea se ignora. |
| `RETURN m` | Devuelve el nodo, tanto si era nuevo como si ya existía. |

**MERGE vs CREATE:**

| | `CREATE` | `MERGE` |
| - | -------- | ------- |
| Si el nodo ya existe | Crea uno nuevo (duplicado) | Devuelve el existente |
| Si el nodo no existe | Lo crea | Lo crea |
| Cuándo usarlo | Importaciones donde sabemos que no hay duplicados | Upsert — cuando no sabemos si ya existe |

**ON CREATE SET vs SET:**

- `ON CREATE SET` → solo si el nodo es nuevo
- `ON MATCH SET` → solo si el nodo ya existía
- `SET` → siempre, tanto si es nuevo como si ya existía

---

## Resumen de cláusulas clave

| Cláusula | Qué hace |
| -------- | -------- |
| `MATCH` | Busca patrones en el grafo |
| `OPTIONAL MATCH` | Como MATCH pero devuelve null si no hay coincidencia (LEFT JOIN) |
| `WHERE` | Filtra resultados |
| `WHERE NOT (patrón)` | Excluye nodos que tienen ese patrón de relación |
| `RETURN` | Devuelve resultados |
| `WITH` | Encadena pasos, permite filtrar sobre agregaciones (HAVING) |
| `CREATE` | Crea nodos o relaciones |
| `MERGE` | Crea solo si no existe ya |
| `ORDER BY` | Ordena (ASC por defecto, DESC para descendente) |
| `LIMIT` | Limita el número de resultados |
| `UNION` | Une dos consultas eliminando duplicados |
| `UNION ALL` | Une dos consultas manteniendo duplicados |
| `count(*)` | Cuenta filas |
| `count(DISTINCT x)` | Cuenta valores únicos |
| `collect(x)` | Agrupa valores en una lista |
| `CALL { }` | Subconsulta por fila |

---

## En el examen

- Leer una query Cypher y explicar qué hace paso a paso
- Distinguir `MERGE` vs `CREATE` (MERGE no duplica si ya existe)
- Explicar por qué se usan constraints e índices antes de importar
- El patrón `(a)-[:REL]->(b)` = relación dirigida de a hacia b
- Saber qué diferencia `UNION` de `UNION ALL`
- `WITH` como HAVING: filtrar después de agregar

## Relación con otros conceptos

- [[neo4j]] — conceptos generales de grafos
- [[mongodb-cluster-comandos]] — comandos equivalentes en MongoDB
