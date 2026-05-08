## Preprocesado de texto

Pipeline que se aplica tanto a los documentos al indexar **como a las queries al buscar**. El mismo proceso en ambos lados — si al indexar conviertes a minúsculas, al buscar también.

---

### 1. Tokenización

Dividir el texto en tokens (unidades). Parece simple pero hay casos problemáticos:

- **"Nueva York"** — ¿un token o dos? Si los separas, pierdes que es una entidad. Si los unes, necesitas detectar expresiones multipalabra.
- **"don't"** — ¿es "do" + "n't", o "do" + "not", o un solo token?
- **URLs, emails, "C++"** — ninguna regla simple funciona para todo.

La decisión de cómo tokenizar afecta directamente a qué entra en el índice.

---

### 2. Normalización

Convertir tokens a una forma estándar para que "Windows" y "windows" casen con la misma entrada del índice. Incluye:

- Convertir a minúsculas
- Eliminar puntuación
- Expandir contracciones ("don't" → "do not")

**Problema**: no siempre conviene normalizar todo. "Windows" (sistema operativo) y "windows" (ventanas) son cosas distintas. Normalizar a minúsculas puede perder significado en nombres propios.

---

### 3. Stemming

Reducir la palabra a su raíz morfológica para que variantes de la misma palabra casen con la misma entrada del índice.

- "corriendo", "corrió", "correr" → "corr"
- Algoritmo más usado: **Porter stemmer**
- **No siempre produce palabras reales** — el stem es un truncamiento artificial.

**Diferencia con lematización**: la lematización devuelve la forma canónica real del diccionario ("ran" → "run", "mejor" → "bueno"). Es más precisa pero más costosa computacionalmente. El stemming es más rápido pero más agresivo.

---

### 4. Stopword removal

Eliminar palabras muy comunes que aportan poco significado: "el", "la", "de", "y", "a"...

**Efecto en el índice**: las posting lists de stopwords son las más grandes del índice (aparecen en casi todos los documentos). Eliminarlas reduce drásticamente el tamaño total de las postings — mucho más que del diccionario, que solo pierde una entrada por stopword.

**Caso importante**: en phrase queries pueden ser necesarias. "To be or not to be" — si eliminas las stopwords, la frase desaparece del índice y no puedes buscarla. En web retrieval no siempre se eliminan por esta razón.

---

## Índice posicional y phrase queries

El índice invertido básico guarda en qué documentos aparece cada término. Suficiente para AND/OR/NOT, pero **no para phrase queries** ("Nueva York", "to be").

Para phrase queries necesitas un **índice posicional**: además del docID, guardas las posiciones exactas donde aparece el término dentro del documento.

```
to   → [doc1: (1, 19, 22),  doc2: (1, 39, 42)]
be   → [doc1: (2, 21, 25),  doc2: (1, 40, 41)]
```

### Cómo resolver una phrase query

Para "to be" (dos palabras consecutivas):

1. Obtén los docIDs en común (igual que un AND)
2. Para cada doc en común, comprueba si existe posición **p** en `to` tal que **p+1** esté en `be`

**Doc1**: `to` en {1,19,22}, `be` en {2,21,25}

- 1 → 2? ✓
- 19 → 20? no
- 22 → 23? no → Doc1 contiene "to be" en posición 1

**Doc2**: `to` en {1,39,42}, `be` en {1,40,41}

- 1 → 2? no
- 39 → 40? ✓
- 42 → 43? no → Doc2 contiene "to be" en posición 39

Para **"be to"** (orden inverso): buscas posición **p** en `be` tal que **p+1** esté en `to`.

**Doc1**: `be` en {2,21,25}, `to` en {1,19,22}

- 2 → 3? no
- 21 → 22? ✓
- 25 → 26? no → Doc1 contiene "be to" en posición 21

**Doc2**: `be` en {1,40,41}, `to` en {1,39,42}

- 1 → 2? no
- 40 → 41? no
- 41 → 42? ✓ → Doc2 contiene "be to" en posición 41

El orden de los términos en la phrase query importa — "to be" y "be to" dan resultados distintos.

---

## MapReduce para indexado distribuido

Cuando el corpus es enorme no puedes construir el índice en una sola máquina. MapReduce lo distribuye en tres fases:

**Fase Map** — los _parsers_ recorren los documentos y emiten pares `(término, docID)`. Cada máquina procesa un subconjunto de documentos de forma independiente y en paralelo.

**Fase Shuffle** — el framework agrupa automáticamente todos los pares por término: `término → [docID1, docID2, ...]`

**Fase Reduce** — los _inversores_ reciben todos los docIDs de un término y construyen su posting list ordenada: `término → [docID1, docID2, docID3, ...]`

**Particionado**: en Map se reparte por documento (cada parser procesa N documentos); en Reduce se reparte por término (cada inversor es responsable de ciertos términos).

---
Se refiere a cómo se divide el trabajo entre máquinas.

**En la fase Map**: cada máquina (parser) recibe un subconjunto de documentos y los procesa todos. Por ejemplo, si tienes 1 millón de documentos y 10 máquinas, cada una procesa 100.000 documentos. El reparto es por documento.

**En la fase Reduce**: cada máquina (inversor) es responsable de construir las posting lists de un subconjunto de términos. Por ejemplo, una máquina construye las posting lists de los términos de la A a la M, otra de la N a la Z. El reparto es por término.

Es simplemente cómo se divide el trabajo para que todo pueda hacerse en paralelo.

Fase Shuffle — el framework agrupa automáticamente todos los pares por término: término → [docID1, docID2, ...] Fase Reduce — los inversores reciben todos los docIDs de un término y construyen su posting list ordenada: término → [docID1, docID2, docID3, ...] en que se diferencian estos dos

Buena pregunta. La diferencia es:

**Shuffle** — agrupa los pares que salieron del Map, pero sin orden. Simplemente junta todos los docIDs que emitieron los distintos parsers para el mismo término. El resultado puede ser `gato → [3, 1, 5, 2]` — desordenado.

**Reduce** — toma ese grupo y construye la posting list definitiva: ordena por docID y elimina duplicados si los hay. El resultado es `gato → [1, 2, 3, 5]` — ordenada, lista para usar.

El Shuffle agrupa, el Reduce ordena y formaliza.

que es un inversor y un parser

Son simplemente los nombres que se les da a las máquinas según su rol:

**Parser** — la máquina que ejecuta la fase Map. Se llama parser porque su trabajo es leer ("parsear") los documentos y extraer los términos. Emite pares `(término, docID)`.

**Inversor** — la máquina que ejecuta la fase Reduce. Se llama inversor porque su trabajo es construir el índice invertido: toma docIDs agrupados por término y construye la posting list. "Invierte" la relación — en lugar de documento→términos, produce término→documentos.