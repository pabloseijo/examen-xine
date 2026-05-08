---
title: Métricas de Evaluación RI
type: concepto
fuentes: [examenes.pdf]
última_actualización: 2026-05-07
---

# Métricas de Evaluación RI

> **Pregunta fija en todos los exámenes**: calcular MRR, P@k, MAP, NDCG dado un ranking de resultados. Hay que saber aplicar las fórmulas con números concretos.

## Precisión (Precision)

Fracción de documentos recuperados que son relevantes:

```
Precision = relevantes recuperados / total recuperados
```

## Recall

Fracción de documentos relevantes que han sido recuperados:

```
Recall = relevantes recuperados / total relevantes en la colección
```

## P@k — Precision at k

Precisión considerando solo los primeros k resultados:

```
P@k = (documentos relevantes en el top k) / k
```

**Ejemplo**: si los primeros 5 resultados son [R, NR, R, NR, R] → P@5 = 3/5 = 0.6

## MAP — Mean Average Precision

Promedio de la Average Precision (AP) sobre múltiples queries.

**Para una sola query**: calcular precisión en cada posición donde aparece un documento relevante y promediar.

**Proceso**:
1. Para cada posición k donde hay un relevante, calcular P@k
2. Promediar esas precisiones → AP de esa query
3. Promediar las AP de todas las queries → MAP

**Ejemplo**: query con 3 relevantes en posiciones 1, 3, 5 (de 5 resultados):
- P@1 = 1/1 = 1.0
- P@3 = 2/3 = 0.67
- P@5 = 3/5 = 0.6
- AP = (1.0 + 0.67 + 0.6) / 3 = 0.76

**Ojo del examen 2025**: "el número de relevantes totales para cada query era mayor que el número de relevantes recuperados" → hay relevantes que no aparecen en el ranking, se incluyen como precisión 0 en el promedio.

## MRR — Mean Reciprocal Rank

Mide la posición del **primer documento relevante**. Útil cuando solo importa encontrar uno.

```
MRR = (1/Q) × Σ (1/rank_i)
```
- Q = número de queries
- rank_i = posición del primer relevante para la query i

**Ejemplo**: primer relevante en posición 3 → MRR = 1/3 ≈ 0.33

**Ejemplo con 2 queries**: primera en posición 2, segunda en posición 1:
MRR = (1/2)(1/2 + 1/1) = (1/2)(1.5) = 0.75

## NDCG — Normalized Discounted Cumulative Gain

Tiene en cuenta **grados de relevancia** (no solo binario). Los documentos más relevantes en posiciones más altas valen más.

**Paso 1 — DCG@p**:
```
DCG_p = Σᵢ₌₁ᵖ (2^relevancia_i - 1) / log₂(i+1)
```

**Paso 2 — IDCG@p**: DCG del ranking ideal (ordenado por relevancia descendente)

**Paso 3 — NDCG@p**:
```
NDCG_p = DCG_p / IDCG_p
```

**Ejemplo del examen** (NDCG@3 con relevancias [3, 2, 1] en posiciones 1,2,3):
- DCG@3 = (2³-1)/log₂(2) + (2²-1)/log₂(3) + (2¹-1)/log₂(4)
- = 7/1 + 3/1.585 + 1/2
- = 7 + 1.89 + 0.5 = 9.39
- IDCG@3 = DCG del ranking ideal (mismos valores si ya están ordenados) = 9.39
- NDCG@3 = 9.39/9.39 = 1.0

**Truco**: el examen dice "se pueden expresar las ecuaciones con los valores numéricos concretos sin necesidad de resolverlas" → puedes escribir la fórmula desarrollada con números y no calcular el resultado final.

## Evaluación por clicks

No clicks = relevancia, sino clicks = **preferencia**.
- Se tiene en cuenta el sesgo posicional: las posiciones más altas tienen más clicks por defecto
- Un doc es bueno si tiene más clicks de los esperados para su posición
- Se analiza también: tiempo entre clicks, orden de navegación

## Interleaved rankings y A/B testing

**Interleaving**: mezclar resultados de dos sistemas A y B (ABABAB...) y ver cuál recibe más clicks.
- Se suman puntos al sistema cuya página recibe el click
- Si ambos la devuelven en la misma posición: suma a ambos; si no: solo al que la puso más alta

**A/B testing**: mostrar el sistema nuevo a un porcentaje pequeño de usuarios.

## En el examen

Formato típico: tabla con 2 queries, cada una con una lista de resultados marcados como R/NR y número de relevantes totales en la colección.

1. **MRR**: para cada query, ¿en qué posición aparece el primer R? → 1/posición → promedio
2. **P@5**: contar R en primeras 5 posiciones / 5 → promedio de queries
3. **MAP**: para cada query, P@k en cada posición con R → promedio → promedio de queries
4. **NDCG@3**: necesitas valores de relevancia (1, 2, 3...) → aplicar fórmula DCG/IDCG

## Relación con otros conceptos

- Evalúan los rankings producidos por [[tfidf]], [[bm25]], [[pagerank]]
- [[pagerank]] usa su propio concepto de "relevancia" (estructura de links, no relevancia de usuario)
