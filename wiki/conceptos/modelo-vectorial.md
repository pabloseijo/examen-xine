---
title: Modelo Vectorial y Similitud del Coseno
type: concepto
fuentes: [examenes.pdf]
última_actualización: 2026-05-10
---

# Modelo Vectorial y Similitud del Coseno

![[modelo-vectorial.png]]

## La idea central

Cada documento y cada query se representa como un **vector** en un espacio multidimensional donde cada dimensión = un término del vocabulario. El valor de cada dimensión = su tf-idf.

**Ejemplo** con vocabulario {gato, perro}:

| | tf-idf(gato) | tf-idf(perro) |
|--|-------------|--------------|
| Doc1 | 1.5 | 1.0 |
| Doc2 | 0.3 | 2.8 |
| Query "gato" | 1.0 | 0.2 |

El gráfico de la izquierda muestra estos vectores. La query "gato" apunta casi en horizontal → Doc1 está mucho más alineado con ella que Doc2.

## Similitud del Coseno

No se mide distancia entre puntos sino el **ángulo entre vectores**. Dos vectores con la misma dirección tienen ángulo 0 → similitud 1. Perpendiculares → similitud 0.

```
sim(d, q) = (d · q) / (||d|| × ||q||)
```

- **d · q** = producto punto = Σ (peso_i_doc × peso_i_query) para cada término
- **||d||** = norma L2 del documento = √(Σ xᵢ²)
- **||q||** = norma L2 de la query

## Normalización por longitud

**El problema**: un documento largo acumula más ocurrencias de cualquier término → mayor producto punto aunque no sea más relevante.

**La solución**: dividir cada vector por su norma L2 → vector unitario (longitud = 1).

**El resultado**: el gráfico de la derecha muestra que Doc corto (1.0, 0.5) y Doc largo (3.0, 1.5) tienen exactamente la misma dirección — mismo contenido, distinta longitud. Tras normalizar, sus vectores son idénticos.

**Cómo responderlo en el examen**:

> Sin normalización, los documentos más largos tienen ventaja injusta porque acumulan más ocurrencias de cualquier término. La normalización L2 divide cada vector por su norma, haciendo que todos sean unitarios. Así la similitud mide únicamente la dirección del vector — el contenido — y no su magnitud — la longitud del documento.

## Relación con otros conceptos

- Los pesos de los vectores vienen de [[tfidf]]
- [[metricas-ri]] evalúan los rankings que produce este modelo
- [[bim]] y [[bm25]] son alternativas probabilísticas al modelo vectorial
