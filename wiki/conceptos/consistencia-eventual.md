---
title: Consistencia Eventual
type: concepto
fuentes: [NoSQL3.Consistencia.pdf]
última_actualización: 2026-05-07
---

# Consistencia Eventual (Eventual Consistency)

## Definición
Un sistema es **eventualmente consistente** si, en ausencia de nuevas actualizaciones, todos los nodos convergerán al mismo valor con el tiempo.

Es el modelo de consistencia más relajado en el espectro ACID-BASE.

## Propiedades BASE
La consistencia eventual forma parte del modelo BASE:
- **B**asically Available: el sistema garantiza disponibilidad (según [[teorema-cap]])
- **S**oft State: el estado puede cambiar con el tiempo incluso sin nuevas escrituras (por propagación asíncrona)
- **E**ventual Consistency: el sistema llegará a ser consistente cuando deje de recibir actualizaciones

## Ventana de inconsistencia
El período durante el cual nodos distintos pueden tener valores distintos. Depende de:
- Velocidad de red
- Carga del sistema
- Frecuencia de replicación

## Cuándo es aceptable
- **Contadores de visitas** en un blog: no importa si dos lectores ven valores ligeramente distintos
- **Redes sociales**: un "like" que tarda 1 segundo en propagarse es aceptable
- **Carros de compra** (Amazon Dynamo): siempre se permite añadir, se consolida al checkout

## Cuándo NO es aceptable
- **Cuentas bancarias**: transferencias deben ser atómicas y consistentes
- **Reservas de asiento en avión**: no puede haber overbooking
- **Precios en bolsa**: datos de inversión requieren consistencia fuerte

## Relación con quórums
Con [[replicacion]] peer-to-peer y quórums (W, R, N):
- Si `R + W <= N`: consistencia eventual (no hay garantía de leer la escritura más reciente)
- Si `R + W > N`: consistencia más fuerte (siempre se lee al menos un nodo que tuvo la escritura más reciente)

## Resolución de conflictos
Con consistencia eventual pueden surgir conflictos escritura-escritura que hay que resolver:
- Last Write Wins (timestamp)
- [[vector-stamp]]: detectar conflictos
- Merge específico del dominio (Amazon: unir carros)

## Ver también
[[teorema-cap]], [[acid-vs-base]], [[replicacion]], [[03-nosql-consistencia]]
