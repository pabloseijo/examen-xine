---
title: Tema 11 — Recuperación de Información (RI)
type: tema
fuentes: [examenes.pdf]
última_actualización: 2026-05-07
---

# Tema 11 — Recuperación de Información (RI)

> Bloque completo de RI. La parte RI del examen es **casi idéntica cada año**. Ver [[00-examenes-reales]] para las preguntas exactas.

## RI vs. Recuperación Estructurada (RE)

| | RI | RE (SQL) |
|-|----|---------|
| Datos | No estructurados (texto) | Estructurados (tablas) |
| Consulta | Booleana / ranked / lenguaje natural | SQL |
| Resultado | Ranking por relevancia | Conjunto exacto |
| Relevancia | Subjetiva, varía por usuario | Exacta |
| Ejemplo | Google | SELECT * FROM... |

**Definiciones**:
- **Necesidad de información**: concepto que queremos obtener ("cómo hacer bizcocho")
- **Query**: consulta concreta ("bizcocho receta")
- **Colección/corpus**: conjunto de documentos donde se busca
- **Precisión**: fracción recuperada que es relevante
- **Recall**: fracción de relevantes que se recuperan

## Modelos de búsqueda booleana

### Matriz de incidencia término-documento
Matriz binaria: fila=término, columna=documento, valor=1 si el término está en el documento.
- Ineficiente: 1M documentos × 500K términos = 1 trillón de celdas (mayormente 0)

### Índice invertido (solución eficiente)
Solo almacena en qué documentos aparece cada término. Ver [[indice-invertido]].

## Pipeline de indexado

1. **Recolección** de documentos
2. **Tokenización**: separar en tokens
3. **Normalización**: minúsculas, puntuación
4. **Stemming**: reducir a raíz
5. **Stopword removal**: eliminar palabras comunes
6. **Construcción del índice**

Ver [[preprocesado-texto]] para detalles.

## Queries booleanas

AND: intersección de posting lists — O(x+y) con merge algorithm
OR: unión de posting lists
NOT: complemento

**Optimización AND**: procesar términos en orden de frecuencia ascendente (menor posting list primero).

## Phrase queries

Necesitan [[indice-invertido]] con información posicional. Para "A B": buscar documentos donde A está en posición p y B en posición p+1.

## Indexado distribuido: MapReduce

Para indexar la web (colecciones enormes):
- **Map** (Parsers): leer documentos → emitir pares (término, docID)
- **Shuffle & Sort**: agrupar por término
- **Reduce** (Inversores): para cada término, crear su posting list

**Dos formas de dividir el índice**:
- **Por término** (vertical): cada nodo maneja un subconjunto de términos
- **Por documento** (horizontal): cada nodo maneja un subconjunto de documentos, todos tienen el vocabulario completo

## Ranked retrieval: TF-IDF y modelo vectorial

Ver [[tfidf]] para fórmulas completas.

Resumen:
- Score(q,d) = Σ tf-idf(t,d) para t en query ∩ documento
- Similitud coseno normaliza por longitud de documento
- **¿Por qué normalizar por longitud?** Para que documentos más largos no tengan ventaja injusta

## Modelo probabilístico: BIM y BM25

Ver [[bim]] para fórmulas y ejemplos.

## Métricas de evaluación

Ver [[metricas-ri]] para todas las fórmulas.

Resumen rápido:
- **Precision**: ¿qué fracción de lo que devuelvo es buena?
- **Recall**: ¿qué fracción de lo bueno devuelvo?
- **P@k**: precisión en los primeros k
- **MAP**: promedio de precisiones en posiciones con relevante, promediado sobre queries
- **MRR**: 1/posición del primer relevante, promediado sobre queries
- **NDCG**: ganancia acumulada con descuento por posición, normalizada

## Clasificación de textos

Ver [[clasificacion-textos]].

## Link analysis: PageRank y HITS

Ver [[pagerank]].

## Crawling

El crawling es el proceso de descubrir y descargar páginas web para indexarlas.

**Características necesarias de un crawler**:
- **Robusto**: no caer en spider traps (bucles infinitos de links dinámicos)
- **Educado**: respetar robots.txt y no sobrecargar servidores
- **Distribuido**: ejecutarse en múltiples máquinas
- **Escalable**: añadir máquinas para aumentar el rango
- **Eficiente**: aprovechar recursos de red
- **Prioritario**: indexar páginas de mayor calidad primero
- **Continuo**: re-indexar páginas ya vistas

**Problemas**: spam, spider traps, páginas dinámicas, contenido duplicado/mirrors.

## Preguntas probables de examen

1. AND query con 3 términos: proceso de intersección + optimización con merge algorithm
2. Índice invertido + phrase query: construir el índice posicional y resolver la query
3. PageRank: dada la estructura del grafo y el factor de teleportación, calcular fila de la matriz P
4. Métricas: dado ranking con relevantes, calcular MRR, P@k, MAP, NDCG
5. ¿Por qué normalizar por longitud? → Para igualar documentos largos y cortos
6. Efecto de eliminar stopword → Afecta principalmente al tamaño de las posting lists
7. Tipo test: IDF más alto → término más raro; TF-IDF más alto → frecuente en pocos docs
