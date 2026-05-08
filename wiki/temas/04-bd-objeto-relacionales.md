---
title: Bases de Datos Objeto-Relacionales
type: tema
fuentes: [BasesDatosObjetoRelacionales.pdf]
última_actualización: 2026-05-07
---

# Bases de Datos Objeto-Relacionales

## Objetivo y motivación

Las BD objeto-relacionales (BDOR) **extienden el modelo relacional** con características de orientación a objetos, manteniendo compatibilidad con SQL estándar.

**Por qué**: el modelo relacional es potente pero limitado en expresividad para tipos de datos complejos. Los OO permiten encapsulación, herencia y tipos compuestos. El estándar SQL:1999 (SQL3) y SQL:2003 incorporan estas extensiones.

**Generaciones de SGBD**:
1. Pre-relacional (jerárquico, red)
2. Relacional (SQL, 1970s-80s)
3. **Objeto-relacional** (finales de 1990s) ← estamos aquí

---

## Tipos de dato incorporados en SQL

Tipos básicos ya existentes en SQL:
- `BOOLEAN`, `CHAR(n)`, `VARCHAR(n)`, `TEXT`
- `BLOB` (Binary Large Object), `CLOB` (Character Large Object)
- `DATE`, `TIME`, `TIMESTAMP`, `INTERVAL`
- Numéricos: `SMALLINT`, `INTEGER`, `BIGINT`, `NUMERIC(p,s)`, `REAL`, `DOUBLE PRECISION`

---

## Constructores de tipo SQL:1999

### ROW
Tipo compuesto anónimo con campos nombrados:
```sql
SELECT id, (calle, numero, localidad) AS direccion
FROM empleado
```
No se puede usar directamente como columna de tabla sin un `CREATE TYPE`.

### ARRAY
Array de un tipo base (unidimensional en estándar SQL, multidimensional en PostgreSQL):
```sql
CREATE TABLE empleado (
    id INT PRIMARY KEY,
    nombre VARCHAR(100),
    paga_mensual NUMERIC(8,2) ARRAY[12],   -- 12 pagas
    paga_mensual_trienio NUMERIC(8,2)[3][12]  -- 3 trienios x 12 meses
);
```
- Acceso por índice: `paga_mensual[7]` (7ª paga)
- Acceso a rango: `paga_mensual[4:8]` (pagas 4 a 8)
- Filtro: `WHERE paga_mensual[8] > 13423`
- Unnest: `FROM empleado, unnest(paga_mensual) AS pagas(paga)`
- `UNNEST WITH ORDINALITY`: genera columna adicional con el índice del array
- `array_agg(expr)`: función de agregado que construye un array
- `ARRAY(subquery)`: genera un array a partir de una subconsulta

### MULTISET
Conjunto de valores sin orden, con duplicados permitidos (menos común en PostgreSQL).

---

## Tipos estructurados (CREATE TYPE)

### Definición básica
```sql
CREATE TYPE direccion AS (
    calle VARCHAR(100),
    num   INTEGER,
    loc   VARCHAR(10)
) NOT FINAL;
```

- `NOT FINAL`: indica que puede haber subtipos (permite herencia)
- `INSTANTIABLE NOT FINAL`: puede tener instancias directas Y subtipos

### Tipo con métodos
```sql
CREATE TYPE empleado AS (
    nombre       VARCHAR(50),
    dir          direccion,
    salario_base DECIMAL(9,2),
    complementos DECIMAL(9,2)
)
INSTANTIABLE NOT FINAL
INSTANCE METHOD salario() RETURNS DECIMAL(9,2);
```

### Tipos de métodos
- **INSTANCE METHOD**: opera sobre una instancia (accede a atributos con `SELF`)
- **STATIC METHOD**: método de clase (no necesita instancia)
- **CONSTRUCTOR METHOD**: crea una nueva instancia del tipo

### Observers y mutators
Para cada atributo, el sistema genera automáticamente:
- **Observer** (getter): `nombre()` → devuelve el valor de `nombre`
- **Mutator** (setter): `nombre(valor)` → establece el valor de `nombre`

### Implementación del método
```sql
CREATE INSTANCE METHOD salario() 
RETURNS DECIMAL(9,2)
FOR empleado
BEGIN
    RETURN SELF.salario_base + SELF.complementos;
END;
```

---

## Tablas con tipo (Typed Tables)

### CREATE TABLE OF
```sql
CREATE TABLE empleados OF empleado (
    REF IS oid SYSTEM GENERATED,
    dep WITH OPTIONS SCOPE departamentos
);
```

- `REF IS oid`: nombre de la columna que actúa como OID (Object Identifier)
- `SYSTEM GENERATED`: el sistema genera el OID automáticamente
- `USER GENERATED`: el usuario proporciona el OID
- `DERIVED`: el OID se deriva de los atributos del tipo

### Tipos de REF IS
| Tipo | Descripción |
|------|-------------|
| `SYSTEM GENERATED` | OID generado automáticamente por el sistema (opaco, no significativo) |
| `USER GENERATED` | El usuario proporciona el OID al insertar |
| `DERIVED` | El OID se calcula a partir de los valores de los atributos (OID funcional) |

---

## Referencias (REF y DEREF)

### Declarar una referencia en un tipo
```sql
CREATE TYPE empleado AS (
    nombre VARCHAR(50),
    dep    REF(departamento)   -- referencia a un departamento
) ...;
```

### Acotar el scope (tabla de destino)
```sql
CREATE TABLE empleados OF empleado (
    REF IS oid SYSTEM GENERATED,
    dep WITH OPTIONS SCOPE departamentos  -- dep solo puede referenciar filas de 'departamentos'
);
```

### Acceder a través de la referencia
Usando el operador `->` (equivalente a DEREF + acceso de campo):
```sql
SELECT e.nombre, e.dep->nombre
FROM Empleados e
WHERE e.dep->dir.loc = 'Santiago';
```

Usando `DEREF()` explícito:
```sql
SELECT e.nombre, DEREF(e.dep).nombre
FROM Empleados e;
```

**Importante**: No se puede hacer `e.dep.nombre` directamente. Hay que usar `->` o `DEREF()` porque `dep` es una referencia, no un valor inlineado.

### Diferencia con JOIN relacional
Las referencias en BDOR son más expresivas: no requieren JOIN explícito, pero siguen siendo "punteros" lógicos (no garantizan integridad referencial por defecto).

---

## Herencia de tipos

### Definición de jerarquía
```sql
CREATE TYPE persona AS (
    nombre VARCHAR(100),
    edad   INTEGER
) NOT FINAL;

CREATE TYPE empleado UNDER persona (
    salario DECIMAL(9,2)
) NOT FINAL;

CREATE TYPE directivo UNDER empleado (
    presupuesto DECIMAL(12,2)
) INSTANTIABLE NOT FINAL;
```

### Características de herencia
- `UNDER supertipo`: el subtipo hereda todos los atributos y métodos del supertipo
- Un subtipo puede **redefinir métodos** del supertipo: `OVERRIDING METHOD`
- `NOT INSTANTIABLE`: tipo abstracto, no se pueden crear instancias directas

### Sobreescribir métodos
```sql
CREATE INSTANCE METHOD salario()
RETURNS DECIMAL(9,2)
FOR directivo
OVERRIDING METHOD salario() ...
BEGIN
    -- implementación específica para directivo
END;
```

### Filtrar por tipo en consultas
```sql
-- Solo filas que son exactamente de tipo 'empleado' (no subtipos)
SELECT * FROM Empleados WHERE DEREF(oid) IS OF (empleado);

-- Usando IS OF con lista
SELECT * FROM Empleados WHERE DEREF(oid) IS OF (empleado, directivo);
```

---

## Herencia entre tablas

### Definición
```sql
CREATE TABLE personas OF persona (
    REF IS oid SYSTEM GENERATED
);

CREATE TABLE empleados OF empleado
UNDER personas;   -- empleados hereda de personas

CREATE TABLE directivos OF directivo
UNDER empleados;
```

### Comportamiento de herencia entre tablas

**SELECT en subtabla**: hace un JOIN automático con la supertabla para devolver todos los atributos.
```sql
SELECT * FROM empleados;
-- Devuelve: nombre, edad (de personas) + salario (de empleados)
```

**ONLY**: excluir filas de subtipos.
```sql
SELECT * FROM ONLY(personas);
-- Solo filas que son exactamente 'persona', no empleados ni directivos
```

**INSERT en subtabla**: inserta automáticamente en la supertabla también.
```sql
INSERT INTO empleados VALUES ('Ana', 30, 50000);
-- Inserta 'Ana', 30 en personas Y 50000 en empleados
```

**DELETE en cascada**: borrar de supertabla elimina también de subtabla.
```sql
DELETE FROM personas WHERE nombre = 'Ana';
-- Elimina de personas Y de empleados (y directivos si aplica)
```

---

## PostgreSQL: tipos compuestos y arrays

PostgreSQL implementa estas características con sintaxis propia:

```sql
-- Tipo compuesto en PostgreSQL
DROP TYPE IF EXISTS tipo_direccion;
CREATE TYPE tipo_direccion AS (
    calle    VARCHAR(255),
    numero   VARCHAR(5),
    cp       VARCHAR(5),
    localidad VARCHAR(100)
);

CREATE TABLE empleado2 (
    id        INT PRIMARY KEY,
    nombre    VARCHAR(100),
    direccion tipo_direccion,
    paga_mensual NUMERIC(8,2)[12]
);

-- Insertar con CAST
INSERT INTO empleado2
SELECT id, nombre,
       CAST((calle, numero, cp, localidad) AS tipo_direccion),
       paga_mensual
FROM empleado;
```

### JSON en PostgreSQL
PostgreSQL permite almacenar datos no estructurados como JSON:
```sql
CREATE TABLE peliculas (
    id           INT4,
    titulo       TEXT,
    generos      JSON,
    reparto      JSON,
    personal     JSON,
    ...
);

-- Navegar JSON con ->  (devuelve JSON) y ->> (devuelve texto)
SELECT empresa->>'name' AS productora
FROM peliculas, json_array_elements(empresas_produccion) AS e(empresa)
WHERE presupuesto = (SELECT MAX(presupuesto) FROM peliculas);
```

Ver también: [[bd-objeto-relacionales]], [[agregados]], [[nosql-vs-sql]]
