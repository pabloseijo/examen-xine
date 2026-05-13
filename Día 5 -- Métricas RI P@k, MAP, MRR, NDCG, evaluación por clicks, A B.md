## Métricas de evaluación RI

Estas métricas responden a la pregunta: **¿cómo de bueno es un sistema de RI?** Le das una query, devuelve un ranking, y mides si los documentos relevantes están arriba.

Antes de empezar, un concepto clave que afecta a varias métricas:

- **Relevante recuperado**: documento relevante que el sistema devolvió
- **Total relevantes**: todos los relevantes que existen en la colección, los devuelva o no el sistema

---

## Precision y Recall

```
Precision = relevantes recuperados / total recuperados
Recall    = relevantes recuperados / total relevantes en la colección
```

Son opuestas: si devuelves muchos documentos, el recall sube pero la precision baja.

**Ejemplo**: colección con 5 relevantes, sistema devuelve 4 documentos de los cuales 3 son relevantes:

- Precision = 3/4 = 0.75
- Recall = 3/5 = 0.60

---

## P@k — Precision at k

Precision considerando solo los primeros k resultados. Ignora todo lo que haya después de la posición k.

```
P@k = relevantes en el top k / k
```

**Ejemplo**: ranking = [R, NR, R, NR, R, NR, NR]

- P@1 = 1/1 = 1.0
- P@3 = 2/3 = 0.67
- P@5 = 3/5 = 0.60

---

## MAP — Mean Average Precision

Es la métrica más completa para evaluar un sistema con varias queries. Se calcula en tres pasos:

**Paso 1** — Para una query, calcular P@k en cada posición donde aparece un relevante:

Ranking = [R, NR, R, NR, NR, R], total relevantes en colección = 6

|Posición|Doc|¿Relevante?|P@k en esa posición|
|---|---|---|---|
|1|R|✓|1/1 = 1.0|
|2|NR|—|—|
|3|R|✓|2/3 = 0.67|
|4|NR|—|—|
|5|NR|—|—|
|6|R|✓|3/6 = 0.50|

**Paso 2** — Average Precision (AP) de esa query:

El sistema solo recuperó 3 de 6 relevantes. Los 3 no recuperados cuentan como P=0:

```
AP = (1.0 + 0.67 + 0.50 + 0 + 0 + 0) / 6 = 2.17 / 6 = 0.36
```

**Paso 3** — MAP = promedio de AP sobre todas las queries:

```
MAP = (AP_query1 + AP_query2 + ...) / número de queries
```

**El matiz crítico del examen**: el denominador del AP es el **total de relevantes en la colección**, no los que recuperó el sistema. Los relevantes que no aparecen en el ranking cuentan como cero.

---
## MRR — Mean Reciprocal Rank

Mide únicamente dónde aparece el **primer documento relevante**. No le importa nada más.

```
MRR = (1/Q) × Σ (1/posición del primer relevante)
```

**Ejemplo con 2 queries**:

- Query 1: ranking = [NR, R, NR, NR] → primer relevante en posición 2 → 1/2
- Query 2: ranking = [NR, NR, R, NR] → primer relevante en posición 3 → 1/3

```
MRR = (1/2)(1/2 + 1/3) = (1/2)(0.5 + 0.33) = 0.42
```

Sencillo. Siguiente.

---

## NDCG — Normalized Discounted Cumulative Gain

Es la más compleja pero la más potente: usa **grados de relevancia** (0, 1, 2, 3...) en lugar de solo R/NR, y penaliza que los más relevantes aparezcan tarde.

Se calcula en 3 pasos:

**Paso 1 — DCG@p**: ganancia acumulada con descuento por posición

```
DCG@p = Σ (2^relevancia_i - 1) / log₂(i+1)
```

**Paso 2 — IDCG@p**: el DCG del ranking **ideal** — los mismos documentos pero ordenados de mayor a menor relevancia. Es el máximo posible.

**Paso 3 — NDCG@p**:

```
NDCG@p = DCG@p / IDCG@p
```

Resultado entre 0 y 1. Si el ranking es perfecto, NDCG = 1.

---

**Ejemplo completo** — ranking con relevancias [3, 1, 2], calcular NDCG@3:

**DCG@3**:

|Posición i|Relevancia|(2^rel - 1) / log₂(i+1)|
|---|---|---|
|1|3|(2³-1) / log₂(2) = 7 / 1 = 7.0|
|2|1|(2¹-1) / log₂(3) = 1 / 1.585 = 0.63|
|3|2|(2²-1) / log₂(4) = 3 / 2 = 1.5|

```
DCG@3 = 7.0 + 0.63 + 1.5 = 9.13
```

**IDCG@3** — ranking ideal sería [3, 2, 1]:

|Posición i|Relevancia|(2^rel - 1) / log₂(i+1)|
|---|---|---|
|1|3|7 / 1 = 7.0|
|2|2|3 / 1.585 = 1.89|
|3|1|1 / 2 = 0.5|

```
IDCG@3 = 7.0 + 1.89 + 0.5 = 9.39
```

**NDCG@3**:

```
NDCG@3 = 9.13 / 9.39 = 0.97
```

Casi perfecto — el sistema solo falló en poner el 2 antes que el 1 en posiciones 2 y 3.

---
## Evaluación por clicks

Hasta ahora las métricas asumían que sabemos qué documentos son relevantes. En la práctica, Google no le pregunta a nadie — usa los **clicks** de los usuarios.

Pero clicks ≠ relevancia. Si el primer resultado tiene 1000 clicks y el quinto tiene 10, no es porque el primero sea mejor — es porque **la gente hace click en lo que está arriba** aunque no sea lo mejor. Esto se llama **sesgo posicional**.

Lo que sí indica un click es **preferencia**: el usuario eligió ese documento sobre los que tenía delante en ese momento.

Tres señales útiles:

**1. Número de clicks ajustado por posición**: un documento es bueno si recibe más clicks de los esperados para su posición, no si recibe muchos clicks en términos absolutos.

**2. Tiempo entre clicks**: si el usuario abre un documento y en 2 segundos abre otro, el primero probablemente era malo. Si tarda 3 minutos, probablemente lo leyó y le sirvió. Hay mucho ruido (pausas, distracciones) pero es una señal útil.

**3. Orden de navegación**: si el usuario hace click en el resultado 1, luego en el 3 (saltándose el 2), puedes inferir que 3 > 2. Solo funciona con el inmediatamente anterior — no puedes comparar con los que el usuario ya olvidó.

---

## Interleaved Rankings

Forma de comparar dos sistemas A y B sin preguntarle nada al usuario.

**Proceso**:

1. Tomas el ranking de A y el de B
2. Los mezclas alternando: primero el 1º de A, luego el 1º de B, luego el 2º de A... eliminando duplicados
3. Se lo muestras al usuario como si fuera un solo ranking
4. Cada click puntúa al sistema que puso ese documento en posición más alta

El resultado: sabes cuál de los dos sistemas genera resultados que los usuarios prefieren, sin que el usuario sepa que está participando en un experimento.

Se alterna quién va primero en cada consulta (ABABAB... en una, BABABA... en la siguiente) para evitar sesgos.

---

## A/B Testing

Alternativa al interleaving. En lugar de mezclar los resultados:

- El **grupo A** (mayoría de usuarios) ve el sistema actual
- El **grupo B** (porcentaje pequeño) ve el sistema nuevo

Se mide qué grupo tiene mejores métricas (clicks, tiempo en página, etc.).

**Cuándo usar cada uno**:

||Interleaving|A/B Testing|
|---|---|---|
|Velocidad|Más rápido — necesita menos usuarios|Más lento|
|Qué detecta|Preferencia entre rankings|Cualquier cambio (también interfaz)|
|Limitación|No sirve si el cambio es en la interfaz|Necesita mucho tráfico|

Ambos son **evaluación naturalista** — los usuarios no saben que están siendo evaluados, por lo que se comportan con normalidad.