---
title: ACID vs. BASE
type: concepto
fuentes: [NoSQL3.Consistencia.pdf]
última_actualización: 2026-05-07
---

# ACID vs. BASE

## ACID (Bases de datos relacionales)
| Propiedad | Descripción |
|-----------|-------------|
| **Atomicity** | La transacción se completa entera o se deshace (todo o nada) |
| **Consistency** | La BD pasa de un estado válido a otro estado válido |
| **Isolation** | Transacciones concurrentes no se interfieren entre sí |
| **Durability** | Una vez confirmada, la transacción persiste aunque haya fallos |

## BASE (NoSQL)
| Propiedad | Descripción |
|-----------|-------------|
| **Basically Available** | El sistema garantiza disponibilidad según [[teorema-cap]] |
| **Soft State** | El estado puede cambiar con el tiempo incluso sin escrituras (propagación asíncrona) |
| **Eventual Consistency** | El sistema llegará a ser consistente si deja de recibir actualizaciones ([[consistencia-eventual]]) |

## Comparación

| Aspecto | ACID | BASE |
|---------|------|------|
| Consistencia | Fuerte, inmediata | Eventual |
| Disponibilidad | Puede degradarse | Priorizada |
| Partición de red | Difícil de manejar | Tolerada |
| Latencia | Mayor (coordinación) | Menor |
| Throughput | Limitado | Alto |
| Complejidad | En el SGBD | En la aplicación |
| Casos de uso | Banca, reservas | Redes sociales, analytics |

## No es binario
El espectro ACID-BASE es continuo. Se puede elegir el nivel de consistencia apropiado:
- Algunas operaciones de NoSQL pueden ser ACID (transacciones multi-documento en MongoDB)
- Algunas BD relacionales permiten relajar el aislamiento (READ UNCOMMITTED)

## Relajar durabilidad en NoSQL
- Confirmar escritura tras 1 réplica: rápido pero menos durable
- Confirmar escritura tras W réplicas (quórum): más durable, más lento
- La fórmula `R + W > N` garantiza consistencia en quórum

## Ver también
[[teorema-cap]], [[consistencia-eventual]], [[replicacion]], [[03-nosql-consistencia]]
