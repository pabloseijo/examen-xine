---
title: NoSQL — Modelado
type: tema
fuentes: [NoSQL1.Modelado.pdf]
última_actualización: 2026-05-07
---

# NoSQL — Modelado

## Motivación: por qué NoSQL

### Impedance mismatch
El modelo relacional trabaja con tablas y filas. Las aplicaciones modernas trabajan con objetos, grafos, documentos jerárquicos. Traducir entre ambos mundos (ORM) es costoso, frágil y genera código boilerplate.

### Cluster computing
Las BD relacionales no escalan bien horizontalmente. Están diseñadas para un solo nodo con garantías ACID fuertes. Internet a escala requiere distribuir datos en cientos de nodos. Las BD NoSQL nacen diseñadas para el cluster desde el inicio.

---

## Los cuatro modelos de datos NoSQL

### 1. Clave-Valor
- Estructura más simple: diccionario gigante distribuido
- Clave: string único. Valor: blob opaco (la BD no lo interpreta)
- Operaciones: GET(key), PUT(key, value), DELETE(key)
- No hay consultas por contenido del valor
- Ejemplos: Redis, Amazon DynamoDB, Riak
- Caso de uso: sesiones de usuario, caché, configuración

### 2. Documentos
- El valor es un documento estructurado (JSON, BSON, XML)
- La BD puede indexar y consultar campos dentro del documento
- Documentos anidados y arrays de forma nativa
- Sin esquema fijo: cada documento puede tener campos diferentes
- Ejemplos: MongoDB, CouchDB, Couchbase
- Caso de uso: catálogos de productos, perfiles de usuario, contenido

### 3. Familias de columnas (Column-Family)
- Organiza datos en familias de columnas (column families)
- Cada fila puede tener columnas distintas dentro de la misma familia
- Diseñado para cargas de trabajo analíticas (leer muchas filas de pocas columnas)
- Eficiente para agregaciones sobre subconjuntos de columnas
- Ejemplos: Apache Cassandra, HBase, Google Bigtable
- Caso de uso: series temporales, logs, analítica

### 4. Grafos
- Nodos y relaciones (aristas) como ciudadanos de primera clase
- Las relaciones tienen tipo y propiedades propias
- Óptimo para consultas de traversal y conectividad
- Típicamente se ejecutan en un solo servidor (difícil distribuir grafos)
- Ejemplos: Neo4J, Amazon Neptune, JanusGraph
- Caso de uso: redes sociales, recomendaciones, fraud detection, knowledge graphs

---

## El concepto de Agregado

### Definición
Un **agregado** es una colección de objetos relacionados que tratamos como una unidad para:
- Manipulación de datos (lectura/escritura atómica)
- Distribución en el cluster (el agregado vive en un solo nodo)

### Relación con DDD (Domain-Driven Design)
El concepto viene de DDD: los agregados son la unidad de consistencia del dominio. En NoSQL, además son la unidad de distribución.

### Qué modelos usan agregados
- **Clave-valor**, **documentos** y **column-family** son "aggregate-oriented"
- El modelo de **grafos** NO usa el concepto de agregado (relaciones cruzadas entre entidades)

### Implicaciones del diseño por agregados
1. Las operaciones son atómicas dentro del agregado, no entre agregados
2. No hay transacciones entre agregados (en general)
3. Los datos que se acceden juntos deben modelarse juntos (desnormalización intencional)
4. Las relaciones entre agregados se gestionan con referencias (como claves foráneas, pero sin JOIN)

---

## Bases de datos sin esquema (Schema-less)

### En BD relacionales
- El esquema se declara antes de insertar datos (`CREATE TABLE`)
- Cambiar el esquema es costoso: `ALTER TABLE` en tablas grandes puede tardar horas
- Migraciones de esquema requieren planificación y downtime

### En BD NoSQL
- No hay esquema declarado en la BD (o es mínimo/opcional)
- El **esquema está en la aplicación** — la aplicación sabe qué campos esperar
- Cambios de esquema: simplemente empieza a escribir documentos con la nueva estructura
- **Migración incremental**: los documentos viejos tienen formato antiguo, los nuevos tienen el nuevo formato. La aplicación maneja ambos.

### Ventajas
- Mayor flexibilidad para iterar rápido
- No hay downtime por migraciones
- Polimorfismo natural: documentos de la misma colección pueden tener estructura diferente

### Desventajas
- La integridad de datos recae en la aplicación
- Más difícil hacer consultas ad-hoc sobre datos sin estructura predecible
- Herramientas de análisis y reporting son menos maduras

---

## Modelado según patrón de acceso

### Principio fundamental
En NoSQL el diseño del modelo de datos depende de **cómo se van a leer los datos**, no solo de cómo están relacionados lógicamente.

### Estrategia 1: Datos embebidos (embedded)
Poner todos los datos relacionados dentro de un único documento.

```json
{
  "_id": "orden-123",
  "cliente": {
    "nombre": "Ana García",
    "email": "ana@example.com"
  },
  "items": [
    {"producto": "Laptop", "precio": 999, "cantidad": 1},
    {"producto": "Ratón", "precio": 25, "cantidad": 2}
  ],
  "total": 1049
}
```

**Ventajas**: Una sola lectura devuelve todo. Atómico. Sin JOINs.
**Desventajas**: Duplicación de datos. Si el cliente cambia de nombre hay que actualizar todas sus órdenes.

### Estrategia 2: Referencias entre agregados
Guardar el ID del documento relacionado, como una clave foránea.

```json
{
  "_id": "orden-123",
  "cliente_id": "cliente-456",
  "items": [...]
}
```

**Ventajas**: Sin duplicación. Una sola copia del cliente.
**Desventajas**: Necesita dos lecturas (una para la orden, otra para el cliente). Sin garantía de integridad referencial.

### Cuándo embeber vs. referenciar
| Embeber | Referenciar |
|---------|-------------|
| Datos siempre accedidos juntos | Datos accedidos independientemente |
| Relación 1-a-pocos | Relación 1-a-muchos o M-a-N |
| El subdocumento no crece sin límite | Subdocumento puede crecer mucho |
| La consistencia es crítica | Duplicación es aceptable |

### Column-families para lecturas eficientes
En modelos de column-family, agrupar las columnas que se leen juntas en la misma familia permite lecturas muy eficientes (solo se leen las columnas de la familia necesaria). Cada familia de columnas se almacena separada físicamente.

---

## Resumen comparativo de modelos

| Característica | Clave-Valor | Documentos | Col-Family | Grafos |
|----------------|-------------|------------|------------|--------|
| Estructura | Par simple | Jerárquica | Tabular extendida | Nodos+aristas |
| Consultas | Solo por clave | Por campos | Por filas/columnas | Traversals |
| Agregados | Sí | Sí | Sí | No |
| Distribución | Fácil | Fácil | Fácil | Difícil |
| Relaciones complejas | No | Limitado | Limitado | Nativo |

Ver también: [[nosql]], [[agregados]], [[mongodb]], [[neo4j]]
