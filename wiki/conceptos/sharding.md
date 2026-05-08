---
title: Sharding (Particionado)
type: concepto
fuentes: [NoSQL2.Distribucion.pdf, IntroMongoDB.pdf]
última_actualización: 2026-05-07
---

# Sharding (Particionado horizontal)

## Definición
Técnica de distribución de datos en la que los datos se **dividen** entre múltiples nodos. Cada nodo almacena un subconjunto diferente de los datos (en contraposición a [[replicacion]], donde cada nodo tiene una copia de todos los datos).

## Qué mejora
- **Capacidad de almacenamiento**: más shards = más espacio total
- **Throughput de lectura Y escritura**: las operaciones se distribuyen entre shards
- **Localidad de datos**: datos relacionados en el mismo shard → lecturas eficientes

## Qué NO mejora
- **Disponibilidad**: si un shard falla, los datos de ese shard quedan inaccesibles. Para disponibilidad hay que combinar sharding con [[replicacion]].

## Conceptos clave

### Shard Key (Clave de particionado)
- Campo (o campos) que determina en qué shard va cada dato
- Debe estar indexada
- Tres criterios de elección:
  - **Cardinalidad**: cuántos valores distintos tiene → determina el número máximo de shards
  - **Frecuencia**: ¿los valores están bien distribuidos? → hotspots si está sesgada
  - **Monotonicidad**: ¿siempre crece? → todos los datos nuevos van al mismo shard

### Chunks
- Los datos se dividen en chunks (rangos de valores de la shard key)
- El balanceador automático redistribuye chunks entre shards
- Tamaño por defecto en MongoDB: 128MB

### Tipos de sharding en MongoDB
| Tipo | Shard Key | Ventaja | Desventaja |
|------|-----------|---------|------------|
| **Hashed** | `{campo: "hashed"}` | Distribución uniforme | Malo para consultas de rango |
| **Ranged** | `{campo: 1}` | Bueno para rangos | Riesgo de hotspot con clave monotónica |

## Arquitectura MongoDB con sharding
```
Cliente → mongos (router) → Config Servers (metadatos)
                         → Shard1 (replica set)
                         → Shard2 (replica set)
                         → Shard3 (replica set)
```

## Zonas
Asignan rangos de shard key a shards específicos. Útiles para:
- Localidad geográfica (datos europeos en servidores europeos)
- Cumplimiento normativo (GDPR)
- Tiered storage

## Gestión automática
MongoDB gestiona el sharding automáticamente:
- El balanceador mueve chunks entre shards para mantener equilibrio
- Los mongos routers redirigen las consultas al shard correcto sin que el cliente lo sepa

## Ver también
[[replicacion]], [[mongodb]], [[escalado-horizontal]], [[05-mongodb]], [[07-replicacion-sharding-mongodb]]
