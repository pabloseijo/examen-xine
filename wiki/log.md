# Log de actividad

Registro cronológico de ingests, queries y lints. Append-only — no borrar entradas antiguas.

Formato de entrada: `## [YYYY-MM-DD] tipo | descripción`

---

## [2026-05-07] setup | Inicialización del vault

- Estructura de directorios creada
- CLAUDE.md configurado
- index.md y log.md inicializados
- Wiki lista para recibir PDFs

## [2026-05-07] ingest | Ingest completo de todos los ficheros fuente

Fuentes leídas:

- NoSQL1.Modelado.pdf — modelado NoSQL, 4 modelos, agregados, impedance mismatch
- NoSQL2.Distribucion.pdf — sharding, replicación, Map-Reduce
- NoSQL3.Consistencia.pdf — CAP, BASE, quórums, Vector Stamp
- BasesDatosObjetoRelacionales.pdf — SQL:1999, CREATE TYPE, herencia, referencias
- IntroMongoDB.pdf — documentos BSON, replica set, sharding
- IntroNeo4J.pdf — grafos, Cypher, patrones de consulta
- Agregados en PostgreSQL.pdf — arrays, tipos compuestos, JSON
- Aspectos fundamentales RI.pdf — lista oficial del profesor con temas de examen
- EntregaDia1..pdf / ComandosDia2.txt — solución ejercicio MongoDB cluster
- Neo4J.txt — setup Docker Neo4J + APOC, 10 consultas del ejercicio
- Ejercicio evaluación MongoDB.pdf — las 11 tareas del ejercicio
- Ejercicio evaluación Neo4J.pdf — las 4 importaciones y 10 consultas
- Replicación y Sharding en MongoDB.pdf — ejemplos de configuración

Páginas creadas en `temas/` (11):

- 00-aspectos-examen-RI.md
- 01-nosql-modelado.md
- 02-nosql-distribucion.md
- 03-nosql-consistencia.md
- 04-bd-objeto-relacionales.md
- 05-mongodb.md
- 06-neo4j.md
- 07-replicacion-sharding-mongodb.md
- 08-agregados-postgresql.md
- 09-ejercicio-evaluacion-mongodb.md
- 10-ejercicio-evaluacion-neo4j.md

Páginas creadas en `conceptos/` (13):

- nosql.md
- teorema-cap.md
- consistencia-eventual.md
- sharding.md
- replicacion.md
- acid-vs-base.md
- agregados.md
- modelado-documentos.md
- escalado-horizontal.md
- mongodb.md
- neo4j.md
- grafos.md
- bd-objeto-relacionales.md

Páginas creadas en `comparaciones/` (3):

- nosql-vs-sql.md
- mongodb-vs-neo4j.md
- acid-vs-base.md

Páginas actualizadas:

- overview.md — síntesis completa de la asignatura
- index.md — catálogo completo con todas las páginas

## [2026-05-14] ingest | ComandosDia2.txt — Comandos prácticas MongoDB

- Página creada: conceptos/mongodb-cluster-comandos.md
- Contenido: todos los comandos del cluster explicados parámetro a parámetro
- Cubre: mongod (--configsvr vs --shardsvrr), rs.initiate, mongos, sh.addShard, sh.shardCollection, createIndex, read preferences, aggregation pipeline ($match, $unwind, $group, $project, $sort, $limit)
- Hallazgo clave: --configsvr y --shardsvrr son flags distintos; createIndex va ANTES de shardCollection

## [2026-05-14] audit | Auditoría PDFs inbox — lagunas cubiertas

Páginas actualizadas con contenido de prioridad alta/media ausente en la wiki:

- `temas/03-nosql-consistencia.md` — añadido: sticky sessions como solución a read-your-writes
- `conceptos/consistencia-eventual.md` — añadido: sección "Garantizar read-your-writes" con sticky sessions, quórum fuerte y causal consistency
- `temas/01-nosql-modelado.md` — añadido: skinny row vs wide row, supercolumns (Cassandra); migración incremental proyectos nuevos (DBDeploy) vs legacy
- `temas/02-nosql-distribucion.md` — ampliado: Map-Reduce incremental con caso de reducer no combinable (redes de dependencias)
- `temas/04-bd-objeto-relacionales.md` — añadido: TREAT() vs IS OF, NEW para instanciar tipos, NOT INSTANTIABLE como tipo abstracto, FINAL

## [2026-05-07] ingest | examenes.pdf — DOCUMENTO CRÍTICO

- Fuentes: exámenes reales 2024 y 2025 + apuntes completos RI + Preguntas Viqueira NoSQL
- Páginas de temas creadas (2): 00-examenes-reales.md, 11-recuperacion-informacion.md
- Páginas de conceptos creadas (6): indice-invertido, tfidf, metricas-ri, pagerank, bim, preprocesado-texto, clasificacion-textos
- Hallazgo clave: las preguntas RI de 2024 y 2025 son casi idénticas (mismas posting lists incluso)
- Hallazgo clave: la parte NoSQL 2025 pide comandos de prácticas (MongoDB + Cypher) y Teorema CAP
