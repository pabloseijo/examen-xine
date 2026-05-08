---
title: Teorema CAP
type: concepto
fuentes: [NoSQL3.Consistencia.pdf]
última_actualización: 2026-05-07
---

# Teorema CAP

## Enunciado
En un sistema distribuido, es imposible garantizar simultáneamente las tres propiedades:
- **C** — Consistency (Consistencia)
- **A** — Availability (Disponibilidad)
- **P** — Partition tolerance (Tolerancia a particiones)

## Definiciones exactas (importante para el examen)

**Consistency**: todos los nodos ven los mismos datos al mismo tiempo. Cualquier lectura devuelve la escritura más reciente.

**Availability** (definición del slide, en rojo): *"Cada petición recibida por un nodo que no falla DEBE ser respondida."*

**Partition tolerance**: el sistema sigue funcionando aunque haya una partición de red (grupos de nodos que no pueden comunicarse entre sí).

## Las tres combinaciones

| Tipo | Descripción | Implicación |
|------|-------------|-------------|
| **CA** | Consistente y disponible, sin tolerancia a particiones | Solo posible en sistema centralizado (un nodo). En cluster real, siempre hay riesgo de partición. |
| **CP** | Consistente y tolerante a particiones | Ante partición, algunos nodos se vuelven inaccesibles para mantener consistencia. |
| **AP** | Disponible y tolerante a particiones | Ante partición, todos los nodos responden pero pueden dar datos obsoletos. |

## Conclusión práctica
En un cluster real, siempre hay riesgo de partición de red → P es obligatorio → solo podemos elegir entre CP y AP.

Los sistemas CA solo son posibles con un único servidor (sin distribución).

## Ejemplos
- **CA**: PostgreSQL, MySQL (sin clustering)
- **CP**: HBase, Zookeeper, MongoDB (configuración por defecto)
- **AP**: Cassandra, CouchDB, Amazon Dynamo, Riak

## Relajar consistencia
El teorema CAP dice que ante una partición hay que sacrificar C o A. En ausencia de partición se pueden tener ambas. Muchos sistemas NoSQL relajan la consistencia incluso sin partición para mejorar latencia y throughput ([[consistencia-eventual]], [[acid-vs-base]]).

## Ver también
[[consistencia-eventual]], [[acid-vs-base]], [[03-nosql-consistencia]], [[replicacion]]
