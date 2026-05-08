---
title: Bases de Datos Objeto-Relacionales
type: concepto
fuentes: [BasesDatosObjetoRelacionales.pdf]
última_actualización: 2026-05-07
---

# Bases de Datos Objeto-Relacionales (BDOR)

## Qué son
Extensión del modelo relacional con características de orientación a objetos, manteniendo compatibilidad con SQL. Estándar SQL:1999 (SQL3) y SQL:2003.

## Características principales

### Tipos estructurados (CREATE TYPE)
```sql
CREATE TYPE empleado AS (
    nombre VARCHAR(50),
    dir    direccion,
    salario_base DECIMAL(9,2)
) INSTANTIABLE NOT FINAL
INSTANCE METHOD salario() RETURNS DECIMAL(9,2);
```

### Tablas con tipo (CREATE TABLE OF)
```sql
CREATE TABLE empleados OF empleado (
    REF IS oid SYSTEM GENERATED,
    dep WITH OPTIONS SCOPE departamentos
);
```

### Referencias y derreferenciación
```sql
-- Acceder a través de referencia con ->
SELECT e.nombre, e.dep->nombre
FROM empleados e
WHERE e.dep->dir.loc = 'Santiago';
```

### Herencia
```sql
CREATE TYPE directivo UNDER empleado (
    presupuesto DECIMAL(12,2)
) NOT FINAL;

CREATE TABLE directivos OF directivo UNDER empleados;
```

### Filtrar por tipo
```sql
-- Solo filas que son exactamente 'empleado'
SELECT * FROM ONLY(empleados);
-- Equivalente con IS OF
SELECT * FROM empleados WHERE DEREF(oid) IS OF (empleado);
```

## Constructores de tipo SQL
| Constructor | Descripción | Ejemplo |
|-------------|-------------|---------|
| `ROW` | Tipo compuesto anónimo | `(calle, num) AS dir` |
| `ARRAY` | Array de tipo base | `paga NUMERIC(8,2)[12]` |
| `MULTISET` | Conjunto sin orden | `SET(VARCHAR(50))` |

## Herencia entre tablas
- `CREATE TABLE sub OF tipo UNDER tabla_padre`
- SELECT en subtabla hace join automático con supertabla
- `ONLY(tabla)` para excluir subtipos
- INSERT en subtabla inserta también en supertabla
- DELETE desde supertabla borra en cascada hacia subtablas

## Ver también
[[04-bd-objeto-relacionales]], [[08-agregados-postgresql]], [[nosql-vs-sql]]
