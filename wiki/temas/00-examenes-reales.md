---
title: Exámenes Reales 2024 y 2025
type: tema
fuentes: [examenes.pdf]
última_actualización: 2026-05-07
---

# Exámenes Reales — XINE

> **DOCUMENTO MÁS IMPORTANTE.** Exámenes reales de 2024 y 2025 con preguntas exactas. Los patrones se repiten casi literalmente entre años.

---

## EXAMEN 2025 JUNIO

### Parte Recuperación de Información *("muy similar al del 2024")*

1. **AND query con 3 términos**: `sistema AND operativo AND Windows`
   - Explicar el proceso para hacer la consulta (intersección de posting lists, merge algorithm)
   - Explicar cómo **optimizar** la query (Merge algorithm, explicación sin código)
   - → Ver [[indice-invertido]] y [[queries-booleanas]]

2. **Índice invertido + phrase query** "to be" y "be to"
   - Dado el índice posicional:
     - `to`: doc1: [1,19,22]; doc2: [1,39,42];...
     - `be`: doc1: [2,21,25]; doc2: [1,40,41];...
   - Decir qué números aparecen en cada posting list y dar el resultado de ambas phrase queries
   - → Ver [[indice-invertido]]

3. **PageRank**: dado un grafo + probabilidad de teleportación = 5%
   - Calcular el valor concreto para una fila de la matriz P
   - → Ver [[pagerank]]

4. **Métricas de evaluación**: dados 2 queries con sus resultados
   - Calcular **MRR** y **P@5** del sistema
   - Calcular **MAP** (número de relevantes totales por query > número recuperados)
   - Calcular **NDCG@3** para una de las queries (con valores de relevancia 1, 3, 2, etc.)
   - *Se pueden expresar ecuaciones con valores numéricos sin resolverlas*
   - → Ver [[metricas-ri]]

5. **Posting list + stopword**: de una posting list, eliminar una stopword del diccionario
   - Eliminarla de la posting list y decir en qué nivel de tamaño afecta más
   - → Ver [[indice-invertido]]

6. **Normalización por longitud**: explicar el **porqué** de aplicarla
   - → Ver [[modelo-vectorial]]

7. **Tipo test**: 2 carillas de preguntas de respuesta única, 4 opciones (una siempre "ninguna de las anteriores")
   - Preguntas prácticas con pequeños documentos (3-5 palabras): término con mayor IDF, valor TF/IDF mayor, etc.
   - Preguntas teóricas: definición de HITS, etc.

### Parte Bases de Datos NoSQL *(1 carilla y media)*

1. **Esquema**: ventajas y desventajas
   - En BBDD relacionales (Nuevo y Legacy): almacenar cambios y scripts de migración, mantener soporte
   - En NoSQL: esquema a nivel de aplicación, punto de acceso a datos, limitar acceso a partes del agregado, la aplicación es la encargada de los cambios, datos que no se actualizan, caso de grafos
   - → Ver [[nosql]]

2. **Teorema CAP**: explicar el teorema
   - Caso de sistema centralizado (única máquina)
   - Caso de sistemas distribuidos
   - Compromiso entre **latencia y consistencia** a nivel real
   - → Ver [[teorema-cap]]

3. **Comandos/códigos de las prácticas** *(los códigos no son los mismos pero siguen esta forma)*:

   **a.** Arrancar un mongod como parte de un replica set de shard:
   ```
   mongod --replSet rsShard1 --port 27018 --dbpath /home/alumnogreibd/shard1 --bind_ip localhost
   ```

   **b.** Inicializar el Config Server:
   ```javascript
   rs.initiate({
     _id: "rsConfServer",
     configsvr: true,
     members: [{_id: 0, host: "localhost:27017"}]
   })
   ```

   **c.** Preferencia de lectura en MongoDB:
   ```javascript
   db.getMongo().setReadPref('primaryPreferred')
   ```

   **d.** Comando Cypher Neo4J (MERGE):
   ```cypher
   MERGE (m:Movie { title:"Cloud Atlas" })
   ON CREATE SET m.released = 2012
   RETURN m;
   ```

---

## EXAMEN 2024 JUNIO

### Parte Recuperación de Información

8. **AND query**: `sistema AND operativo AND Windows` — explicar el proceso (igual que 2025 pero sin la parte de optimización)

9. **Índice invertido + phrase query** "to be" y "be to":
   - `to`: doc1: [1,19,22]; doc2: [1,39,42];...
   - `be`: doc1: [2]; doc2: [20]; doc5: [30,50,23]
   - *(Misma pregunta que 2025, posting lists ligeramente distintas)*

10. **PageRank**: grafo + 5% teleportación — igual que 2025

11. **Métricas**: MRR, P@5, MAP, NDCG — dados 2 queries con resultados

12. **Posting list + stopword**: mismo enunciado que 2025

### Parte Bases de Datos NoSQL *(1 carilla, 3 preguntas)*

4. **Acceso a datos en NoSQL**: por qué hay que tener en cuenta cómo se van a acceder los datos cuando se hace el modelado → [[modelado-documentos]]

5. **Replicación**: maestro-esclavo y peer-to-peer → [[replicacion]]

6. **Problema write-write**: estrategias optimistas y pesimistas, almacenamiento de datos distribuidos y soluciones → [[consistencia-eventual]]

---

## Patrones que se repiten

| Pregunta | 2024 | 2025 |
|----------|------|------|
| AND query 3 términos | ✓ | ✓ + optimización |
| Índice invertido + phrase query (mismas posting lists) | ✓ | ✓ |
| PageRank con 5% teleportación | ✓ | ✓ |
| MRR + MAP + NDCG | ✓ | ✓ + P@5 |
| Posting list + stopword | ✓ | ✓ |
| Normalización por longitud | — | ✓ |
| Tipo test RI | — | ✓ (2 carillas) |
| Teorema CAP | — | ✓ |
| Esquema NoSQL vs relacional | — | ✓ |
| Comandos prácticas (MongoDB + Neo4J) | — | ✓ |
| Replicación | ✓ | — |
| Write-write / consistencia | ✓ | — |
| Acceso a datos al modelar | ✓ | — |

**Conclusión**: La parte RI es casi idéntica cada año. La parte NoSQL varía más pero rota entre: CAP, esquema, replicación, consistencia, modelado, comandos.

---

## Lo que hay que saber resolver sí o sí

### RI — Ejercicios mecánicos
1. Construir índice invertido (con y sin posición) a partir de un corpus
2. Resolver AND query: intersección de posting lists paso a paso
3. Phrase query: verificar posiciones consecutivas
4. Calcular TF-IDF de términos en documentos
5. Calcular similitud coseno entre query y documento
6. Calcular MRR, P@k, MAP, NDCG@k dado un ranking
7. Calcular PageRank: construir matriz P = (1-d)·(1/N) + d·M

### NoSQL — Conceptos con desarrollo
1. Teorema CAP: qué es, sistema centralizado vs. distribuido, compromiso real latencia/consistencia
2. Esquema: diferencias relacional (scripts de migración) vs. NoSQL (schema-on-read, app encargada)
3. Comandos MongoDB: rs.initiate, mongos, sh.addShard, setReadPref
4. Comandos Neo4J: MERGE, ON CREATE SET, MATCH...WHERE...RETURN
