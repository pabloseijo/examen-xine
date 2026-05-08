---
title: Preprocesado de Texto
type: concepto
fuentes: [examenes.pdf]
última_actualización: 2026-05-07
---

# Preprocesado de Texto

Transformaciones aplicadas al texto antes de indexarlo (y a las queries antes de buscar). El mismo preprocesado debe aplicarse a documentos Y queries.

## Las 4 etapas

### 1. Tokenización
Separar la cadena de caracteres en tokens (unidades). Parece simple pero hay casos complejos: "New York", "don't", "C++", URLs.

### 2. Normalización
Modificar los tokens para que tengan el mismo formato que los elementos de la query. Ejemplos:
- Convertir a minúsculas: "Windows" → "windows"
- Eliminar puntuación
- Expandir contracciones

### 3. Stemming
Reducir las palabras a su raíz morfológica. **No siempre produce palabras reales.**
- "correr", "corriendo", "corrió" → "corr"
- Algoritmo más usado: Porter stemmer

**Diferencia con lematización**: el lema es la forma canónica real ("ran" → "run"), el stem puede ser un truncamiento artificial.

### 4. Stopword removal
Eliminar palabras muy comunes que aportan poco significado y consumen mucho espacio en el índice ("el", "la", "de", "y"...).

**Cuidado**: en web retrieval NO se eliminan siempre — "to be or not to be" necesita las stopwords para phrase queries.

**Efecto en el índice**: las posting lists de stopwords son las más grandes del índice. Eliminarlas reduce drásticamente el tamaño.

## En el examen

- Pregunta teórica: definir cada paso y su propósito
- Tipo test: "¿qué término tiene mayor IDF?" → el que aparece en menos documentos tras el preprocesado
- Efecto de eliminar stopword: afecta principalmente al tamaño de las **posting lists** (no tanto al diccionario)

## Relación con otros conceptos

- Determina qué entra en el [[indice-invertido]]
- Afecta al cálculo de [[tfidf]] (si se hace stemming, "car" y "cars" comparten peso)
- Las stopwords tienen IDF ≈ 0 → su eliminación tiene poco impacto en ranking [[tfidf]]
