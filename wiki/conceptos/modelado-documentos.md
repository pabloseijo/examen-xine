---
title: Modelado de Documentos
type: concepto
fuentes: [NoSQL1.Modelado.pdf, IntroMongoDB.pdf]
última_actualización: 2026-05-07
---

# Modelado de Documentos

## Principio fundamental
El diseño del modelo de documentos depende de **cómo se van a acceder los datos**, no solo de cómo están relacionados lógicamente (a diferencia del modelo relacional, que normaliza siguiendo reglas formales).

## Decisión principal: Embeber vs. Referenciar

### Embeber (Embedded / Denormalizado)
Poner todos los datos relacionados dentro de un único documento.

```json
{
  "_id": "orden-123",
  "cliente": {"nombre": "Ana", "email": "ana@ex.com"},
  "items": [
    {"producto": "Laptop", "precio": 999}
  ]
}
```

Usar cuando:
- Los datos se acceden siempre juntos
- Relación 1-a-pocos
- Consistencia es crítica (atomicidad del documento)
- El subdocumento no crece sin límite

### Referenciar (Referenced / Normalizado)
Guardar solo el ID del documento relacionado.

```json
{
  "_id": "orden-123",
  "cliente_id": "cliente-456",
  "items": [...]
}
```

Usar cuando:
- Los datos se acceden independientemente
- Relación M-a-N o 1-a-muchos sin límite
- La duplicación de datos es problemática (e.g., el cliente cambia de email)

## Polimorfismo
Los documentos de la misma colección pueden tener estructura diferente (esquema dinámico). Útil para:
- Catálogos de productos con atributos variables
- Versiones del modelo de datos durante migración incremental

## Esquema sin esquema
- No hay `CREATE TABLE` con columnas fijas
- El esquema está en la aplicación (sabe qué campos esperar)
- Validación opcional con JSON Schema (desde MongoDB v3.6)
- Migración incremental: nuevos documentos con nuevo formato, viejos con formato antiguo

## Column-family para lecturas eficientes
En modelos column-family (Cassandra, HBase): agrupar en la misma familia de columnas los datos que se leen juntos. Cada familia se almacena físicamente separada → leer solo las columnas necesarias.

## Ver también
[[agregados]], [[mongodb]], [[nosql]], [[01-nosql-modelado]]
