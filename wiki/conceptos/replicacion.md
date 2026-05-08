---
title: Replicación
type: concepto
fuentes: [NoSQL2.Distribucion.pdf, IntroMongoDB.pdf]
última_actualización: 2026-05-07
---

# Replicación

## Definición
Técnica de distribución en la que los **mismos datos se copian** en múltiples nodos. Cada nodo tiene una copia (réplica) de los datos. Es complementaria (ortogonal) al [[sharding]].

## Para qué sirve
- **Alta disponibilidad**: si un nodo falla, otros tienen los datos
- **Escalado de lecturas**: múltiples nodos pueden servir lecturas en paralelo
- **Durabilidad**: más copias → menos riesgo de pérdida de datos

## Dos modelos de replicación

### Maestro-Esclavo (Primary-Secondary)
- **Un nodo primario**: acepta todas las escrituras
- **N nodos secundarios**: replican desde el primario; pueden servir lecturas

Ventajas:
- Sin conflictos escritura-escritura (solo el primario escribe)
- Escalado de lecturas hacia secundarios

Desventajas:
- Cuello de botella en escrituras (el primario es el único punto de escritura)
- Inconsistencia de lectura si se lee de secundario antes de sincronizar
- Conflictos R-W son transitorios (se resuelven solos cuando el secundario se ponga al día)

### Peer-to-Peer (P2P)
- Todos los nodos son iguales: todos pueden leer y escribir
- Sin cuello de botella en escrituras
- Desventaja crítica: **conflictos escritura-escritura** que son permanentes y requieren resolución

| Tipo de conflicto | Maestro-Esclavo | Peer-to-Peer |
|-------------------|-----------------|--------------|
| Escritura-Lectura | Transitorio | Transitorio |
| Escritura-Escritura | No existe | **Permanente — grave** |

## Quórums W, R, N
Con replicación P2P:
- N = factor de replicación (número de copias)
- W = quórum de escritura (nodos que deben confirmar escritura)
- R = quórum de lectura (nodos que deben responder lectura)

Garantías:
- `W > N/2`: solo puede haber una escritura con mayoría (evita conflictos)
- `R + W > N`: siempre se lee un nodo que tuvo la escritura más reciente (consistente)

## Durabilidad de replicación
Antes de confirmar al cliente:
- Confirmar tras 1 nodo: mayor rendimiento, menor durabilidad
- Confirmar tras N nodos: mayor durabilidad, peor rendimiento/disponibilidad

## En MongoDB: Replica Set
- 1 Primary + N Secondaries + opcionales Arbiters
- Replicación asíncrona
- Recuperación automática con heartbeat y elección
- Por defecto: leer del primario (`readPreference: "primary"`)

## Ver también
[[sharding]], [[mongodb]], [[teorema-cap]], [[consistencia-eventual]], [[03-nosql-consistencia]]
