---
title: ACID vs. BASE (Comparación)
type: comparacion
fuentes: [NoSQL3.Consistencia.pdf]
última_actualización: 2026-05-07
---

# ACID vs. BASE — Comparación detallada

## Tabla comparativa principal

| Propiedad | ACID | BASE |
|-----------|------|------|
| **Consistencia** | Fuerte e inmediata | Eventual (llegará a ser consistente) |
| **Disponibilidad** | Puede reducirse para mantener consistencia | Prioridad máxima |
| **Partición de red** | Muy difícil de manejar | Tolerada por diseño |
| **Latencia** | Mayor (coordinación entre nodos) | Menor (respuesta rápida sin coordinación) |
| **Throughput** | Limitado por el coordinador | Alto (sin bloqueos globales) |
| **Complejidad** | En el SGBD (transparente para el dev) | En la aplicación (el dev gestiona conflictos) |
| **Estado del sistema** | Siempre válido tras cada transacción | Puede ser inconsistente transitoriamente |
| **Rollback** | Sí, automático | No nativo; compensación manual |
| **Aislamiento** | Sí (niveles: READ COMMITTED, SERIALIZABLE…) | No garantizado entre operaciones |

## ACID en detalle

| Letra | Propiedad | Descripción |
|-------|-----------|-------------|
| **A** | Atomicity (Atomicidad) | La transacción se completa **entera** o se deshace (todo o nada) |
| **C** | Consistency (Consistencia) | La BD pasa de un estado **válido** a otro estado **válido** (constraints, FK, triggers) |
| **I** | Isolation (Aislamiento) | Transacciones concurrentes no se interfieren entre sí |
| **D** | Durability (Durabilidad) | Una vez confirmada (COMMIT), la transacción persiste aunque haya fallo de hardware |

## BASE en detalle

| Acrónimo | Propiedad | Descripción |
|----------|-----------|-------------|
| **BA** | Basically Available | El sistema garantiza disponibilidad según [[teorema-cap]]; puede haber datos desactualizados |
| **S** | Soft State | El estado puede cambiar con el tiempo **incluso sin escrituras nuevas** (propagación asíncrona de réplicas) |
| **E** | Eventual Consistency | Si el sistema deja de recibir actualizaciones, **eventualmente** todos los nodos convergerán al mismo valor |

## El espectro ACID-BASE no es binario

```
ACID ←————————————————————→ BASE
Banca  Reservas  E-commerce  Analytics  Redes sociales
```

- Algunas operaciones NoSQL pueden ser ACID:
  - MongoDB 4.0+: **transacciones multi-documento** dentro de un replica set
  - MongoDB 4.2+: transacciones multi-shard
  - CockroachDB / Spanner: NewSQL con ACID distribuido

- Algunas BD relacionales permiten relajar el aislamiento:
  - `SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED`
  - Dirty reads para mayor rendimiento

## Quórums: el puente entre ACID y BASE

Los sistemas NoSQL usan quórums para graduar la consistencia:

| Configuración | R | W | N | Consistencia | Latencia |
|---------------|---|---|---|--------------|----------|
| Alta disponibilidad | 1 | 1 | 3 | Muy baja (stale reads) | Muy baja |
| Quórum balanceado | 2 | 2 | 3 | Alta (R+W>N) | Media |
| Consistencia máxima | 3 | 3 | 3 | Fuerte | Alta |

Reglas clave:
- `W > N/2` → No hay conflictos de escritura (majority write)
- `R + W > N` → Siempre se lee al menos una réplica con la escritura más reciente

## Casos de uso

### ACID es imprescindible
- **Banca y finanzas**: debitar cuenta A y acreditar cuenta B deben ser atómicos
- **Reservas**: una plaza de avión no puede venderse dos veces
- **Registros médicos**: integridad de datos crítica
- **Inventario**: stock no puede ser negativo

### BASE es suficiente (y más eficiente)
- **Redes sociales**: un "like" que llega 500ms tarde es aceptable
- **Analytics y métricas**: datos aproximados son útiles
- **Carritos de compra**: el carrito puede estar "stale" unos segundos
- **DNS**: el sistema de nombres más usado del mundo es eventual
- **Caché**: Redis por definición es BASE

## Relación con el Teorema CAP

BASE es consecuencia directa de elegir **AP** en el triángulo CAP:
- Se prioriza Availability (A) y Partition Tolerance (P)
- Se sacrifica Consistency (C) → se obtiene Eventual Consistency

ACID corresponde a sistemas **CA** (sin partición) o **CP** (con partición pero sacrificando disponibilidad).

→ Ver [[teorema-cap]], [[consistencia-eventual]]

## Ver también
[[acid-vs-base]], [[teorema-cap]], [[consistencia-eventual]], [[replicacion]], [[03-nosql-consistencia]]
