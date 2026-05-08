---
title: Escalado Horizontal
type: concepto
fuentes: [NoSQL2.Distribucion.pdf]
última_actualización: 2026-05-07
---

# Escalado Horizontal (Scale-out)

## Definición
Estrategia de escalado que añade más nodos (servidores) al sistema en lugar de mejorar el hardware de un solo servidor (escalado vertical / scale-up).

## Escalado vertical vs. horizontal
| | Vertical (scale-up) | Horizontal (scale-out) |
|-|---------------------|------------------------|
| Cómo | Mejor CPU, más RAM, disco más rápido | Más servidores |
| Límite | Limitado por hardware máximo disponible | Casi ilimitado |
| Coste | Servidores de alto rendimiento son muy caros | Servidores commodity baratos |
| Disponibilidad | Un solo punto de fallo | Alta disponibilidad distribuida |
| Complejidad | Baja | Alta (coordinación distribuida) |

## NoSQL y escalado horizontal
Las BD NoSQL están diseñadas desde el inicio para el escalado horizontal:
- Datos distribuidos con [[sharding]]
- Alta disponibilidad con [[replicacion]]
- Sin estado compartido entre nodos (shared-nothing architecture)
- Consistencia relajada para permitir operación distribuida ([[consistencia-eventual]])

## Implicaciones
- Los sistemas distribuidos tienen que gestionar [[teorema-cap]]
- El escalado horizontal introduce complejidad: particiones de red, sincronización, etc.
- Los sistemas [[nosql]] aceptan estas complejidades a cambio de escala

## Ver también
[[sharding]], [[replicacion]], [[nosql]], [[02-nosql-distribucion]]
