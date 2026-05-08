---
title: PageRank y HITS
type: concepto
fuentes: [examenes.pdf]
última_actualización: 2026-05-07
---

# PageRank y HITS

> **Pregunta fija en todos los exámenes**: dado un grafo + probabilidad de teleportación (5%), calcular una fila de la matriz P.

## PageRank

Mide la "visitabilidad" de una página — cuántas veces llegarías a ella en un random walk por la web. Valor entre 0 y 1.

**Intuición**: una página tiene alto PageRank si:
- Muchos otros la enlazan (muchos inlinks)
- Y esas páginas también tienen alto PageRank
- Y pocos outlinks (el valor se "diluye" menos)

### Modelo del random surfer

Un usuario aleatorio:
- Con probabilidad **(1-d)** = 5% → salta a cualquier página aleatoriamente (**teleportación**)
- Con probabilidad **d** = 95% → sigue un enlace aleatorio desde la página actual

### Matriz P (lo que pide el examen)

La matriz de transición P combina:
1. **Matriz de adyacencia M**: M[i][j] = 1/outlinks(j) si j enlaza a i, 0 si no
2. **Teleportación**: (1/N) para todos los nodos

```
P = (1 - d) × (1/N) × [matrix de 1s] + d × M
```

Donde d = probabilidad de seguir enlace (0.95 si teleportación = 5%).

**Cómo calcular una fila de P** (lo que pide el examen):

Para la fila i (página i):
- Para cada columna j:
  - Si j enlaza a i: P[i][j] = (1-d)/N + d × (1/outlinks(j))
  - Si j NO enlaza a i: P[i][j] = (1-d)/N

**Ejemplo** con d=0.95 (teleport=5%), N=4 páginas, nodo j tiene 2 outlinks y uno va a i:
```
P[i][j] = (0.05/4) + 0.95 × (1/2) = 0.0125 + 0.475 = 0.4875
```

Si j no enlaza a i:
```
P[i][j] = 0.05/4 = 0.0125
```

### Propiedades

- **Independiente de la query**: se calcula offline para toda la web
- **Convergencia**: iterar multiplicación de vectores por P hasta estabilizar
- Suma de cada columna de M = 1 (matriz estocástica)

## HITS (Hyperlink-Induced Topic Search)

Planteado por Kleinberg en 1999. A diferencia de PageRank:
- **Depende de la query** (se calcula en tiempo de búsqueda)
- Clasifica páginas en dos tipos con scores diferentes

### Hubs y Autoridades

- **Autoridad** (authority score): página que responde directamente la consulta. Una autoridad es buena por su **propio contenido**. Una buena autoridad es apuntada por muchos buenos hubs.
- **Hub** (hub score): página que no responde la consulta pero enlaza a páginas relevantes. Un hub es bueno por los **enlaces que proporciona**. Un buen hub enlaza a muchas buenas autoridades.

**Mutualismo**: buenos hubs → señalan buenas autoridades → que a su vez son señaladas por buenos hubs.

### Diferencias con PageRank

| | PageRank | HITS |
|--|---------|------|
| Momento de cálculo | Offline (global) | Online (por query) |
| Dependencia | Independiente de query | Dependiente de query |
| Scores | Uno por página | Dos: hub + authority |
| Mejor para | Encontrar páginas concretas | Temas amplios |

## Anchor text

El texto descriptivo de un hipervínculo describe a la página enlazada. Aporta información que el creador de la página enlazada no controla → señal de calidad no manipulable fácilmente.

**Dos hipótesis**:
1. Un enlace entre páginas = concesión de autoridad / señal de calidad
2. El anchor text describe a la página enlazada

## En el examen

**PageRank** — pasos:
1. Identificar el grafo (quién enlaza a quién, cuántos outlinks tiene cada nodo)
2. Aplicar fórmula P[i][j] para cada celda de la fila pedida
3. Si teleportación = 5% → d = 0.95, factor aleatorio = 0.05/N

**HITS** — típicamente teórico:
- Definir hub y autoridad con sus propias palabras
- Diferencia con PageRank (offline vs. online, query-dependent)

## Relación con otros conceptos

- Son alternativas/complementos a [[tfidf]] para ranking web
- [[metricas-ri]] evalúan la calidad de los rankings producidos
- La web como grafo es análoga a las BD de [[grafos]] en NoSQL
