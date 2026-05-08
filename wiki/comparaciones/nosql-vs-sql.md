---
title: NoSQL vs. SQL
type: comparacion
fuentes: [NoSQL1.Modelado.pdf, NoSQL3.Consistencia.pdf, BasesDatosObjetoRelacionales.pdf]
última_actualización: 2026-05-07
---

# NoSQL vs. SQL (Relacional)

## Tabla comparativa principal

| Aspecto | SQL (Relacional) | NoSQL |
|---------|-----------------|-------|
| **Modelo de datos** | Tablas con filas y columnas | Documentos, clave-valor, grafos, col-family |
| **Esquema** | Rígido, declarado antes de insertar | Flexible (schema-less) o opcional |
| **Consultas** | SQL declarativo, JOINs | Consultas propias de cada BD |
| **Transacciones** | ACID completo | BASE / eventual (o ACID limitado) |
| **Escalado** | Vertical (principalmente) | Horizontal (diseñado para clusters) |
| **Consistencia** | Fuerte por defecto | Variable, típicamente eventual |
| **Madurez** | >50 años, muy madura | Más reciente, menos estándarizada |
| **Integridad referencial** | Nativa (FK, constraints) | Responsabilidad de la aplicación |
| **Joins** | Nativo, eficiente en tablas pequeñas | No existen o son costosos (lookup/denormalización) |
| **Distribución** | Difícil, herramientas limitadas | Diseñado para distribución |

## Cuándo elegir SQL
- Datos estructurados con relaciones complejas y fijas
- Transacciones financieras o médicas (ACID estricto)
- Consultas ad-hoc y reporting complejos
- Equipo y herramientas con experiencia en SQL
- El volumen de datos cabe en un servidor razonablemente potente

## Cuándo elegir NoSQL
- Datos semi-estructurados o con estructura variable
- Escala masiva (millones de usuarios, petabytes de datos)
- Alta disponibilidad como requisito crítico
- Modelo de datos que encaja con un modelo NoSQL específico (grafos → Neo4J, documentos → MongoDB)
- Desarrollo ágil con modelo de datos en evolución

## Impedance Mismatch
El modelo relacional no es natural para programar objetos:
- Los objetos tienen jerarquía, herencia, colecciones anidadas
- Las tablas son planas y rectangulares
- El mapeo objeto-relacional (ORM) añade complejidad y overhead

NoSQL (especialmente documentos) minimiza este mismatch: los documentos JSON se mapean directamente a objetos.

## Extensiones objeto-relacionales (puente SQL-NoSQL)
SQL:1999 añadió características OO al modelo relacional:
- Tipos estructurados (`CREATE TYPE`)
- Herencia (`UNDER`)
- Referencias (`REF`)
- Arrays y multisets
- → Ver [[bd-objeto-relacionales]]

PostgreSQL añade además JSON, arrays multidimensionales → Ver [[08-agregados-postgresql]]

## No es binario
Muchas organizaciones usan **polyglot persistence**: diferentes BD para diferentes necesidades:
- PostgreSQL para datos transaccionales
- MongoDB para catálogos y perfiles
- Redis para caché
- Neo4J para grafos de relaciones

## Ver también
[[nosql]], [[bd-objeto-relacionales]], [[acid-vs-base]], [[teorema-cap]]
