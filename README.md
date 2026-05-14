<div align="center">

# Wiki de Estudio — XINE

*Xestión de Información non Estruturada · USC · Mayo 2026*

[![Examen](https://img.shields.io/badge/Examen-18%20mayo%202026-red?style=flat-square)](wiki/temas/00-examenes-reales.md)
[![RI](https://img.shields.io/badge/Bloque%20RI-7%20pts-0366d6?style=flat-square)](wiki/conceptos/indice-invertido.md)
[![BD](https://img.shields.io/badge/Bloque%20BD-3%20pts-2ea44f?style=flat-square)](wiki/conceptos/nosql.md)
[![Claude](https://img.shields.io/badge/Mantenida%20por-Claude%20Code-8250df?style=flat-square)](CLAUDE.md)

Base de conocimiento mantenida por LLM, organizada por grafos de conceptos,  
con prioridad de examen y lista para compartir con toda la clase.

</div>

---

## 📋 Estructura del Examen

| | Teórica (50%) | Práctica (50%) | Mínimo |
|---|---|---|---|
| **Recuperación de Información** | 7 pts | 3 pts | Aprobar por separado |
| **Bases de Datos NoSQL** | 3 pts | 7 pts | Aprobar por separado |

> ⚠️ Un 10 en RI **no salva** un suspenso en BD. Hay que aprobar cada bloque de forma independiente, tanto en teórica como en práctica.

---

## 🗂️ Contenidos por Bloque

<table>
<tr>
<td width="50%" valign="top">

### 🔍 Recuperación de Información

- Índice invertido — AND, OR, NOT, phrase queries
- TF-IDF, modelo vectorial, similitud coseno
- Métricas — P@k, MAP, MRR, NDCG
- PageRank (matriz P, teleportación) + HITS
- BIM y BM25 — modelos probabilísticos
- Clasificación — Naive Bayes, Rocchio, kNN
- Preprocesado, stopwords, stemming, MapReduce

</td>
<td width="50%" valign="top">

### 🗄️ Bases de Datos NoSQL

- 4 modelos NoSQL — clave-valor, documentos, columnas, grafos
- Teorema CAP — centralizado vs distribuido
- ACID vs BASE, quórums, consistencia eventual
- Replicación maestro-esclavo vs peer-to-peer, Vector Stamp
- Sharding — hashed vs ranged, shard key
- **MongoDB cluster** — mongod, mongos, rs.initiate, sh.addShard, pipeline
- **Neo4J / Cypher** — MATCH, MERGE, WITH, collect, UNION, OPTIONAL MATCH
- BD objeto-relacionales — SQL:1999, CREATE TYPE, ONLY()

</td>
</tr>
</table>

---

## 🚀 Cómo usar esta wiki

**1. Clona el repositorio y ábrelo en Obsidian**

```
git clone https://github.com/pabloseijo/examen-xine.git
```

Importa la carpeta `XINE/` como vault en Obsidian. Los enlaces `[[]]` se resuelven automáticamente.

**2. Lee primero los documentos de entrada**

Empieza por [`00-criterios-evaluacion`](wiki/temas/00-criterios-evaluacion.md) y [`00-examenes-reales`](wiki/temas/00-examenes-reales.md). Todo lo demás sale de ahí.

**3. Navega por el grafo de conceptos**

Usa [`wiki/index.md`](wiki/index.md) como mapa. Está organizado por bloques con prioridad de examen (⭐⭐⭐ = sale seguro, ⭐⭐⭐ incluye cálculos).

**4. Usa Claude como tutor**

Abre Claude Code en la raíz del repo. El [`CLAUDE.md`](CLAUDE.md) tiene un algoritmo BFS de routing para que Claude navegue la wiki de forma eficiente. Ejemplos de uso:

```
"explícame el índice invertido"      → Claude lee wiki/conceptos/indice-invertido.md
"repásame métricas RI"               → Claude formula preguntas tipo examen
"ingesta [archivo.pdf]"              → Claude procesa el PDF y crea páginas nuevas
"revisa la wiki"                     → Claude detecta lagunas y enlaces rotos
```

---

## 🎬 Cómo configurar Claude como brain de estudio

Tutorial sobre cómo montar este sistema de wiki mantenida por LLM:

<div align="center">

[![Cómo settear Claude como tu brain de estudio](https://img.youtube.com/vi/sboNwYmH3AY/maxresdefault.jpg)](https://www.youtube.com/watch?v=sboNwYmH3AY)

*▶ Haz clic para ver el tutorial*

</div>

---

## 📁 Estructura del repositorio

```
XINE/
├── CLAUDE.md                  ← instrucciones para el LLM + algoritmo BFS de routing
├── README.md                  ← este archivo
├── raw/                       ← PDFs originales (solo lectura)
│   └── assets/                ← imágenes generadas
└── wiki/
    ├── index.md               ← grafo de navegación + prioridad de examen ⭐
    ├── log.md                 ← historial de cambios (append-only)
    ├── overview.md            ← síntesis general de la asignatura
    ├── temas/                 ← resúmenes por PDF  (00-examenes-reales ← leer primero)
    ├── conceptos/             ← una página atómica por concepto
    ├── comparaciones/         ← análisis cruzados y tablas comparativas
    └── Apuntes/               ← notas de sesiones de estudio diarias
```

<details>
<summary><b>Páginas clave por prioridad de examen</b></summary>

### Bloque RI
| Prioridad | Página | Qué es |
|-----------|--------|--------|
| ⭐⭐⭐ | [indice-invertido](wiki/conceptos/indice-invertido.md) | AND/OR/NOT, phrase queries, optimización |
| ⭐⭐⭐ | [metricas-ri](wiki/conceptos/metricas-ri.md) | P@k, MAP, MRR, NDCG con cálculos |
| ⭐⭐⭐ | [pagerank](wiki/conceptos/pagerank.md) | Matriz P con teleportación |
| ⭐⭐⭐ | [tfidf](wiki/conceptos/tfidf.md) | TF-IDF, modelo vectorial, similitud coseno |
| ⭐⭐ | [bim](wiki/conceptos/bim.md) | BIM (pᵢ, rᵢ) + BM25 conceptual |

### Bloque BD
| Prioridad | Página | Qué es |
|-----------|--------|--------|
| ⭐⭐⭐ | [mongodb-cluster-comandos](wiki/conceptos/mongodb-cluster-comandos.md) | Todos los comandos del cluster de prácticas |
| ⭐⭐⭐ | [neo4j-cypher-practica](wiki/conceptos/neo4j-cypher-practica.md) | Importaciones JDBC + 10 patrones Cypher |
| ⭐⭐⭐ | [teorema-cap](wiki/conceptos/teorema-cap.md) | CAP centralizado vs distribuido |
| ⭐⭐⭐ | [nosql](wiki/conceptos/nosql.md) | 4 modelos, casos de uso, esquema relacional vs NoSQL |
| ⭐⭐ | [replicacion](wiki/conceptos/replicacion.md) | Maestro-esclavo vs peer-to-peer, Vector Stamp |

</details>

---

## 🤝 Contribuir

Esta wiki está pensada para ser compartida. Si quieres añadir contenido:

1. Abre una sesión de Claude Code en la raíz del repo
2. Dile que ingeste tu material: `"ingesta [archivo.pdf]"`
3. Claude creará las páginas siguiendo las convenciones del `CLAUDE.md` y actualizará el índice
4. Haz un PR con los cambios

---

<div align="center">

Generado con [Claude Code](https://claude.ai/code) · Asignatura XINE · Universidade de Santiago de Compostela

</div>
