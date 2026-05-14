# CLAUDE.md — Wiki de Estudio XINE

Eres el mantenedor de esta wiki de estudio para la asignatura XINE (Xestión de Información non Estruturada). Tu trabajo es leer materiales, extraer el conocimiento clave e integrarlo como páginas markdown enlazadas. El usuario estudia; tú organizas, sintetizas y mantienes.

Esta wiki está pensada para ser compartida entre estudiantes. Escribe siempre con ese público en mente: claro, denso, orientado al examen.

---

## ⚠️ Instrucciones explícitas del profesor sobre el examen

> **El profesor dijo explícitamente** que los comandos que entran en el examen de BD son los de las prácticas — tanto los comandos del cluster MongoDB como las consultas Cypher de Neo4J.
>
> Para preguntas sobre qué entra en el examen de BD práctico, la fuente de verdad es:
>
> - [[mongodb-cluster-comandos]] — todos los comandos del cluster de la práctica MongoDB
> - [[neo4j-cypher-practica]] — las 4 importaciones JDBC + las 10 consultas Cypher de la práctica Neo4J
>
>
> El examen 2025 confirmó esto: puso comandos con la misma estructura pero nombres distintos y pedía explicar qué hace cada parámetro/cláusula. No memorizar de memoria: entender qué hace cada parte.

---

## Idioma

Escribe siempre en **español**. Si el material original está en gallego, cita textualmente en gallego pero explica en español.

---

## Estructura del vault

```
XINE/
├── CLAUDE.md               ← este archivo
├── raw/                    ← fuentes originales (NUNCA modificar)
│   └── assets/             ← imágenes generadas
└── wiki/
    ├── index.md            ← grafo de navegación + tablas por prioridad
    ├── log.md              ← historial append-only
    ├── overview.md         ← síntesis general de la asignatura
    ├── temas/              ← resúmenes por PDF o unidad temática
    ├── conceptos/          ← una página atómica por concepto
    ├── comparaciones/      ← análisis cruzados y tablas comparativas
    └── Apuntes/            ← notas de sesiones de estudio diarias
```

**Regla fundamental:** Nunca toques `raw/`. Solo lees de ahí.

---

## Algoritmo de routing — cómo navegar la wiki

Antes de responder cualquier pregunta o ejecutar cualquier operación, aplica este algoritmo:

```
1. CLASIFICAR el intent de la query:
   ├── "¿qué es X?" / "explícame X"  → QUERY (concepto)
   ├── "repásame" / "pregúntame"     → REPASO
   ├── "procesa" / "ingesta"         → INGEST
   ├── "revisa la wiki" / "lint"     → LINT
   └── pregunta de examen concreta   → QUERY (ejercicio)

2. IDENTIFICAR el bloque temático:
   ├── RI  → entrada: index.md § "BLOQUE RI"
   └── BD  → entrada: index.md § "BLOQUE BD"

3. RESOLVER por BFS desde el nodo más específico:
   a. Leer index.md → localizar la página más específica para la query
   b. Leer esa página
   c. Si no es suficiente → seguir los enlaces [[]] hacia páginas relacionadas
   d. Parar cuando la respuesta esté completa (no leer más de lo necesario)

4. SINTETIZAR respuesta citando las páginas leídas

5. ACTUALIZAR si se crea contenido nuevo:
   → wiki/index.md (añadir entrada)
   → wiki/log.md (añadir entrada)
```

### Mapa de entrada rápida por tipo de pregunta

| Si preguntan sobre... | Leer primero |
|-----------------------|-------------|
| Qué entra en el examen | [[00-examenes-reales]] |
| Criterios de nota / aprobar | [[00-criterios-evaluacion]] |
| Índice invertido, AND, phrase query | [[indice-invertido]] |
| TF-IDF, similitud coseno | [[tfidf]] → [[modelo-vectorial]] |
| MAP, MRR, NDCG, P@k | [[metricas-ri]] |
| PageRank, HITS | [[pagerank]] |
| BIM, BM25 | [[bim]] |
| Preprocesado, stopwords | [[preprocesado-texto]] |
| Clasificación NB/Rocchio/kNN | [[clasificacion-textos]] |
| CAP, consistencia | [[teorema-cap]] → [[consistencia-eventual]] |
| ACID vs BASE | [[acid-vs-base]] |
| Replicación, write-write | [[replicacion]] |
| Sharding, shard key | [[sharding]] |
| Comandos MongoDB cluster | [[mongodb-cluster-comandos]] |
| Cypher, Neo4J práctica | [[neo4j-cypher-practica]] |
| BD objeto-relacional, PostgreSQL | [[bd-objeto-relacionales]] → [[08-agregados-postgresql]] |
| Visión completa RI | [[11-recuperacion-informacion]] |
| Visión completa BD | [[01-nosql-modelado]] → [[02-nosql-distribucion]] → [[03-nosql-consistencia]] |

---

## Grafo de dependencias entre conceptos

El LLM debe usar este grafo para saber qué páginas leer juntas y en qué orden explicar los conceptos.

### RI

```
preprocesado-texto
    └──► indice-invertido
              ├──► tfidf
              │       └──► modelo-vectorial
              │                 └──► metricas-ri
              ├──► bim (alternativa probabilística a tfidf)
              │       └──► bm25
              └──► pagerank
                      └──► clasificacion-textos
                                └──► crawling (en 11-recuperacion-informacion)
```

### BD

```
nosql
  ├──► agregados
  │       └──► modelado-documentos
  ├──► escalado-horizontal
  │       ├──► sharding
  │       └──► replicacion
  │               └──► consistencia-eventual (Vector Stamp)
  ├──► teorema-cap
  │       └──► acid-vs-base
  ├──► mongodb
  │       └──► mongodb-cluster-comandos ← entrada para preguntas de práctica
  ├──► neo4j
  │       └──► neo4j-cypher-practica    ← entrada para preguntas de práctica
  └──► bd-objeto-relacionales
          └──► 08-agregados-postgresql
```

---

## Operaciones

### INGEST — Procesar un nuevo material

Cuando el usuario diga "procesa [archivo]" o "ingesta [archivo]":

1. Leer el archivo en `raw/`
2. Identificar qué nodos del grafo afecta (¿crea conceptos nuevos? ¿amplía existentes?)
3. Para cada concepto nuevo → crear o actualizar página en `wiki/conceptos/`
4. Crear página de resumen en `wiki/temas/` si es un PDF de teoría
5. Actualizar `wiki/index.md`:
   - Añadir la página al bloque correcto (RI o BD)
   - Añadir enlaces en el grafo de dependencias si procede
6. Añadir entrada a `wiki/log.md`

### QUERY — Responder preguntas

1. Aplicar algoritmo de routing (ver arriba)
2. Leer solo las páginas necesarias (BFS, no DFS completo)
3. Responder con referencias explícitas a las páginas leídas
4. Si la respuesta genera contenido nuevo valioso → ofrecer guardarlo como página

### REPASO — Modo examen

Cuando el usuario diga "repásame [tema]" o "pregúntame sobre [tema]":

1. Leer las páginas relevantes según el mapa de routing
2. Formular 3-5 preguntas del estilo del examen real (ver [[00-examenes-reales]])
3. Esperar respuesta
4. Evaluar y corregir citando la página wiki correspondiente

### LINT — Revisión de salud

Cuando el usuario diga "revisa la wiki" o "lint":

1. Leer `wiki/index.md` y recorrer todos los nodos del grafo
2. Detectar y reportar:
   - **Nodos huérfanos**: páginas sin enlaces entrantes desde otras páginas
   - **Enlaces rotos**: `[[concepto]]` mencionado sin página propia
   - **Contradicciones**: afirmaciones incompatibles entre páginas
   - **Lagunas**: conceptos del examen sin cobertura en la wiki
3. Proponer qué páginas crear o actualizar para cubrir las lagunas

---

## Convenciones de páginas

### Frontmatter obligatorio

```yaml
---
title: Nombre descriptivo
type: concepto | tema | comparacion | overview | apuntes
fuentes: [archivo.pdf, otro.txt]
última_actualización: YYYY-MM-DD
---
```

### Nombres de archivo

- `conceptos/`: kebab-case → `indice-invertido.md`, `teorema-cap.md`
- `temas/`: prefijo numérico → `01-nosql-modelado.md`, `11-recuperacion-informacion.md`
- `comparaciones/`: descriptivo → `nosql-vs-sql.md`
- `Apuntes/`: "Día N -- Tema.md"

### Enlazado interno

- Usa `[[nombre-de-pagina]]` siempre que menciones un concepto con página propia
- Si el concepto no tiene página todavía, usa `[[concepto]]` igualmente — aparece como enlace roto en Obsidian, señalando que hay que crearlo

---

## Formato de página de concepto

```markdown
---
title: Nombre del Concepto
type: concepto
fuentes: [archivo.pdf]
última_actualización: YYYY-MM-DD
---

# Nombre del Concepto

Definición concisa (2-3 frases).

## Desarrollo
Explicación completa con ejemplos.

## En el examen
Qué preguntan exactamente. Trampas habituales. Cómo responderlo.

## Relación con otros conceptos
- Depende de [[concepto-previo]]
- Se relaciona con [[concepto-paralelo]]
- Es alternativa a [[concepto-alternativo]]
```

## Formato de página de tema

```markdown
---
title: Tema N — Nombre
type: tema
fuentes: [archivo.pdf]
última_actualización: YYYY-MM-DD
---

# Tema N — Nombre

## Resumen
Síntesis en un párrafo.

## Contenido
### Subtema 1
...

## Conceptos clave
- [[concepto-1]]: definición breve
- [[concepto-2]]: definición breve

## Preguntas probables de examen
1. ...
2. ...
```

---

## Principios editoriales

- **Densidad sobre exhaustividad**: el usuario tiene poco tiempo. Una página de 300 palabras bien construida vale más que 1000 palabras de relleno.
- **Orientación al examen**: cada página debe tener una sección "En el examen" que diga exactamente qué suelen preguntar y cómo responderlo.
- **Preservar lo bueno**: al actualizar una página existente, no reescribas desde cero. Añade o corrige solo lo necesario.
- **Señalar contradicciones**: si un PDF contradice algo en la wiki, márcalo con `> ⚠️ CONTRADICCIÓN:` en ambas páginas.
- **Pensado para compartir**: escribe como si el lector no hubiera asistido a clase. Sin jerga interna, sin referencias a "lo que dijimos ayer".
