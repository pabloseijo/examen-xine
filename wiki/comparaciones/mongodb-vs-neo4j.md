---
title: MongoDB vs. Neo4J
type: comparacion
fuentes: [NoSQL1.Modelado.pdf, IntroMongoDB.pdf, IntroNeo4J.pdf]
última_actualización: 2026-05-07
---

# MongoDB vs. Neo4J

## Tabla comparativa principal

| Aspecto | MongoDB | Neo4J |
|---------|---------|-------|
| **Modelo de datos** | Documentos BSON (JSON) | Grafo (nodos + relaciones) |
| **Unidad de almacenamiento** | Documento (agregado) | Nodo / Relación |
| **Relaciones** | Referencias o embedding | Primera clase (aristas con propiedades) |
| **Consultas** | MQL / Aggregation Pipeline | Cypher (lenguaje declarativo de patrones) |
| **JOINs / Traversals** | `$lookup` (costoso si frecuente) | Traversal nativo, coste constante por salto |
| **Escalado horizontal** | Excelente (sharding nativo) | Limitado (difícil partir un grafo) |
| **Transacciones** | ACID multi-documento (desde 4.0) | ACID completo |
| **Esquema** | Schema-less, flexible | Semi-flexible (etiquetas y propiedades libres) |
| **Disponibilidad** | Alta (Replica Set + sharding) | Media-alta (Enterprise: Causal Cluster) |
| **Indexado** | Índices en campos de documento | Índices en propiedades de nodo/relación |
| **Madurez** | Alta, muy adoptado | Alta en su nicho (grafos) |
| **Casos de uso típicos** | Catálogos, CMS, IoT, logs | Redes sociales, fraude, recomendación, KG |

## Cuándo usar MongoDB

- Los datos tienen estructura variable o jerárquica
- El acceso principal es por clave o por atributo del documento
- Se necesita escalado horizontal masivo
- Las relaciones son pocas o se pueden embeber
- Alto volumen de escrituras distribuidas

## Cuándo usar Neo4J

- Las relaciones **son** los datos: navegación multi-hop, caminos más cortos, comunidades
- Consultas de conectividad: "¿quién conoce a quién a distancia 3?"
- Datos altamente interconectados donde `$lookup` múltiple sería ineficiente
- Grafos de conocimiento, detección de fraude, recomendación basada en grafo
- El volumen de datos cabe en un servidor (o cluster pequeño)

## Modelo de datos comparado

### MongoDB — documento de película
```json
{
  "_id": ObjectId("..."),
  "titulo": "El Laberinto del Fauno",
  "anio": 2006,
  "directores": ["Guillermo del Toro"],
  "actores": [
    {"nombre": "Ivana Baquero", "personaje": "Ofelia"},
    {"nombre": "Sergi López", "personaje": "Vidal"}
  ],
  "generos": ["Fantasia", "Drama"]
}
```

### Neo4J — el mismo grafo
```cypher
(p:Pelicula {titulo: "El Laberinto del Fauno", anio: 2006})
(d:Persona {nombre: "Guillermo del Toro"})-[:DIRIGE]->(p)
(a:Persona {nombre: "Ivana Baquero"})-[:ACTUA_EN {personaje:"Ofelia"}]->(p)
(a)-[:PERTENECE]->(g:Genero {nombre: "Fantasia"})
```

En MongoDB las relaciones se almacenan **dentro** del documento (embedding) o como referencias (array de ids). En Neo4J son entidades propias con tipo y propiedades.

## Consultas comparadas

### "Películas en las que actúa Penélope Cruz"

**MongoDB (MQL):**
```js
db.peliculas.find({ "cast": "Penélope Cruz" })
```

**Neo4J (Cypher):**
```cypher
MATCH (p:Persona {nombre: "Penélope Cruz"})-[:ACTUA_EN]->(peli:Pelicula)
RETURN peli.titulo
```

### "Actores que trabajaron con los mismos directores que Penélope Cruz" (2 saltos)

**MongoDB:** requiere dos `$lookup` o lógica en aplicación — costoso.

**Neo4J:**
```cypher
MATCH (penelope:Persona {nombre:"Penélope Cruz"})-[:ACTUA_EN]->(p)<-[:DIRIGE]-(d)
MATCH (d)-[:DIRIGE]->(p2)<-[:ACTUA_EN]-(actor)
WHERE actor <> penelope
RETURN DISTINCT actor.nombre
```

La consulta de 2 saltos en Neo4J tiene coste similar a la de 1 salto. En MongoDB el coste crece exponencialmente con cada `$lookup`.

## Escalado: diferencia clave

| | MongoDB | Neo4J |
|-|---------|-------|
| Sharding | Sí, nativo | No (difícil por naturaleza del grafo) |
| Replicación | Replica Set automático | Enterprise: Causal Cluster |
| Límite práctico | Petabytes distribuidos | Billones de nodos/relaciones en 1 servidor |
| Cuello de botella | Shard key mal elegida | Tamaño del grafo en memoria |

## Resumen de elección

```
¿Los datos son principalmente jerárquicos/documentales?
    → MongoDB

¿Las relaciones entre entidades son la consulta principal?
    → Neo4J

¿Necesitas escalar horizontalmente a gran escala?
    → MongoDB

¿Necesitas atravesar grafos profundos eficientemente?
    → Neo4J

¿Combinación de ambos?
    → Polyglot persistence: MongoDB + Neo4J
```

## Ver también
[[mongodb]], [[neo4j]], [[grafos]], [[modelado-documentos]], [[nosql-vs-sql]], [[05-mongodb]], [[06-neo4j]]
