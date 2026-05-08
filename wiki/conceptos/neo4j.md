---
title: Neo4J
type: concepto
fuentes: [IntroNeo4J.pdf, Neo4J.txt]
última_actualización: 2026-05-07
---

# Neo4J

## Qué es
Base de datos de grafos [[nosql]] líder del mercado. Almacena datos como nodos y relaciones (aristas), con propiedades en ambos. Usa el lenguaje declarativo Cypher.

## Modelo de datos
- **Nodo**: entidad. Puede tener 0 o más **etiquetas** (Person, Movie...) y propiedades.
- **Relación**: conecta dos nodos. Tiene exactamente **un tipo** (ACTED_IN, DIRECTED...), dirección, y propiedades.
- **Propiedad**: par clave-valor en nodo o relación.

## Cypher — patrones esenciales
```cypher
-- MATCH básico
MATCH (p:Person {name: "Tom Hanks"})-[:ACTED_IN]->(m:Movie)
RETURN m.title

-- CREATE nodo y relación
CREATE (p:Person {name: "Ana"})-[:KNOWS]->(q:Person {name: "Luis"})

-- OPTIONAL MATCH (como LEFT JOIN)
OPTIONAL MATCH (actor:Person)-[:ACTED_IN]->(pelicula)

-- UNION
MATCH ... RETURN ...
UNION
MATCH ... RETURN ...

-- Subconsulta CALL
MATCH (p:Person)
CALL { WITH p ... RETURN ... }

-- MATCH con variable de relación
MATCH (p)-[r:ACTED_IN {order: 0}]->(m)
```

## Cuándo usar BD de grafos
- Las relaciones entre entidades son tan importantes como los datos en sí
- Consultas de traversal (¿quién conoce a quién? ¿qué camino une A con B?)
- Datos altamente conectados: redes sociales, recomendaciones, fraud detection

## Diferencia con MongoDB/relacional
- En BD relacionales y documentos, los JOINs/lookups son costosos con muchos saltos
- En grafos, navegar relaciones es la operación nativa y eficiente
- Grafos NO escalan bien horizontalmente (relaciones cruzadas entre todos los nodos)

## Restricciones e índices
```cypher
CREATE CONSTRAINT idPelicula FOR (p:Pelicula) REQUIRE p.id IS UNIQUE;
CREATE INDEX FOR (p:Pelicula) ON (p.titulo);
```

## Ver también
[[06-neo4j]], [[grafos]], [[mongodb-vs-neo4j]], [[nosql]]
