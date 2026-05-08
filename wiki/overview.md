---
title: Visión General — XINE
type: overview
fuentes: [NoSQL1.Modelado.pdf, NoSQL2.Distribucion.pdf, NoSQL3.Consistencia.pdf, IntroMongoDB.pdf, IntroNeo4J.pdf, BasesDatosObjetoRelacionales.pdf, Agregados en PostgreSQL.pdf, Aspectos fundamentales RI.pdf]
última_actualización: 2026-05-07
---

# Visión General — XINE (Xestión de Información non Estruturada)

## De qué trata la asignatura

XINE es una asignatura de 4º curso del Grado en Ingeniería Informática (USC) que cubre dos grandes bloques:

1. **Bases de datos no relacionales (NoSQL)**: modelos alternativos al relacional, distribución, consistencia, y extensiones objeto-relacionales de SQL
2. **Recuperación de Información (RI)**: indexado, modelos de ranking, clasificación de textos y análisis de enlaces

La asignatura parte de la premisa de que los datos del mundo real no siempre encajan bien en tablas relacionales: son semi-estructurados, masivos, distribuidos, o representan conexiones entre entidades. Los sistemas NoSQL, las extensiones objeto-relacionales y los motores de RI son las respuestas a estos casos.

---

## Temas principales

### Bloque 1 — NoSQL y Bases de Datos Alternativas

| Tema | Contenido |
|------|-----------|
| [[01-nosql-modelado]] | Modelos clave-valor, documentos, column-family, grafos; agregados como unidad de distribución; impedance mismatch; modelado por patrón de acceso |
| [[02-nosql-distribucion]] | Sharding y replicación (técnicas ortogonales); maestro-esclavo vs. peer-to-peer; Map-Reduce para indexado/análisis distribuido |
| [[03-nosql-consistencia]] | Teorema CAP; BASE vs. ACID; consistencia eventual; quórums (W>N/2, R+W>N); Vector Stamp para detección de conflictos |
| [[04-bd-objeto-relacionales]] | SQL:1999: CREATE TYPE, CREATE TABLE OF, herencia (UNDER), referencias (REF, ->), ONLY(), IS OF; PostgreSQL: arrays, JSON, tipos compuestos |
| [[05-mongodb]] | Documentos BSON; Replica Set; Sharding con mongos/config servers/shards; hashed vs. ranged sharding; criterios de shard key; zonas |
| [[06-neo4j]] | Modelo de grafos; Cypher: MATCH, WHERE, RETURN, CREATE, MERGE, WITH, UNION, OPTIONAL MATCH, CALL; traversals |
| [[07-replicacion-sharding-mongodb]] | Tres configuraciones de ejemplo: replica set puro, sharding sin réplicas, cluster completo 3×4 procesos |
| [[08-agregados-postgresql]] | Arrays multidimensionales, UNNEST, WITH ORDINALITY; tipos compuestos; JSON con ->/->>, json_array_elements, json_agg |
| [[09-ejercicio-evaluacion-mongodb]] | Las 11 tareas del ejercicio: cluster, replica sets, importación, sharding, consultas, prueba de disponibilidad |
| [[10-ejercicio-evaluacion-neo4j]] | Las 4 importaciones JDBC y 10 consultas Cypher del ejercicio |

### Bloque 2 — Recuperación de Información (RI)

| Tema | Contenido |
|------|-----------|
| [[00-aspectos-examen-RI]] | **Lista oficial del profesor** — todos los temas de RI relevantes para el examen con fórmulas y ejemplos |

---

## Conceptos fundamentales

### Conceptos NoSQL

- [[nosql]] — qué es, modelos, casos de uso
- [[teorema-cap]] — Consistency, Availability, Partition tolerance; CA/CP/AP
- [[consistencia-eventual]] — definición, modelos, conflictos, Vector Stamp
- [[sharding]] — particionado horizontal; hashed vs. ranged; shard key
- [[replicacion]] — maestro-esclavo vs. peer-to-peer; quórums
- [[acid-vs-base]] — propiedades ACID, propiedades BASE, espectro continuo
- [[agregados]] — unidad natural de modelado NoSQL
- [[modelado-documentos]] — embedding vs. referenciado
- [[escalado-horizontal]] — scale-out vs. scale-up

### Conceptos de tecnologías específicas

- [[mongodb]] — documento BSON, colección, replica set, shard
- [[neo4j]] — nodo, etiqueta, relación, propiedad, Cypher
- [[grafos]] — cuándo usar BD de grafos, ventajas en traversal, desventaja en escalado
- [[bd-objeto-relacionales]] — SQL:1999, CREATE TYPE, herencia, referencias

### Comparaciones

- [[nosql-vs-sql]] — tabla comparativa completa, cuándo elegir cada uno, polyglot persistence
- [[mongodb-vs-neo4j]] — documentos vs. grafo, queries comparadas, escalado
- [[acid-vs-base]] (comparacion) — ACID vs. BASE con quórums y relación con CAP

---

## Estructura del examen

Según la lista oficial del profesor (`Aspectos fundamentales de cara al examen de teoría de la parte de RI`):

### Parte RI (examen de teoría)

Temas con mayor peso:

1. Índices invertidos (con y sin posición)
2. Preprocesado: tokenización, normalización, stemming, stopwords
3. Queries booleanas y phrase queries
4. Ranked retrieval: tf-idf, modelo vectorial, similitud coseno
5. Normalización por longitud del documento
6. BIM (Binary Independence Model) — estimación de p_i y r_i
7. BM25
8. Métricas: Precision, Recall, P@k, MAP, MRR, NDCG
9. Clasificación: Naive Bayes, kNN, Rocchio
10. Link analysis: PageRank, HITS
11. Crawling

### Parte NoSQL/BD (ejercicio práctico / preguntas)

- Configuración de clusters MongoDB (replica set + sharding)
- Consultas Cypher en Neo4J
- Queries PostgreSQL con arrays, tipos compuestos, JSON
- Modelado NoSQL vs. relacional

---

## Hilo conductor de la asignatura

El modelo relacional es excelente pero tiene limitaciones:

- **Impedance mismatch**: los objetos no se mapean bien a tablas
- **Escalado**: difícil distribuir datos relacionales en clusters
- **Esquema rígido**: los datos del mundo real son variables

Las soluciones que estudia XINE:

1. **NoSQL**: nuevos modelos (documentos, grafos, clave-valor, column-family) diseñados para distribución y esquemas flexibles
2. **BD Objeto-Relacionales**: extender SQL con tipos complejos, herencia y referencias (compromiso)
3. **RI**: cuando los datos son texto no estructurado, se necesitan técnicas de indexado y ranking, no SQL

Todo confluye en el concepto de **polyglot persistence**: usar la herramienta correcta para cada tipo de dato y consulta.
