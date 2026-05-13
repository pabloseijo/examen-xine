## Modelo Vectorial

La idea: representar cada documento y cada query como un **vector** en un espacio donde cada dimensión es un término del vocabulario. El peso de cada dimensión es su tf-idf.

Doc1 con términos {gato, perro, casa}:

```
Doc1 = (tf-idf(gato,Doc1), tf-idf(perro,Doc1), tf-idf(casa,Doc1))
     = (1.48, 0, 2.1)
```

La query se representa igual — como un vector de tf-idf de sus términos.

Para saber qué documento es más relevante para una query, medimos qué vector está más "cerca" de la query en ese espacio.

### Similitud del Coseno

No usamos distancia euclídea sino el **ángulo** entre los vectores. Cuanto menor el ángulo, más similares.

```
sim(d, q) = (d · q) / (||d|| × ||q||)
```

- **d · q** = producto punto = suma de productos de cada dimensión común
- **||d||** = norma del vector documento = √(Σ xᵢ²)
- **||q||** = norma del vector query

El resultado está entre 0 y 1. Cuanto más cerca de 1, más relevante el documento.

![[modelo-vectorial.png]]




---

## Normalización por longitud — por qué

Sin la división por las normas, un documento largo tendría ventaja injusta: más palabras = más ocurrencias de cualquier término = mayor producto punto, aunque no sea más relevante.

Al dividir por ||d||, conviertes el vector en unitario — su longitud pasa a ser 1. Así dos documentos con el mismo contenido pero distinta longitud tienen el mismo vector normalizado y la misma similitud con cualquier query.

**Cómo responderlo en el examen:**

> Sin normalización, los documentos más largos tienen una ventaja injusta porque acumulan más ocurrencias de cualquier término. La normalización L2 divide cada vector por su norma, haciendo que todos los vectores sean unitarios. Así la similitud mide únicamente la dirección del vector — el contenido — y no su magnitud — la longitud del documento.

---


## BIM — Binary Independence Model

Modelo probabilístico — en lugar de puntuar documentos por tf-idf, estima la **probabilidad de que un documento sea relevante** dado que contiene ciertos términos.

Dos simplificaciones:

- Los documentos se representan como vectores **binarios** (1 si el término está, 0 si no — ignora cuántas veces aparece)
- Los términos son **independientes** entre sí (asunción incorrecta pero funciona bien en práctica)

### Las dos probabilidades clave

- **pᵢ** = probabilidad de que el término i esté en un documento **relevante**
- **rᵢ** = probabilidad de que el término i esté en un documento **irrelevante**

Para estimarlas necesitas saber, de un conjunto de documentos con relevancia conocida:

|Variable|Significado|
|---|---|
|S|total de documentos relevantes|
|s|relevantes que contienen el término|
|N|total de documentos en la colección|
|df|total de documentos que contienen el término|

```
pᵢ = s / S
rᵢ = (df - s) / (N - S)
```

**Por qué tiene sentido**: pᵢ es "de todos los relevantes, ¿qué fracción tiene este término?". rᵢ es "de todos los irrelevantes, ¿qué fracción tiene este término?".

### Ejemplo numérico (tipo examen)

|Término|s (rel. con término)|S (total rel.)|df (docs con término)|N (total docs)|
|---|---|---|---|---|
|Term1|10|20|15|100|
|Term2|5|20|50|100|

- p₁ = 10/20 = **0.5** — la mitad de los relevantes tienen Term1
- r₁ = (15-10)/(100-20) = 5/80 = **0.0625**
- p₂ = 5/20 = **0.25**
- r₂ = (50-5)/(100-20) = 45/80 = **0.5625**

Fíjate en Term2: pᵢ < rᵢ — aparece más en irrelevantes que en relevantes. Ese término no ayuda a discriminar.

---

## BM25

Nace para corregir las tres limitaciones de BIM:

|Limitación de BIM|Solución en BM25|
|---|---|
|Ignora cuántas veces aparece el término|TF normalizada con **saturación**: más ocurrencias ayudan, pero con rendimientos decrecientes|
|Ignora la longitud del documento|Normalización por longitud con parámetro **b**|
|IDF básico|IDF más sofisticado|

Dos parámetros ajustables:

- **k1**: controla cuánto satura la frecuencia del término (típicamente 1.2–2.0)
- **b**: controla cuánto penaliza la longitud del documento (entre 0 y 1)

**Saturación** es el concepto clave: en BIM un término que aparece 100 veces vale igual que uno que aparece 1 vez (solo 0 o 1). En BM25 aparece más veces ayuda, pero a partir de cierto punto ya no mejora mucho el score — la curva se aplana.

---

**Cómo responder BM25 en el examen** — es solo conceptual:

> BM25 mejora BIM en tres aspectos: tiene en cuenta la frecuencia del término mediante una TF normalizada con saturación (más ocurrencias ayudan pero con rendimientos decrecientes), ajusta según la longitud del documento para no penalizar documentos largos injustamente, y usa un IDF más sofisticado. Los parámetros k1 y b permiten ajustar ambos comportamientos.