
## TF — Term Frequency

El problema del modelo booleano es que solo dice si un término está o no está. Pero intuitivamente, si "gato" aparece 10 veces en un documento y 1 vez en otro, el primero es más relevante para una búsqueda sobre gatos.

**TF(t, d)** = número de veces que el término t aparece en el documento d.

Pero la relevancia no crece proporcionalmente — que aparezca 10 veces no lo hace 10 veces más relevante que si aparece 1. Por eso se pondera logarítmicamente:

```
w(t,d) = 1 + log₁₀(tf)   si tf > 0
         0                 si tf = 0
```

---

## IDF — Inverse Document Frequency

El problema del TF solo es que no distingue entre términos importantes y términos comunes. "El" puede aparecer 50 veces en un documento, pero no dice nada útil.

La solución: penalizar los términos que aparecen en muchos documentos.

```
idf(t) = log₁₀(N / df(t))
```

- **N** = total de documentos en la colección
- **df(t)** = número de documentos que contienen t

Cuanto más raro es el término, mayor es su IDF. Si aparece en todos los documentos, idf = log(N/N) = 0.

---

## TF-IDF

Se combinan los dos:

```
tf-idf(t, d) = w(t,d) × idf(t)
```

Y la puntuación de un documento para una query es la suma de los tf-idf de los términos que comparten:

```
Score(q, d) = Σ tf-idf(t, d)    para cada t que esté en q y en d
```

**Ejemplo** con N=100 documentos:

- Doc1 tiene "car" 3 veces, "insurance" 2 veces
- Doc2 tiene "car" 1 vez, "insurance" 5 veces
- df("car") = 10, df("insurance") = 2

Calculamos IDF:

- idf("car") = log(100/10) = 1
- idf("insurance") = log(100/2) ≈ 1.7

Calculamos TF ponderado (w):

- w("car", Doc1) = 1 + log(3) ≈ 1.48
- w("insurance", Doc1) = 1 + log(2) ≈ 1.30
- w("car", Doc2) = 1 + log(1) = 1
- w("insurance", Doc2) = 1 + log(5) ≈ 1.70

TF-IDF:

- "car" en Doc1: 1.48 × 1 = 1.48
- "insurance" en Doc1: 1.30 × 1.7 = 2.21
- "car" en Doc2: 1 × 1 = 1
- "insurance" en Doc2: 1.70 × 1.7 = 2.89

---
