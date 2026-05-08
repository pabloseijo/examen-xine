---
title: Aspectos fundamentales para el examen — Parte RI
type: tema
fuentes: [Xestión de Información non Estruturada [G4012453] [2025_2026]_ Aspectos fundamentales de cara al examen de teoría de la parte de RI _ Campus Virtual.pdf]
última_actualización: 2026-05-07
---

# Aspectos fundamentales para el examen — Parte RI

> Lista oficial del profesor (última modificación: 22 de abril de 2026). Cubre la parte de Recuperación de Información (RI) del examen de teoría.

---

## Lista oficial de temas a dominar

### 1. Qué es la Recuperación de Información (RI)
- Diferencia entre **RI** y **Recuperación Estructurada** (estilo bases de datos relacionales)
- En RI los documentos son texto no estructurado; la relevancia es subjetiva y gradual
- En BD estructuradas la consulta devuelve resultados exactos (match exacto)

### 2. Modelos booleanos de búsqueda
- Documentos son conjuntos de términos (presencia/ausencia)
- Consultas: AND, OR, NOT
- Resultado es binario: relevante o no relevante
- Problema: no hay ranking, ni tratamiento de relevancia gradual

### 3. Índices invertidos
- Estructura central de cualquier motor de búsqueda
- Mapea cada término a la lista de documentos que lo contienen (**posting list**)
- **Índice sin información posicional**: `término → [doc1, doc2, doc3]`
- **Índice posicional**: `término → [doc1:(pos1,pos2), doc2:(pos3)]`
- Saber **dibujar** un índice invertido a partir de un corpus de ejemplo
- Saber construirlo **con** y **sin** información posicional

### 4. Preprocesado de texto
| Paso | Descripción | Ejemplo |
|------|-------------|---------|
| **Tokenización** | Dividir texto en tokens (palabras) | "running fast" → ["running","fast"] |
| **Normalización** | Minúsculas, eliminar signos | "USA" → "usa" |
| **Stemming** | Reducir a raíz morfológica | "running" → "run" |
| **Stopword removal** | Eliminar palabras vacías | "the", "is", "a" → eliminadas |

### 5. Procesamiento de queries booleanas
- **Intersección de posting lists**: algoritmo merge para AND
- Ordenar posting lists por longitud (menor primero) para eficiencia
- Complejidad O(n+m) para intersección de dos listas

### 6. Índice posicional para phrase queries
- Permite responder a consultas de frase: "Nueva York" como frase exacta
- Requiere que los términos aparezcan en posiciones consecutivas en el mismo documento
- Algoritmo: intersección de posting lists + comprobación de posiciones adyacentes

### 7. Indexado distribuido — Map-Reduce
- **Map**: para cada par (término, documento) emite (término, docID)
- **Reduce**: agrega todas las entradas con la misma clave (término) para construir la posting list
- Permite construir índices sobre corpus masivos en paralelo
- Ver [[map-reduce]] en el contexto de NoSQL para detalles del paradigma

### 8. Ranked retrieval — tf/idf
- **tf (term frequency)**: frecuencia del término en el documento
  - Variante logarítmica: `tf = 1 + log(count)` si count > 0, 0 si no aparece
- **idf (inverse document frequency)**: `idf = log(N / df_t)` donde N = total docs, df_t = docs con el término
- **Peso tf-idf**: `w = tf × idf`
- Términos frecuentes en el documento pero raros en la colección tienen más peso

### 9. Modelo vectorial de RI
- Documentos y consultas representados como vectores en espacio de términos
- Cada dimensión corresponde a un término del vocabulario
- **Similitud coseno**: `sim(d,q) = (d·q) / (|d|·|q|)`
- Ranking por similitud descendente
- Los documentos más similares al vector de consulta son los más relevantes

### 10. Normalización por longitud de documento
- Sin normalización, documentos largos tienen ventaja por mayor frecuencia bruta de términos
- Normalización coseno: dividir el vector del documento por su módulo
- Variante pivoted: reduce agresividad de la normalización

### 11. Modelo probabilístico — BIM (Binary Independence Model)
- **Aspectos básicos** (sin memorizar fórmulas exactas)
- Cada documento representado como vector binario de presencia/ausencia
- Independencia entre términos (supuesto)
- Estimar probabilidades:
  - **p_i**: P(término i presente | documento relevante)
  - **r_i**: P(término i presente | documento no relevante)
- **Saber estimarlas a partir de tabla** de ocurrencia de términos en docs relevantes e irrelevantes:
  - `p_i = (r_i + 0.5) / (R + 1)` (con suavizado)
  - `r_i = (n_i - r_i + 0.5) / (N - R + 1)`
  - Donde R = docs relevantes totales, n_i = docs con término i

### 12. BM25
- Mejoras sobre BIM:
  1. **Term frequency**: incorpora la frecuencia del término (BIM solo usa presencia/ausencia)
  2. **Longitud del documento**: penaliza documentos largos (normalización)
  3. **Term frequency en la consulta**: pondera términos que aparecen más en la query
- Parámetros: k1 (saturación de tf, típicamente 1.2–2.0), b (normalización de longitud, típicamente 0.75)
- En la práctica supera a tf-idf clásico en la mayoría de colecciones

### 13. Métricas de evaluación — saber computarlas
| Métrica | Definición | Cuándo usarla |
|---------|------------|----------------|
| **Precision** | TP / (TP + FP) | ¿cuántos recuperados son relevantes? |
| **Recall** | TP / (TP + FN) | ¿cuántos relevantes se recuperaron? |
| **P@k** | Precisión en los primeros k resultados | Evaluación de ranking |
| **MAP** | Media de Average Precision sobre queries | Evaluación global de sistema |
| **MRR** | Media del recíproco del rango del primer relevante | Cuando solo importa el 1er resultado |
| **NDCG** | Normalized Discounted Cumulative Gain | Rankings con relevancia gradual (1,2,3) |

**Cómo computar MAP**: para cada query, calcular AP (área bajo curva P-R). AP = media de P@k en cada posición k donde hay un documento relevante. MAP = media de AP sobre todas las queries.

**Cómo computar NDCG**:
- DCG@k = sum_{i=1}^{k} rel_i / log2(i+1)
- IDCG@k = DCG@k del ranking ideal
- NDCG@k = DCG@k / IDCG@k

### 14. Evaluación mediante clicks
- Clicks como señal implícita de relevancia
- **Interleaved rankings**: mezclar resultados de dos sistemas A y B, observar qué resultados reciben más clicks
- **A/B testing**: dividir usuarios aleatoriamente, mostrar sistema A a un grupo y B a otro, medir engagement

### 15. Clasificación de textos
- **Tarea**: asignar categorías predefinidas a documentos de texto
- Aplicaciones: spam, sentimiento, categorización de noticias
- **Métodos**:
  - **Reglas manuales**: IF término X → categoría Y
  - **Naive Bayes**: modelo probabilístico, asume independencia entre términos, eficiente
  - **kNN (k-nearest neighbors)**: asigna la categoría de los k vecinos más similares
  - **Rocchio**: representa cada clase como centroide, clasifica por distancia al centroide
  - Aprendizaje supervisado en general (SVM, redes neuronales...)
- **Métricas**: accuracy, precisión/recall por clase, F1, macro/micro averaging

### 16. Link analysis — PageRank y HITS
- **Anchor text**: texto de un enlace da información sobre el contenido del destino
- **PageRank**: importancia de una página = suma ponderada de importancias de páginas que la enlazan
  - Fórmula: `PR(d) = (1-d)/N + d * sum(PR(t)/C(t))` para páginas t que enlazan a d
  - d = factor de amortiguación (~0.85), C(t) = grado de salida de t
  - Interpretación: probabilidad de que un surfista aleatorio llegue a la página
- **HITS (Hubs and Authorities)**:
  - **Authority**: página referenciada por muchos buenos hubs
  - **Hub**: página que enlaza a muchas buenas authorities
  - Cálculo iterativo: auth(d) = sum hub(t) para todos t que enlazan a d; hub(d) = sum auth(t) para todos t a los que d enlaza

### 17. Crawling
- Proceso de descubrimiento y descarga automática de páginas web
- Componentes: frontier (cola de URLs pendientes), downloader, parser de enlaces
- Desafíos: escala, cortesía (robots.txt, delays), contenido duplicado, páginas dinámicas

---

## Prioridad de estudio (según énfasis del profesor)

1. **Índices invertidos** — saber dibujar, con y sin posición
2. **tf-idf** y **modelo vectorial** — saber calcular pesos y similitud coseno
3. **Métricas** — saber computar MAP, NDCG, MRR a mano
4. **BIM** — saber estimar p_i y r_i desde tabla
5. **Map-Reduce para indexado**
6. **BM25** — diferencias con BIM
7. **PageRank** — concepto e iteración
