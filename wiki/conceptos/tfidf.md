---
title: TF-IDF y Modelo Vectorial
type: concepto
fuentes: [examenes.pdf]
última_actualización: 2026-05-07
---

# TF-IDF y Modelo Vectorial

## TF — Term Frequency

Número de veces que el término t aparece en el documento d. Se pondera logarítmicamente porque la relevancia no crece proporcionalmente:

```
w(t,d) = 1 + log₁₀(tf(t,d))   si tf > 0
         0                      si tf = 0
```

## IDF — Inverse Document Frequency

Penaliza términos que aparecen en muchos documentos (poco discriminativos):

```
idf(t) = log₁₀(N / df(t))
```
- N = número total de documentos
- df(t) = número de documentos que contienen t

**Propiedades del IDF**:
- Mayor cuando t aparece en pocos documentos → término discriminativo
- Menor cuando t aparece en muchos documentos → término común
- = 0 cuando t aparece en todos los documentos

**Ejemplo** (N=100):
- "car": df=10 → idf = log(100/10) = 1
- "insurance": df=2 → idf = log(100/2) = 1.7

## TF-IDF

```
tf-idf(t,d) = tf(t,d) × idf(t)
```
O si se usa tf ponderado: `w(t,d) × idf(t)`

**Puntuación de un documento para una query**:
```
Score(q, d) = Σ tf-idf(t,d)    para t ∈ q ∩ d
```
Solo se suman los términos que aparecen tanto en la query como en el documento.

## Modelo Vectorial

Documentos y consultas se representan como vectores en un espacio donde cada dimensión = un término del vocabulario. El peso de cada dimensión = tf-idf.

- Documento d = (w(t₁,d), w(t₂,d), ..., w(tₙ,d))
- Query q = vector similar

## Similitud del Coseno

Para comparar un documento con una query:

```
sim(d, q) = cos(θ) = (d · q) / (||d|| × ||q||)
```

- **Producto punto** d·q: suma de productos de pesos de términos comunes
- **||d||**: norma L2 del vector documento = √(Σ xᵢ²)
- **||q||**: norma L2 del vector query

## Normalización por longitud

**¿Por qué normalizar?** Sin normalización, documentos más largos tienen ventaja injusta (más ocurrencias de cualquier término). La normalización hace que dos documentos con el mismo contenido pero diferente longitud sean equivalentes.

> Un vector se normaliza dividiendo cada componente por su norma L2. Resultado: vector unitario (||d|| = 1).

**Trampa del examen**: "si d' es d adjuntado a sí mismo (d'=ddd), después de normalizar L2, d y d' tienen vectores idénticos." → Esto justifica la normalización.

## En el examen

- Calcular TF-IDF de un término en un documento → aplicar fórmulas paso a paso
- Calcular Score(q,d) sumando tf-idf de términos comunes
- Calcular similitud coseno: producto punto dividido entre producto de normas
- **Tipo test**: "¿qué término tiene mayor IDF?" → el que aparece en menos documentos

## Relación con otros conceptos

- Extiende el [[indice-invertido]] con ponderación
- [[bim]] y [[bm25]] son alternativas probabilísticas
- [[metricas-ri]] evalúan los rankings que produce este modelo
