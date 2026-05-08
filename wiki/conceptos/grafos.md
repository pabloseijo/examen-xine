---
title: Bases de Datos de Grafos
type: concepto
fuentes: [IntroNeo4J.pdf]
última_actualización: 2026-05-07
---

# Bases de Datos de Grafos

## Modelo de datos
- **Nodo**: entidad con etiquetas y propiedades
- **Relación (arista)**: conecta dos nodos, tiene tipo, dirección y propiedades
- **Etiqueta**: categoría de un nodo (un nodo puede tener varias)
- **Propiedad**: par clave-valor en nodo o relación
- **Recorrido (Traversal)**: consulta que sigue relaciones de nodo en nodo

## Cuándo usar grafos
- Las relaciones entre entidades son tan importantes como los datos
- Consultas de conectividad: ¿quién conoce a quién? ¿cuál es el camino más corto?
- Datos altamente conectados sin profundidad fija
- Casos de uso: redes sociales, motores de recomendación, detección de fraude, knowledge graphs

## Ventaja sobre modelo relacional
Con JOINs múltiples en SQL, la consulta se vuelve exponencialmente más cara con cada salto. En una BD de grafos, navegar relaciones es la operación nativa y eficiente independientemente de la profundidad.

## Desventaja: escalado horizontal
Los grafos son difíciles de partir (los nodos tienen relaciones con cualquier otro nodo). Por eso la mayoría de BD de grafos operan en un único servidor (o servidor maestro). No existe un sharding natural de grafos.

## BD de grafos más populares
- **[[neo4j]]**: líder del mercado, usa Cypher, ACID completo
- Amazon Neptune: servicio gestionado en AWS
- JanusGraph: distribuido (sobre HBase/Cassandra)
- ArangoDB: multi-modelo (documentos + grafos)

## Lenguaje Cypher (Neo4J)
- Lenguaje declarativo de patrones
- `MATCH (n:Label {prop: valor})-[:TIPO]->(m) RETURN m`
- `CREATE`, `MERGE`, `SET`, `DELETE`, `WITH`, `UNION`, `OPTIONAL MATCH`, `CALL`

## Ver también
[[neo4j]], [[06-neo4j]], [[nosql]], [[mongodb-vs-neo4j]]
