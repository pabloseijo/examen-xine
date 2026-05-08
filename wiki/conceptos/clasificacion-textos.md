---
title: Clasificación de Textos
type: concepto
fuentes: [examenes.pdf]
última_actualización: 2026-05-07
---

# Clasificación de Textos

Tarea de asignar categorías/etiquetas a documentos basándose en su contenido. Caso particular de aprendizaje supervisado (variable discreta = clase).

**Aplicaciones**: detección de spam, clasificación de páginas, análisis de sentimiento, detección de contenido explícito.

## Métodos de clasificación

### Clasificación manual
Útil para colecciones pequeñas. No escalable.

### Reglas codificadas manualmente
Alta efectividad si las reglas son buenas. Caro de construir y mantener. Difícil encontrar expertos.

### Aprendizaje supervisado
El clasificador aprende automáticamente de datos de entrenamiento. Es el método estudiado en la asignatura.

## Naive Bayes

Clasificador probabilístico. Asume independencia entre términos (igual que BIM).
- Usa **pesos previos** de las clases y **parámetros condicionales** de cada palabra
- Entrenamiento y test muy rápidos
- Buenos resultados con características igual de importantes
- Robusto frente a características irrelevantes

## Rocchio

Calcula el **centroide** (vector medio) de cada clase y asigna a cada documento la clase del centroide más cercano.
- Centroide = media de las posiciones de los documentos de esa clase en el espacio vectorial
- NO necesita el vecino más cercano a todos, sino solo al centroide

## kNN (k Nearest Neighbors)

Obtiene los k documentos de entrenamiento más cercanos al documento a clasificar y le asigna la clase mayoritaria.
- **NO necesita entrenamiento** (lazy learning)
- Sensible a la calidad del training set
- Proceso: usar documento de test como query → resultados = vecinos cercanos

## Métricas de rendimiento

Además de Precision y Recall:
- **Accuracy**: proporción de documentos correctamente clasificados
- **F1 Score**: media armónica de precisión y recall: `F1 = 2 × (P × R) / (P + R)`

## En el examen

- Definir clasificación documental y por qué es aprendizaje supervisado
- Describir Naive Bayes, Rocchio y kNN conceptualmente (diferencias)
- Métricas: F1 + todas las de [[metricas-ri]]

## Relación con otros conceptos

- Usa representación vectorial de [[tfidf]]
- Las métricas se solapan con [[metricas-ri]] (precision, recall)
