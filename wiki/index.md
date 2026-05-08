---
title: Índice de la Wiki
type: index
---

# Índice de la Wiki — XINE

> Catálogo de todas las páginas. Se actualiza en cada ingest.

---

## Visión general

| Página | Descripción |
|--------|-------------|
| [[overview]] | Síntesis general de la asignatura, estructura del examen, hilo conductor |

---

## Temas

| Página | Fuentes | Descripción |
|--------|---------|-------------|
| [[00-criterios-evaluacion]] | criterios | Pesos, mínimos, normas — leer antes de cualquier cosa |
| [[00-aspectos-examen-RI]] | Aspectos fundamentales RI.pdf | Lista oficial del profesor — todos los temas RI para el examen |
| [[01-nosql-modelado]] | NoSQL1.Modelado.pdf | Modelos NoSQL, agregados, impedance mismatch, modelado por acceso |
| [[02-nosql-distribucion]] | NoSQL2.Distribucion.pdf | Sharding, replicación, Map-Reduce distribuido |
| [[03-nosql-consistencia]] | NoSQL3.Consistencia.pdf | Teorema CAP, BASE, quórums, Vector Stamp |
| [[04-bd-objeto-relacionales]] | BasesDatosObjetoRelacionales.pdf | SQL:1999, CREATE TYPE, herencia, referencias, ONLY() |
| [[05-mongodb]] | IntroMongoDB.pdf | Documentos BSON, Replica Set, Sharding, shard key |
| [[06-neo4j]] | IntroNeo4J.pdf | Grafos, Cypher, consultas del ejercicio |
| [[07-replicacion-sharding-mongodb]] | Replicación y Sharding.pdf | Ejemplos de configuración: RS puro, sharding, cluster completo |
| [[08-agregados-postgresql]] | Agregados en PostgreSQL.pdf | Arrays, tipos compuestos, JSON en PostgreSQL |
| [[09-ejercicio-evaluacion-mongodb]] | EntregaDia1.pdf, ComandosDia2.txt | Las 11 tareas del ejercicio MongoDB con comandos completos |
| [[10-ejercicio-evaluacion-neo4j]] | Neo4J.txt, Ejercicio Neo4J.pdf | Las 4 importaciones JDBC y 10 consultas Cypher |
| [[00-examenes-reales]] | examenes.pdf | **⭐ PRIORIDAD MÁXIMA** — Exámenes reales 2024 y 2025 con preguntas exactas y patrones |
| [[11-recuperacion-informacion]] | examenes.pdf | Bloque completo RI: índices, modelos, métricas, PageRank, clasificación, crawling |

---

## Conceptos

| Página | Fuentes | Descripción |
|--------|---------|-------------|
| [[nosql]] | NoSQL1.Modelado.pdf | Qué es NoSQL, los 4 modelos, casos de uso |
| [[teorema-cap]] | NoSQL3.Consistencia.pdf | CAP: Consistency, Availability, Partition tolerance |
| [[consistencia-eventual]] | NoSQL3.Consistencia.pdf | Definición, modelos, conflictos, Vector Stamp |
| [[sharding]] | NoSQL2.Distribucion.pdf | Particionado horizontal, hashed vs. ranged, shard key |
| [[replicacion]] | NoSQL2.Distribucion.pdf | Maestro-esclavo vs. peer-to-peer, quórums |
| [[acid-vs-base]] | NoSQL3.Consistencia.pdf | Propiedades ACID, propiedades BASE, espectro |
| [[agregados]] | NoSQL1.Modelado.pdf | Concepto de agregado (DDD), unidad de distribución |
| [[modelado-documentos]] | NoSQL1.Modelado.pdf | Embedding vs. referenciado, cuándo usar cada uno |
| [[escalado-horizontal]] | NoSQL2.Distribucion.pdf | Scale-out vs. scale-up, implicaciones distribuidas |
| [[mongodb]] | IntroMongoDB.pdf | Documento, colección, replica set, mongos |
| [[neo4j]] | IntroNeo4J.pdf | Nodo, etiqueta, relación, propiedad, Cypher |
| [[grafos]] | IntroNeo4J.pdf | BD de grafos, traversal, desventaja de escalado |
| [[bd-objeto-relacionales]] | BasesDatosObjetoRelacionales.pdf | SQL:1999, CREATE TYPE, herencia, referencias |
| [[indice-invertido]] | examenes.pdf | Estructura, AND queries, phrase queries, optimización — sale en todos los exámenes |
| [[tfidf]] | examenes.pdf | TF-IDF, modelo vectorial, similitud coseno, normalización por longitud |
| [[metricas-ri]] | examenes.pdf | Precision, Recall, P@k, MAP, MRR, NDCG — con fórmulas y ejemplos del examen |
| [[pagerank]] | examenes.pdf | PageRank (matriz P, teleportación) + HITS (hubs y autoridades) |
| [[bim]] | examenes.pdf | BIM: estimación de pᵢ y rᵢ; BM25: mejoras conceptuales |
| [[preprocesado-texto]] | examenes.pdf | Tokenización, normalización, stemming, stopword removal |
| [[clasificacion-textos]] | examenes.pdf | Naive Bayes, Rocchio, kNN, métricas de clasificación |

---

## Comparaciones y síntesis

| Página | Descripción |
|--------|-------------|
| [[nosql-vs-sql]] | Tabla comparativa completa; cuándo elegir cada uno; polyglot persistence |
| [[mongodb-vs-neo4j]] | Documentos vs. grafo; queries comparadas; escalado |
| [[acid-vs-base]] (comparacion) | ACID vs. BASE con quórums; relación con CAP |
