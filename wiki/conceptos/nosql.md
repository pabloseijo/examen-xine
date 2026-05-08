---
title: NoSQL
type: concepto
fuentes: [NoSQL1.Modelado.pdf, NoSQL2.Distribucion.pdf, NoSQL3.Consistencia.pdf]
última_actualización: 2026-05-07
---

# NoSQL

## Definición
NoSQL ("Not Only SQL") es un término paraguas para bases de datos que no usan el modelo relacional tabular tradicional. Diseñadas para escalar horizontalmente en clusters de servidores commodity.

## Por qué NoSQL existe
1. **[[impedance-mismatch]]**: las aplicaciones modernas trabajan con objetos y grafos; el modelo relacional obliga a traducir constantemente
2. **Escala horizontal**: internet a escala requiere distribuir en cientos de nodos; los RDBMS clásicos no están diseñados para eso
3. **Flexibilidad de esquema**: iterar rápido en el modelo de datos sin costosas migraciones de esquema
4. **Casos de uso específicos**: grafos, documentos anidados, series temporales... cada modelo NoSQL es óptimo para su dominio

## Los cuatro modelos
| Modelo | Ejemplos | Caso de uso |
|--------|----------|-------------|
| Clave-Valor | Redis, DynamoDB | Caché, sesiones |
| Documentos | [[mongodb]], CouchDB | Catálogos, perfiles |
| Familias de columnas | Cassandra, HBase | Analytics, logs |
| Grafos | [[neo4j]], Neptune | Redes sociales, recomendaciones |

## Características comunes
- Escalado horizontal (no vertical)
- Modelos de consistencia relajados ([[consistencia-eventual]], [[acid-vs-base]])
- [[teorema-cap]]: ante partición de red, elegir entre C y A
- Sin esquema fijo (o esquema flexible)
- Diseñados para [[sharding]] y [[replicacion]]

## Ver también
[[01-nosql-modelado]], [[02-nosql-distribucion]], [[03-nosql-consistencia]], [[agregados]]
