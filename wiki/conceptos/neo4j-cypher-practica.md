---
title: Neo4J y Cypher — Práctica completa
type: concepto
fuentes: [Neo4J.txt, Ejercicio Neo4J.pdf]
última_actualización: 2026-05-14
---

# Neo4J y Cypher — Práctica completa

---

## Estructura del grafo

El grafo de la práctica tiene esta forma:

**Nodos**:
- `(p:Pelicula)` — con propiedades: id, titulo, presupuesto, fechaEmision, ingresos, duracion
- `(p:Persona)` — con propiedades: id, nombre

**Relaciones**:
- `(persona)-[:ACTUO_EN {personaje, orden}]->(pelicula)`
- `(persona)-[:TRABAJO_EN {trabajo, departamento}]->(pelicula)`
- `(persona)-[:DIRIGE]->(pelicula)` — creada a partir de TRABAJO_EN donde trabajo="Director"

---

## Importación con JDBC

Los datos vienen de PostgreSQL. La estructura general de cada importación es:

```cypher
WITH "jdbc:postgresql://host/bd?user=X&password=Y" AS url
CALL apoc.load.jdbc(url, "SELECT ...") YIELD row
MATCH/CREATE ...
```

### 1. Importar películas
```cypher
WITH "jdbc:postgresql://xine-grei-pgsql/xine?user=alumnobd&password=pwalumnobd" AS url
CALL apoc.load.jdbc(url,
  "select id, titulo, presupuesto, fecha_emision, ingresos, duracion
   from peliculas order by ingresos desc, id limit 2000") YIELD row
CREATE (p:Pelicula {
  id: row.id, titulo: row.titulo,
  presupuesto: row.presupuesto, fechaEmision: row.fecha_emision,
  ingresos: row.ingresos, duracion: row.duracion
});
```

### 2. Importar personas (reparto + personal, sin duplicados)
```cypher
-- La SQL une reparto y personal con UNION ALL y elimina duplicados con DISTINCT
CREATE (p:Persona { id: row.id, nombre: row.nombre });
```

### 3. Crear relaciones ACTUO_EN
```cypher
MATCH (pelicula:Pelicula {id: row.pelicula})
MATCH (persona:Persona {id: row.persona})
CREATE (persona)-[:ACTUO_EN { personaje: row.personaje, orden: row.orden }]->(pelicula);
```

### 4. Crear relaciones TRABAJO_EN
```cypher
MATCH (pelicula:Pelicula {id: row.pelicula})
MATCH (persona:Persona {id: row.persona})
CREATE (persona)-[:TRABAJO_EN { trabajo: row.trabajo, departamento: row.departamento }]->(pelicula);
```

---

## Patrones Cypher del examen

### MATCH básico con filtro en relación

```cypher
MATCH (persona:Persona)-[r:ACTUO_EN]->(pelicula:Pelicula {titulo: "Star Wars"})
RETURN r.orden AS orden, r.personaje AS personaje, persona.nombre AS actor
ORDER BY r.orden;
```
- `[r:ACTUO_EN]` — captura la relación en la variable r para acceder a sus propiedades
- `{titulo: "Star Wars"}` — filtra el nodo por propiedad directamente en el MATCH

---

### CREATE relación a partir de otra relación

```cypher
MATCH (p:Persona)-[t:TRABAJO_EN]->(pelicula:Pelicula)
WHERE t.trabajo = "Director"
CREATE (p)-[:DIRIGE]->(pelicula);
```
- Busca todas las relaciones TRABAJO_EN donde trabajo es "Director"
- Crea una nueva relación DIRIGE directamente en el grafo

---

### Calcular campo nuevo y ordenar

```cypher
MATCH (p:Pelicula)
RETURN p.titulo AS titulo,
       p.presupuesto AS presupuesto,
       p.ingresos AS ingresos,
       (p.ingresos - p.presupuesto) AS beneficio
ORDER BY beneficio DESC
LIMIT 10;
```
- Se pueden calcular campos directamente en el RETURN
- `LIMIT` siempre va al final

---

### UNION — combinar dos consultas

```cypher
MATCH (p:Persona {nombre:"Quentin Tarantino"})-[:ACTUO_EN]->(pelicula:Pelicula)
RETURN pelicula.titulo AS titulo, "actuo" AS participacion
UNION
MATCH (p:Persona {nombre:"Quentin Tarantino"})-[:DIRIGE]->(pelicula:Pelicula)
RETURN pelicula.titulo AS titulo, "dirigio" AS participacion
ORDER BY titulo;
```
- `UNION` elimina duplicados (como en SQL)
- `UNION ALL` mantiene duplicados
- Ambas partes deben devolver las mismas columnas con los mismos nombres
- El `ORDER BY` final se aplica al resultado combinado

---

### collect() y count() — agrupar

```cypher
MATCH (persona:Persona)-[t:TRABAJO_EN]->(pelicula:Pelicula {titulo:"The Godfather"})
RETURN t.departamento AS departamento,
       count(*) AS numPersonas,
       collect({nombre: persona.nombre, trabajo: t.trabajo}) AS personal
ORDER BY numPersonas DESC;
```
- `count(*)` — cuenta filas del grupo
- `count(DISTINCT x)` — cuenta valores únicos
- `collect(x)` — agrupa todos los valores en una lista
- `collect({campo: valor})` — agrupa objetos en una lista

---

### WITH — encadenar pasos

```cypher
MATCH (director:Persona)-[:DIRIGE]->(pelicula:Pelicula)
WITH pelicula,
     collect(DISTINCT director.nombre) AS directores,
     count(DISTINCT director) AS numDirectores
WHERE numDirectores > 1
RETURN pelicula.titulo AS titulo, numDirectores, directores
ORDER BY numDirectores DESC;
```
- `WITH` es como un "pipe" — pasa el resultado parcial al siguiente paso
- Permite aplicar `WHERE` sobre resultados calculados (como HAVING en SQL)
- Solo las variables mencionadas en `WITH` están disponibles después

---

### OPTIONAL MATCH — como LEFT JOIN

```cypher
MATCH (steven:Persona {nombre:"Steven Spielberg"})-[t:TRABAJO_EN]->(pelicula:Pelicula)
WITH pelicula, t.trabajo AS trabajo
ORDER BY pelicula.ingresos DESC
LIMIT 1
OPTIONAL MATCH (actor:Persona)-[:ACTUO_EN]->(pelicula)
RETURN pelicula.titulo, count(DISTINCT actor) AS numActores;
```
- `OPTIONAL MATCH` devuelve el nodo aunque no tenga la relación buscada (devuelve null)
- Sin OPTIONAL MATCH, si la relación no existe el nodo desaparece del resultado

---

### WHERE NOT — excluir patrones

```cypher
OPTIONAL MATCH (trabajador:Persona)-[:TRABAJO_EN]->(pelicula)
WHERE NOT (trabajador)-[:ACTUO_EN]->(pelicula)
RETURN count(DISTINCT trabajador) AS numTrabajadores;
```
- `WHERE NOT (patrón)` excluye nodos que cumplen ese patrón de relación

---

### CALL — subconsulta

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
RETURN persona.nombre, pelicula.titulo, numRoles, roles
ORDER BY numRoles DESC
LIMIT 10;
```
- `CALL { ... }` ejecuta una subconsulta por cada fila del contexto exterior
- `WITH persona` dentro del CALL pasa la variable del exterior a la subconsulta

---

### Traversal de dos saltos

```cypher
-- Actores que han trabajado con directores que dirigieron a Marlon Brando
MATCH (:Persona {nombre:"Marlon Brando"})-[:ACTUO_EN]->(:Pelicula)<-[:DIRIGE]-(director:Persona)
MATCH (director)-[:DIRIGE]->(:Pelicula)<-[:ACTUO_EN]-(actor:Persona)
RETURN DISTINCT actor.nombre AS nombre
ORDER BY nombre;
```
- Se pueden encadenar varios MATCH para navegar el grafo en múltiples saltos
- `DISTINCT` elimina duplicados en el resultado

---

## Resumen de cláusulas clave

| Cláusula | Qué hace |
|----------|---------|
| `MATCH` | Busca patrones en el grafo |
| `OPTIONAL MATCH` | Como MATCH pero devuelve null si no hay coincidencia (LEFT JOIN) |
| `WHERE` | Filtra resultados |
| `WHERE NOT (patrón)` | Excluye nodos que tienen ese patrón de relación |
| `RETURN` | Devuelve resultados |
| `WITH` | Encadena pasos, permite filtrar sobre agregaciones |
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

El examen 2025 puso un `MERGE ... ON CREATE SET ... RETURN`. Lo que hay que saber:

- Leer una query Cypher y explicar qué hace
- Distinguir `MERGE` vs `CREATE`
- Explicar qué es un nodo, una relación, una etiqueta, una propiedad
- El patrón `(a)-[:REL]->(b)` = relación dirigida de a hacia b

## Relación con otros conceptos

- [[neo4j]] — conceptos generales de grafos
- [[mongodb-cluster-comandos]] — comandos equivalentes en MongoDB
