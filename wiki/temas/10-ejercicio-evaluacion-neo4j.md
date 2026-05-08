---
title: Ejercicio de Evaluación Neo4J
type: tema
fuentes: [Neo4J.txt, Ejercicio de evaluación de Neo4J _ Campus Virtual.pdf, IntroNeo4J.pdf]
última_actualización: 2026-05-07
---

# Ejercicio de Evaluación Neo4J

## Arquitectura

El ejercicio importa datos desde **PostgreSQL** a **Neo4J** usando APOC y un conector JDBC.

```
PostgreSQL (xine DB)
       ↓  JDBC
    Neo4J (APOC)
       ↓
  Grafo con nodos Persona, Pelicula, Departamento y relaciones ACTUO_EN, TRABAJO_EN, DIRIGE
```

## Setup Docker

```bash
docker run -d \
  --name neo4j \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/password \
  -e NEO4J_PLUGINS='["apoc"]' \
  -e NEO4J_apoc_import_file_enabled=true \
  -e NEO4J_apoc_export_file_enabled=true \
  -v neo4j_data:/data \
  neo4j:latest
```

## Tarea 1: Importar Películas

```cypher
CALL apoc.load.jdbc(
  "jdbc:postgresql://postgres:5432/xine?user=xine&password=xine",
  "SELECT id, title, year FROM movies"
) YIELD row
CREATE (:Pelicula {id: row.id, titulo: row.title, anio: row.year})
```

## Tarea 2: Importar Personas y relación ACTUO_EN

```cypher
CALL apoc.load.jdbc(
  "jdbc:postgresql://postgres:5432/xine?user=xine&password=xine",
  "SELECT p.id, p.name, m.id as movie_id FROM cast c
   JOIN persons p ON p.id = c.person_id
   JOIN movies m ON m.id = c.movie_id"
) YIELD row
MERGE (persona:Persona {id: row.id, nombre: row.name})
WITH persona, row
MATCH (pelicula:Pelicula {id: row.movie_id})
CREATE (persona)-[:ACTUO_EN]->(pelicula)
```

## Tarea 3: Importar Departamentos y relación TRABAJO_EN

```cypher
CALL apoc.load.jdbc(
  "jdbc:postgresql://postgres:5432/xine?user=xine&password=xine",
  "SELECT p.id, p.name, d.name as dept, c.job FROM crew c
   JOIN persons p ON p.id = c.person_id
   JOIN departments d ON d.id = c.department_id
   JOIN movies m ON m.id = c.movie_id"
) YIELD row
MERGE (persona:Persona {id: row.id, nombre: row.name})
MERGE (dept:Departamento {nombre: row.dept})
MERGE (pelicula:Pelicula {id: row.movie_id})
CREATE (persona)-[:TRABAJO_EN {trabajo: row.job}]->(pelicula)
MERGE (persona)-[:PERTENECE]->(dept)
```

## Tarea 4: Crear relación DIRIGE desde TRABAJO_EN

Crear la relación `DIRIGE` para las personas cuyo trabajo en `TRABAJO_EN` es "Director":

```cypher
MATCH (p:Persona)-[t:TRABAJO_EN]->(pelicula:Pelicula)
WHERE t.trabajo = "Director"
CREATE (p)-[:DIRIGE]->(pelicula)
```

## Consulta 1: Películas en las que actúa una persona

```cypher
MATCH (p:Persona {nombre: "Penélope Cruz"})-[:ACTUO_EN]->(peli:Pelicula)
RETURN peli.titulo, peli.anio
ORDER BY peli.anio
```

## Consulta 2: Personas que actuaron en más de N películas

```cypher
MATCH (p:Persona)-[:ACTUO_EN]->(peli:Pelicula)
WITH p, count(peli) AS numPeliculas
WHERE numPeliculas > 10
RETURN p.nombre, numPeliculas
ORDER BY numPeliculas DESC
```

## Consulta 3: Directores de las películas de Penélope Cruz

```cypher
MATCH (penelope:Persona {nombre: "Penélope Cruz"})-[:ACTUO_EN]->(peli:Pelicula)
MATCH (director:Persona)-[:DIRIGE]->(peli)
RETURN DISTINCT director.nombre, peli.titulo
```

## Consulta 4: Actores que trabajaron con los mismos directores que Penélope Cruz

```cypher
MATCH (penelope:Persona {nombre: "Penélope Cruz"})-[:ACTUO_EN]->(p)<-[:DIRIGE]-(d:Persona)
MATCH (d)-[:DIRIGE]->(p2:Pelicula)<-[:ACTUO_EN]-(actor:Persona)
WHERE actor <> penelope
RETURN DISTINCT actor.nombre, d.nombre AS director
ORDER BY actor.nombre
```

## Consulta 5: Películas con OPTIONAL MATCH (incluir películas sin director conocido)

```cypher
MATCH (peli:Pelicula)
OPTIONAL MATCH (director:Persona)-[:DIRIGE]->(peli)
RETURN peli.titulo, peli.anio, director.nombre AS director
ORDER BY peli.anio DESC
LIMIT 20
```

## Consulta 6: Personas por departamento (agrupación con collect)

```cypher
MATCH (p:Persona)-[t:TRABAJO_EN]->(peli:Pelicula)
MATCH (p)-[:PERTENECE]->(dept:Departamento)
WITH dept.nombre AS departamento, peli.titulo AS pelicula,
     collect({nombre: p.nombre, trabajo: t.trabajo}) AS personal
RETURN departamento, pelicula, personal
ORDER BY departamento, pelicula
```

## Consulta 7: UNION — personas que actuaron O dirigieron

```cypher
MATCH (p:Persona)-[:ACTUO_EN]->(peli:Pelicula {titulo: "Volver"})
RETURN p.nombre AS nombre, "actor" AS rol
UNION
MATCH (p:Persona)-[:DIRIGE]->(peli:Pelicula {titulo: "Volver"})
RETURN p.nombre AS nombre, "director" AS rol
```

## Consulta 8: CALL (subconsulta) — películas por persona

```cypher
MATCH (p:Persona)
CALL {
  WITH p
  MATCH (p)-[:ACTUO_EN]->(peli:Pelicula)
  RETURN count(peli) AS numActuaciones
}
RETURN p.nombre, numActuaciones
ORDER BY numActuaciones DESC
LIMIT 10
```

## Consulta 9: Camino más corto entre dos actores (6 grados de separación)

```cypher
MATCH (a:Persona {nombre: "Tom Hanks"}), (b:Persona {nombre: "Penélope Cruz"})
MATCH path = shortestPath((a)-[*]-(b))
RETURN path, length(path) AS distancia
```

## Consulta 10: Subgrafo de colaboradores de un director

```cypher
MATCH (director:Persona {nombre: "Pedro Almodóvar"})-[:DIRIGE]->(peli:Pelicula)
MATCH (actor:Persona)-[:ACTUO_EN]->(peli)
RETURN director, peli, actor
```

## Patrones Cypher clave del ejercicio

| Patrón | Significado |
|--------|-------------|
| `(p:Persona)-[:ACTUO_EN]->(peli:Pelicula)` | Persona que actúa en película |
| `(p:Persona)-[t:TRABAJO_EN {trabajo:"Director"}]->(peli)` | Persona con rol Director |
| `(p)-[:DIRIGE]->(peli)` | Relación directa de dirección |
| `collect({nombre:..., trabajo:...})` | Agrupa en lista de objetos |
| `OPTIONAL MATCH` | LEFT JOIN en grafo |
| `UNION` | Une resultados sin duplicados |
| `CALL { WITH ... MATCH ... RETURN ... }` | Subconsulta con contexto externo |
| `shortestPath((a)-[*]-(b))` | Camino más corto sin restricción de tipo |

## Ver también
[[06-neo4j]], [[neo4j]], [[grafos]], [[mongodb-vs-neo4j]], [[09-ejercicio-evaluacion-mongodb]]
