---
title: Índice Invertido
type: concepto
fuentes: [examenes.pdf]
última_actualización: 2026-05-07
---

# Índice Invertido

Estructura de datos que mapea términos a los documentos (y posiciones) donde aparecen. Base de todos los motores de búsqueda. Ha salido **en todos los exámenes**.

## Estructura

**Diccionario**: conjunto de términos ordenados alfabéticamente (vocabulary/lexicon).
**Posting list**: para cada término, lista de docIDs donde aparece, ordenada por docID.

```
Brutus   →  [1, 2, 4, 11, 31, 45, 173, 174]
Caesar   →  [1, 2, 4, 5, 6, 16, 57, 132]
Calpurnia → [2, 31, 54, 101]
```

**Por qué ordenada por docID**: permite intersección eficiente O(x+y) con merge algorithm.

## Sin información posicional vs. con información posicional

### Sin posición
Solo almacena en qué documentos aparece el término. Ejemplo corpus:
- Doc1: "la casa verde"
- Doc2: "la casa azul"

| Término | Documentos |
|---------|-----------|
| la | 1, 2 |
| casa | 1, 2 |
| verde | 1 |
| azul | 2 |

Sirve para: queries booleanas AND/OR/NOT.
**No sirve para**: phrase queries.

### Con posición (índice posicional)
Almacena docID + posiciones exactas dentro del documento.

| Término | Documentos y posiciones |
|---------|------------------------|
| la | 1:(1), 2:(1) |
| casa | 1:(2), 2:(2) |
| verde | 1:(3) |
| azul | 2:(3) |

Sirve para: queries booleanas + **phrase queries**.

## Queries AND (intersección de posting lists)

**Proceso** (lo que pregunta el examen):
1. Buscar cada término en el diccionario → obtener sus posting lists
2. Intersecar las listas (merge algorithm): dos punteros avanzan por las listas ordenadas
3. Complejidad: O(x + y) donde x, y = tamaños de las listas

**Optimización** (2025 lo pide explícitamente):
- Procesar primero los términos con posting list **más corta** (menor df)
- Para 3+ términos: ordenar por frecuencia ascendente y hacer intersecciones sucesivas
- Esto reduce el tamaño de las intersecciones intermedias

**Ejemplo AND query**: `sistema AND operativo AND Windows`
1. Obtener posting lists de los 3 términos
2. Ordenar por tamaño: supongamos Windows < sistema < operativo
3. Intersecar Windows ∩ sistema → resultado intermedio
4. resultado ∩ operativo → resultado final

## Phrase queries (índice posicional)

Para la frase "A B" (dos términos consecutivos):
1. Obtener posting lists posicionales de A y B
2. Para cada docID en común, verificar si hay posición p en A tal que p+1 esté en B

**Ejemplo del examen** — phrase query "to be":
- `to`: doc1:[1,19,22]; doc2:[1,39,42]
- `be`: doc1:[2,21,25]; doc2:[1,40,41]
- Doc1: ¿hay p en {1,19,22} tal que p+1 ∈ {2,21,25}? → 1→2 ✓, 19→? no, 22→? no → doc1 posición 1
- Doc2: ¿hay p en {1,39,42} tal que p+1 ∈ {1,40,41}? → 1→? no, 39→40 ✓, 42→? no → doc2 posición 39

**Phrase query "be to"** (orden inverso):
- Ahora buscamos posición p en `be` tal que p+1 ∈ `to`
- Doc1: p∈{2,21,25}, p+1∈{1,19,22}? → 2→? no, 21→? no, 25→? no → NO
- Doc2: p∈{1,40,41}, p+1∈{1,39,42}? → 1→? no, 40→? no, 41→42 ✓ → doc2 posición 41

## Efecto de eliminar una stopword

Pregunta del examen: "¿en qué nivel de tamaño afecta más eliminar una stopword?"

- Las stopwords ("el", "la", "de"...) aparecen en **casi todos los documentos** → posting list enormes
- Eliminarlas del diccionario reduce enormemente el tamaño total del índice
- **Afecta más al nivel de las posting lists** (no al diccionario, que solo pierde una entrada)
- El diccionario pierde 1 término; las postings pierden la lista más grande del índice

## En el examen

- **Construir índice** a partir de corpus → tabular términos por documento
- **AND query**: describir el merge algorithm paso a paso
- **Optimización AND**: ordenar términos por df ascendente, procesar menor primero
- **Phrase query**: verificar posiciones p y p+1 en los posting lists posicionales
- **Stopword**: la posting list de una stopword es la más grande del índice

## Relación con otros conceptos

- Base del [[modelo-vectorial]] y [[tfidf]]
- Permite [[queries-booleanas]] eficientes
- Versión distribuida con [[map-reduce]]
