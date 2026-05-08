---
title: Agregados
type: concepto
fuentes: [NoSQL1.Modelado.pdf]
última_actualización: 2026-05-07
---

# Agregados

## Definición
Un **agregado** es una colección de objetos relacionados que se tratan como una unidad para:
1. **Manipulación de datos**: lectura y escritura atómica de todo el conjunto
2. **Distribución**: el agregado es la unidad que reside en un único nodo del cluster

Concepto tomado del **Domain-Driven Design (DDD)**: los agregados son la unidad de consistencia del dominio.

## En NoSQL
Los modelos de datos clave-valor, documentos y familias de columnas son "aggregate-oriented":
- Cada par clave-valor es un agregado
- Cada documento JSON en MongoDB es un agregado
- Cada fila en Cassandra (con sus column families) es un agregado

El modelo de grafos NO usa agregados (relaciones cruzadas entre todas las entidades).

## Implicaciones de diseño

### Atomicidad
Las operaciones dentro de un agregado son atómicas. Las operaciones **entre** agregados no lo son (sin transacciones multi-agregado).

### Modelado
Decidir qué datos van dentro del mismo agregado es la decisión de diseño más importante:
- **Datos que se acceden juntos → mismo agregado** (embedded)
- **Datos que se acceden independientemente → agregados separados + referencias**

### Distribución
El aggregado vive en un único nodo. Las consultas que acceden a un solo agregado son eficientes. Las consultas que acceden a múltiples agregados pueden requerir scatter-gather (query a todos los shards).

## Diferencia con modelo relacional
En relacional, los datos se normalizan: cada entidad en su tabla, relaciones con JOINs. En modelos de agregados, se desnormaliza intencionalmente para que los datos accedidos juntos estén juntos.

## Ejemplo en PostgreSQL
PostgreSQL permite almacenar datos no en 1FN usando:
- Arrays: `paga_mensual NUMERIC(8,2)[12]`
- Tipos compuestos: `direccion tipo_direccion`
- JSON: `generos JSON`

## Ver también
[[nosql]], [[modelado-documentos]], [[mongodb]], [[01-nosql-modelado]], [[08-agregados-postgresql]]
