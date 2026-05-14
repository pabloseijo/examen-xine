---
title: Índice de la Wiki — XINE
type: index
última_actualización: 2026-05-14
---

# Wiki de Estudio — XINE (Xestión de Información non Estruturada)

> Wiki mantenida por LLM para preparar el examen del 18 de mayo de 2026.
> **Para empezar**: lee [[00-criterios-evaluacion]] y [[00-examenes-reales]]. Todo lo demás sale de ahí.

---

## Mapa del repositorio

```
XINE/
├── CLAUDE.md                  ← instrucciones para el LLM mantenedor
├── raw/                       ← PDFs originales (solo lectura)
│   └── assets/                ← imágenes generadas
└── wiki/
    ├── index.md               ← este archivo
    ├── log.md                 ← historial de cambios
    ├── overview.md            ← síntesis general de la asignatura
    ├── temas/                 ← resúmenes por tema y sesiones de estudio
    ├── conceptos/             ← una página por concepto clave
    ├── comparaciones/         ← tablas comparativas y análisis cruzados
    └── Apuntes/               ← notas de sesiones de estudio diarias
```

---

## Estructura del examen

| Bloque | Peso teórica | Mínimo | Notas |
|--------|-------------|--------|-------|
| **RI** | 7 / 10 pts | Aprobar | Sale casi igual cada año |
| **BD** | 3 / 10 pts | Aprobar | Varía más, rota entre temas |

Hay que aprobar RI y BD **por separado**. Un 10 en RI no salva un suspenso en BD.

---

## BLOQUE RI — Recuperación de Información

### Entrada recomendada
1. [[00-examenes-reales]] — preguntas exactas de 2024 y 2025
2. [[11-recuperacion-informacion]] — visión completa del bloque

### Grafo de conceptos RI

```
indice-invertido ──────────────────────────────────────────┐
    ├── preprocesado-texto (tokenización, stemming, stops)  │
    └── phrase queries (índice posicional)                  │
                                                            ▼
tfidf ──────────────────────────────────────────────────► modelo-vectorial
    ├── normalización por longitud                              │
    └── similitud coseno                                        │
                                                                ▼
bim ◄────────────────────────────────────────────────── metricas-ri
    └── bm25 (mejoras conceptuales)                   ├── P@k, MAP, MRR
                                                       ├── NDCG
                                                       └── evaluación por clicks

pagerank ──────────────────────────────────────────────────┐
    ├── HITS (hubs y autoridades)                           │
    └── anchor text                                         │
                                                            ▼
clasificacion-textos ─────────────────────────────────► crawling
    ├── Naive Bayes
    ├── Rocchio
    └── kNN
```

### Páginas RI por prioridad de examen

| Prioridad | Página | Qué es |
|-----------|--------|--------|
| ⭐⭐⭐ | [[indice-invertido]] | AND/OR/NOT, phrase queries, optimización — sale siempre |
| ⭐⭐⭐ | [[metricas-ri]] | P@k, MAP, MRR, NDCG — sale siempre con cálculos |
| ⭐⭐⭐ | [[pagerank]] | Matriz P con teleportación — sale siempre |
| ⭐⭐⭐ | [[tfidf]] | TF-IDF, modelo vectorial, similitud coseno |
| ⭐⭐ | [[modelo-vectorial]] | Normalización por longitud — cayó en 2025 |
| ⭐⭐ | [[bim]] | BIM (pᵢ, rᵢ) + BM25 conceptual |
| ⭐⭐ | [[preprocesado-texto]] | Stopwords, stemming, efecto en índice |
| ⭐ | [[clasificacion-textos]] | NB, Rocchio, kNN, F1 — tipo test |
| ⭐ | [[11-recuperacion-informacion]] | Visión completa: crawling, HITS, MapReduce |

---

## BLOQUE BD — Bases de Datos NoSQL

### Entrada recomendada
1. [[00-examenes-reales]] — sección BD de 2024 y 2025
2. [[nosql]] — visión general antes de entrar en detalle

### Grafo de conceptos BD

```
nosql ──────────────────────────────────────────────────────┐
    ├── agregados (unidad de distribución)                   │
    ├── modelado-documentos (embedding vs referenciado)      │
    └── escalado-horizontal (scale-out)                      │
                                                             ▼
teorema-cap ◄──────────────────────────────────────── acid-vs-base
    ├── sistema centralizado (CA)                       ├── ACID (relacional)
    └── sistema distribuido (CP vs AP)                  └── BASE (NoSQL)
             │
             ▼
replicacion ──────────────────────────────────────────────┐
    ├── maestro-esclavo                                    │
    ├── peer-to-peer                                       │
    └── consistencia-eventual ── Vector Stamp             │
             │                                             │
             ▼                                             ▼
sharding ──────────────────────────────────────────── quórums (R+W>N)
    ├── hashed vs ranged
    └── shard key

bd-objeto-relacionales ─────────────────────────────────────┐
    ├── CREATE TYPE, UNDER, ONLY()                           │
    └── agregados-postgresql ── arrays, UNNEST, JSON         │
                                                             ▼
mongodb ◄────────────────────────────────────────── mongodb-cluster-comandos ⭐
    ├── mongod (--configsvr / --shardsvrr)
    ├── rs.initiate, mongos, sh.addShard
    └── aggregation pipeline ($match, $unwind, $group...)

neo4j ◄──────────────────────────────────────────── neo4j-cypher-practica ⭐
    ├── MATCH, MERGE, WITH, collect
    ├── UNION, OPTIONAL MATCH
    └── importación JDBC con APOC
```

### Páginas BD por prioridad de examen

| Prioridad | Página | Qué es |
|-----------|--------|--------|
| ⭐⭐⭐ | [[mongodb-cluster-comandos]] | Todos los comandos del cluster explicados — el profesor dijo que entran los de prácticas |
| ⭐⭐⭐ | [[neo4j-cypher-practica]] | Importaciones JDBC + 10 patrones Cypher de la práctica |
| ⭐⭐⭐ | [[teorema-cap]] | CAP centralizado vs distribuido, compromiso latencia/consistencia — cayó en 2025 |
| ⭐⭐⭐ | [[nosql]] | 4 modelos, casos de uso, esquema relacional vs NoSQL — cayó en 2025 |
| ⭐⭐ | [[replicacion]] | Maestro-esclavo vs peer-to-peer, write-write, Vector Stamp — cayó en 2024 |
| ⭐⭐ | [[sharding]] | Hashed vs ranged, shard key — contexto para MongoDB |
| ⭐⭐ | [[acid-vs-base]] | ACID vs BASE, quórums — cayó en 2024 |
| ⭐ | [[bd-objeto-relacionales]] | CREATE TYPE, herencia, ONLY() |
| ⭐ | [[consistencia-eventual]] | Vector Stamp, consistencia eventual |
| ⭐ | [[agregados]] | Concepto de agregado, unidad de distribución |
| ⭐ | [[modelado-documentos]] | Embedding vs referenciado |

---

## Sesiones de estudio (Apuntes/)

Notas diarias de las sesiones. Lectura rápida para repasar lo que se estudió cada día.

| Día | Archivo | Contenido |
|-----|---------|-----------|
| Día 1 | [[Día 1 -- RI vs RE, índice invertido, queries booleanas]] | Índice invertido, AND/OR/NOT, optimización |
| Día 2 | [[Día 2 -- Índice posicional + Phrase queries + Preprocesado + MapReduce]] | Phrase queries, preprocesado, MapReduce |
| Día 3 | [[Día 3 -- TF-IDF, modelo vectorial, similitud coseno, normalización por longitud y métricas de evaluación]] | TF-IDF, coseno, normalización |
| Día 4 | [[Día 4 -- Modelo Vectorial, Similitud del Coseno ,Normalización por longitud, BIM y BM25]] | BIM, BM25, modelo vectorial |
| Día 5 | [[Día 5 -- NoSQL, CAP, ACID vs BASE, Quórums, BD Objeto-Relacionales, PostgreSQL]] | BD completo día 1 |

---

## Temas (resúmenes por fuente PDF)

Útiles para búsqueda por origen del material.

| Página | Fuente | Contenido |
|--------|--------|-----------|
| [[00-criterios-evaluacion]] | criterios-evaluacion.txt | Pesos, mínimos, normas — **leer primero** |
| [[00-examenes-reales]] | examenes.pdf | **⭐ PRIORIDAD MÁXIMA** — Exámenes reales 2024 y 2025 |
| [[00-aspectos-examen-RI]] | Aspectos fundamentales RI.pdf | Lista oficial del profesor con todos los temas RI |
| [[00-plan-estudio-10-dias]] | — | Plan de estudio revisado para el examen del 18 mayo |
| [[11-recuperacion-informacion]] | examenes.pdf | Bloque RI completo: índices, modelos, métricas, PageRank, clasificación, crawling |
| [[01-nosql-modelado]] | NoSQL1.Modelado.pdf | Modelos NoSQL, agregados, impedance mismatch |
| [[02-nosql-distribucion]] | NoSQL2.Distribucion.pdf | Sharding, replicación, Map-Reduce distribuido |
| [[03-nosql-consistencia]] | NoSQL3.Consistencia.pdf | CAP, BASE, quórums, Vector Stamp |
| [[04-bd-objeto-relacionales]] | BasesDatosObjetoRelacionales.pdf | SQL:1999, CREATE TYPE, herencia |
| [[05-mongodb]] | IntroMongoDB.pdf | Documentos BSON, Replica Set, Sharding |
| [[06-neo4j]] | IntroNeo4J.pdf | Grafos, Cypher básico |
| [[07-replicacion-sharding-mongodb]] | Replicación y Sharding.pdf | Configuración RS, sharding, cluster completo |
| [[08-agregados-postgresql]] | Agregados en PostgreSQL.pdf | Arrays, tipos compuestos, JSON |
| [[09-ejercicio-evaluacion-mongodb]] | EntregaDia1.pdf, ComandosDia2.txt | 11 tareas del ejercicio MongoDB |
| [[10-ejercicio-evaluacion-neo4j]] | Neo4J.txt, Ejercicio Neo4J.pdf | 4 importaciones JDBC + 10 consultas Cypher |

---

## Comparaciones

| Página | Qué compara |
|--------|-------------|
| [[nosql-vs-sql]] | Cuándo usar NoSQL vs relacional; polyglot persistence |
| [[mongodb-vs-neo4j]] | Documentos vs grafo; queries comparadas; escalado |
| [[acid-vs-base]] | ACID vs BASE; quórums; relación con CAP |
