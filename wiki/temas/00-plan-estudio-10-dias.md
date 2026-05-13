---
title: Plan de Estudio — REVISADO 11 mayo
type: tema
fuentes: [examenes.pdf, criterios-evaluacion.txt]
última_actualización: 2026-05-11
---

# Plan de Estudio — REVISADO (examen el 18 de mayo)

> Quedan **7 días**. Días 1-3 completados. Nuevo plan: 2 días RI restante + 3 días BD + 2 días repaso.

## Estado actual (11 mayo)

| Bloque | Estado |
|--------|--------|
| RI — Índice invertido, queries booleanas, optimización | ✅ Hecho |
| RI — Índice posicional, phrase queries, preprocesado, MapReduce | ✅ Hecho |
| RI — TF-IDF, modelo vectorial, normalización, BIM, BM25 | ✅ Hecho |
| RI — Métricas, PageRank, clasificación, crawling | ⏳ Pendiente |
| BD — Todo | ⏳ Pendiente |

## Plan revisado

> ⚠️ El profesor confirmó explícitamente que entra un ejercicio sobre el código de las prácticas (MongoDB + Neo4J). Hay que saber qué hace cada comando/bloque de código, no memorizarlo.

| Fecha | Qué |
|-------|-----|
| ~~11 mayo~~ | RI — Métricas ✅ |
| ~~12 mayo~~ | RI — PageRank, HITS, clasificación, crawling ✅ |
| **13 mayo (hoy)** | BD — 4 modelos NoSQL, CAP, BASE, quórums, write-write, BD objeto-relacional, PostgreSQL |
| **14 mayo** | BD — Sharding, replicación, MongoDB (comandos prácticas), Neo4J/Cypher (comandos prácticas) |
| **15 mayo** | BD — **Repaso código prácticas**: leer [[09-ejercicio-evaluacion-mongodb]] y [[10-ejercicio-evaluacion-neo4j]] línea a línea y ser capaz de explicar qué hace cada bloque |
| **16 mayo** | Simulacro parte RI + repaso de errores |
| **17 mayo** | Simulacro examen completo + repaso final |
| **18 mayo** | 🎯 EXAMEN |

## Principios

- Hay que **aprobar RI y BD de forma independiente** — un 10 en RI no salva un suspenso en BD
- BD varía más entre exámenes → 3 días para cubrirlo con garantías
- Los 2 días de repaso son innegociables — sin simulacros no se detectan los huecos reales

---

## BLOQUE 1 — RI (Días 1–4)

### Día 1 — Qué es RI + Modelos booleanos + Índice invertido

**Temas**: [[11-recuperacion-informacion]] (sección RI vs RE), [[indice-invertido]]

- RI vs Recuperación Estructurada: qué es cada una, diferencias (datos, consulta, resultado, relevancia)
- Modelo booleano: matriz de incidencia, por qué es ineficiente
- Índice invertido sin posición: estructura (diccionario + posting lists), cómo construirlo
- AND query: merge algorithm paso a paso, complejidad O(x+y), optimización por df
- OR y NOT: cómo funcionan (unión, complemento)

**Práctica**:
1. Dado un corpus de 5 documentos → construir el índice invertido
2. Resolver queries AND, OR, NOT con ese índice
3. Explicar por escrito la optimización del AND con 3 términos

**Check al final del día**: construir un índice y resolver cualquier query booleana sin consultar nada.

---

### Día 2 — Índice posicional + Phrase queries + Preprocesado + MapReduce

**Temas**: [[indice-invertido]] (sección posicional), [[preprocesado-texto]]

**Mañana — Índice posicional y phrase queries**:
- Diferencia índice con y sin posición
- Phrase query: para "A B" buscar posición p en A tal que p+1 ∈ B, para cada doc en común
- Practicar con posting lists reales de los exámenes:
  - `to`: doc1:[1,19,22]; doc2:[1,39,42]
  - `be`: doc1:[2,21,25]; doc2:[1,40,41]
  - Resolver "to be" y "be to" paso a paso

**Tarde — Preprocesado y MapReduce**:
- Tokenización, normalización, stemming, stopword removal: qué es cada una y por qué
- Efecto de eliminar una stopword: su posting list es la más grande → afecta más al tamaño de las postings
- MapReduce para indexado distribuido: fase Map (parsers: emiten pares término→docID), Shuffle (agrupar por término), Reduce (inversores: construyen posting list)
- Particionado por término vs. particionado por documento: diferencias

**Check**: resolver cualquier phrase query + explicar el pipeline de indexado distribuido.

---

### Día 3 — TF-IDF + Modelo Vectorial + Normalización + BIM + BM25

**Temas**: [[tfidf]], [[bim]]

**Mañana — TF-IDF y modelo vectorial**:
- TF ponderado: w(t,d) = 1 + log₁₀(tf) si tf>0, 0 si no
- IDF: idf(t) = log₁₀(N / df(t)) — mayor cuando el término es raro
- TF-IDF: w(t,d) × idf(t)
- Score(q,d) = Σ tf-idf(t,d) para t en q∩d
- Modelo vectorial: documentos y queries como vectores de tf-idf
- Similitud coseno: sim(d,q) = (d·q) / (||d|| × ||q||)
- Normalización por longitud: por qué (documentos largos no deben tener ventaja), cómo (dividir por norma L2)

**Tarde — BIM y BM25**:
- BIM: documentos como vectores binarios, independencia entre términos
- Variables: pᵢ (prob. término en doc relevante) = s/S; rᵢ (prob. en irrelevante) = (df-s)/(N-S)
- Tabla de contingencia: saber construirla y calcular pᵢ y rᵢ con números concretos
- BM25: mejoras sobre BIM — TF normalizada (saturación), IDF mejorado, normalización por longitud con parámetros k1 y b. Solo conceptual, sin fórmula completa.

**Check**: calcular tf-idf de términos en documentos; calcular pᵢ y rᵢ dada una tabla.

---

### Día 4 — Métricas de evaluación: todas

**Temas**: [[metricas-ri]]

Este día es puro ejercicio. Las métricas son mecánicas pero hay que practicarlas con números.

**Fórmulas**:
- Precision = rel. recuperados / total recuperados
- Recall = rel. recuperados / total relevantes en colección
- P@k = rel. en top k / k
- MAP = media de AP por query; AP = media de P@k en cada posición con relevante
- MRR = (1/Q) × Σ (1/posición del primer relevante)
- DCG@p = Σ (2^relev_i - 1) / log₂(i+1); NDCG = DCG / IDCG

**Ejercicios** (inventar 2-3 escenarios propios además de los del examen):
1. Dado ranking con R/NR y total relevantes en colección → calcular todo
2. Dado ranking con grados de relevancia (0, 1, 2, 3) → calcular NDCG@3 y @5
3. Dado 2 queries → calcular MRR del sistema

**Temas adicionales del día**:
- Evaluación por clicks: clicks = preferencia, no relevancia; sesgo posicional; tiempo entre clicks; orden de navegación
- Interleaved rankings: cómo se construye, cómo se puntúa
- A/B testing: qué es, cuándo se usa vs interleaving

**Check**: calcular las 6 métricas correctamente con cualquier input.

---

### Día 5 — PageRank + HITS + Clasificación + Crawling 

**Temas**: [[pagerank]], [[clasificacion-textos]], [[bim]]

> Día comprimido: BIM y BM25 se mueven aquí desde el día 3 para liberar espacio. El día 3 queda entero para TF-IDF y métricas.

**Mañana — PageRank y HITS**:
- Random walk: prob. d de seguir enlace, prob. (1-d) de teleportar
- Matriz P: P[i][j] = (1-d)/N + d×(1/outlinks(j)) si j→i; (1-d)/N si no
- Practicar calculando filas de P con grafos de 3-4 nodos y teleportación del 5% (d=0.95)
- HITS: hub (enlaza a autoridades) vs autoridad (responde la consulta directamente). Diferencia con PageRank: HITS es query-dependent y se calcula online; PageRank es global y offline
- Anchor text: el texto del enlace describe a la página destino → señal de calidad externa

**Tarde — Clasificación y Crawling**:
- Clasificación documental: qué es, por qué es aprendizaje supervisado
- Naive Bayes: probabilístico, asume independencia, rápido, robusto
- Rocchio: centroide de cada clase, asigna al más cercano
- kNN: k vecinos más cercanos, clase mayoritaria, sin entrenamiento
- Métricas de clasificación: accuracy, F1 (media armónica de P y R)
- Crawling: qué es, spider traps, robots.txt, características necesarias (robusto, educado, distribuido, escalable, eficiente, continuo)

**Check**: calcular una fila de la matriz P; distinguir hub de autoridad; describir los 3 clasificadores.

---

## BLOQUE 2 — BD (Días 6–7)

### Día 6 — Modelos NoSQL + Consistencia + BD Objeto-Relacionales

**Temas**: [[nosql]], [[teorema-cap]], [[consistencia-eventual]], [[acid-vs-base]], [[bd-objeto-relacionales]], [[agregados]]

**Mañana — Modelos NoSQL y consistencia**:
- Los 4 modelos NoSQL: clave-valor, documentos, column-family, grafos — características y casos de uso de cada uno
- Agregados: qué es un agregado (unidad natural de datos), por qué facilita la distribución
- Impedance mismatch: por qué los objetos no mapean bien a tablas relacionales
- Modelado por patrón de acceso: en NoSQL se diseña la BD según cómo se va a consultar, no según normalización
- Embedding vs referenciado: cuándo incrustar documentos y cuándo referenciar
- Esquema: relacional (rígido, scripts de migración, New y Legacy) vs NoSQL (schema-on-read, la app controla los cambios)
- Teorema CAP: C, A, P — sistema centralizado (CA) vs distribuido (necesita P, compromiso C vs A)
- En la práctica: no es "elegir 2 de 3" sino compromiso entre consistencia y latencia
- Propiedades ACID vs BASE: qué ofrece cada modelo, relación con CAP
- Quórums: W>N/2 garantiza escritura; R+W>N garantiza que lees lo último escrito
- Vector Stamp: para detectar conflictos en consistencia eventual (write-write)

**Tarde — BD Objeto-Relacionales y Agregados en PostgreSQL**:
- SQL:1999: CREATE TYPE, CREATE TABLE OF, herencia (UNDER), referencias (REF, →), ONLY()
- PostgreSQL: arrays, tipos compuestos, JSON con ->/->>, json_array_elements, json_agg, UNNEST, WITH ORDINALITY
- Cuándo usar BD objeto-relacional vs NoSQL puro

**Check**: explicar los 4 modelos NoSQL; explicar CAP con ejemplo centralizado vs distribuido; escribir un CREATE TYPE con herencia.

---

### Día 7 — Distribución + MongoDB + Neo4J

**Temas**: [[sharding]], [[replicacion]], [[mongodb]], [[neo4j]]

**Mañana — Distribución: sharding y replicación**:
- Sharding (particionado horizontal): cada nodo tiene un subconjunto de datos
  - Hashed sharding vs ranged sharding: diferencias y casos de uso
  - Shard key: qué es, criterios para elegirla (cardinalidad, distribución uniforme, no hotspots)
- Replicación maestro-esclavo: ventajas (disponibilidad en lectura), desventajas (cuello de botella, punto único de fallo)
- Replicación peer-to-peer: ventajas (sin cuello de botella), desventajas (conflictos write-write)
- Problema write-write: estrategia pesimista (bloquear antes) vs optimista (detectar después con Vector Stamp)
- MapReduce distribuido para análisis (no indexado): las mismas fases aplicadas a datos

**Tarde — MongoDB y Neo4J en detalle**:

MongoDB:
- Arquitectura de cluster: mongod (nodos de datos), mongos (router), config servers
- Replica Set: inicializar con rs.initiate, añadir miembros, read preference (primary, primaryPreferred, secondary, nearest)
- Comandos de prácticas — saber qué hace cada uno:
```javascript
mongod --replSet nombre --port X --dbpath ruta --bind_ip localhost
rs.initiate({ _id: "nombre", configsvr: true/false, members: [...] })
mongos --configdb rsConf/host:port --port X
sh.addShard("rsNombre/host:port")
sh.enableSharding("baseDatos")
sh.shardCollection("bd.coleccion", { campo: "hashed" })
db.getMongo().setReadPref('primaryPreferred')
```

Neo4J y Cypher:
```cypher
-- Crear/buscar nodo (MERGE = crear si no existe)
MERGE (n:Label { propiedad: valor })
ON CREATE SET n.campo = valor
RETURN n

-- Consultar relaciones
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WHERE m.title = "Cloud Atlas"
RETURN a.name, m.title

-- Crear relación
MATCH (p:Person {name:"X"}), (m:Movie {title:"Y"})
CREATE (p)-[:DIRECTED]->(m)

-- Aggregation
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
RETURN p.name, COUNT(m) AS peliculas
ORDER BY peliculas DESC

-- OPTIONAL MATCH (como LEFT JOIN)
MATCH (p:Person)
OPTIONAL MATCH (p)-[:DIRECTED]->(m:Movie)
RETURN p.name, m.title
```

**Check**: explicar diferencia hashed vs ranged sharding; describir arquitectura de cluster MongoDB; escribir 3 queries Cypher sin ayuda.

---

## BLOQUE 3 — Repaso y simulacros (Días 8–10)

### Día 8 — Repaso RI + Simulacro RI

**Mañana**: repasar los conceptos de RI que menos seguros estén (revisar las páginas wiki de los temas flojos).

**Tarde — Simulacro parte RI** (objetivo: 60 min sin consultar nada):
Reproducir la parte RI del examen 2025 completa:
1. AND query + optimización
2. Phrase query "to be" / "be to"
3. PageRank (fila de la matriz P)
4. MRR + P@5 + MAP + NDCG@3
5. Efecto de eliminar stopword
6. Normalización por longitud
7. 5 preguntas tipo test inventadas

Corrección: consultar wiki y anotar qué falló.

---

### Día 9 — Repaso BD + Simulacro BD

**Mañana**: repasar los conceptos de BD que menos seguros estén.

**Tarde — Simulacro parte BD** (objetivo: 30 min sin consultar nada):
Reproducir la parte BD del examen 2025:
1. Esquema: ventajas/desventajas en relacional vs NoSQL
2. Teorema CAP: sistema centralizado, sistemas distribuidos, compromiso real
3. Explicar qué hace cada uno de los 4 comandos del examen (mongod, rs.initiate, setReadPref, MERGE Cypher)

Luego simular la parte BD del examen 2024:
1. Acceso a datos al modelar en NoSQL
2. Replicación maestro-esclavo vs peer-to-peer
3. Problema write-write y estrategias

Corrección: consultar wiki y anotar qué falló.

---

### Día 10 — Simulacro completo + repaso final de huecos

**Mañana — Simulacro del examen completo** (~90 minutos, condiciones reales):
- Sin consultar nada
- Reproducir el examen 2025 de principio a fin
- Anotar tiempo por sección

**Tarde — Solo repasar lo que falló**:
- No volver a leer todo
- Ir directamente a las páginas wiki de los conceptos donde hubo errores
- Hacer 2-3 ejercicios adicionales sobre esos puntos concretos

---

## Resumen del plan

| Día | Bloque | Temas |
|-----|--------|-------|
| 1 | RI | RI vs RE, modelos booleanos, índice invertido, AND/OR/NOT |
| 2 | RI | Índice posicional, phrase queries, preprocesado, MapReduce |
| 3 | RI | TF-IDF, modelo vectorial, normalización, BIM, BM25 |
| 4 | RI | Métricas: P@k, MAP, MRR, NDCG, evaluación por clicks, A/B |
| 5 | RI | PageRank, HITS, clasificación (NB/Rocchio/kNN), crawling |
| 6 | BD | 4 modelos NoSQL, CAP, BASE, quórums, BD objeto-relacional, PostgreSQL |
| 7 | BD | Sharding, replicación, write-write, MongoDB, Neo4J/Cypher |
| 8 | Repaso | Repaso RI + simulacro parte RI |
| 9 | Repaso | Repaso BD + simulacro parte BD |
| 10 | Repaso | Simulacro examen completo + repaso de huecos |
