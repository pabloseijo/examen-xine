---
title: Agregados en PostgreSQL
type: tema
fuentes: [Xestión de Información non Estruturada [G4012453] [2025_2026]_ Agregados en PostgreSQL _ Campus Virtual.pdf]
última_actualización: 2026-05-07
---

# Agregados en PostgreSQL

## Objetivo

Demostrar cómo PostgreSQL, siendo un SGBD relacional, soporta **tipos de datos compuestos y arrays** que permiten almacenar datos no en primera forma normal (datos agregados/anidados), haciendo de puente entre el mundo relacional y el NoSQL.

---

## Tipos de dato ARRAY en PostgreSQL

PostgreSQL soporta arrays de cualquier tipo base, con cualquier número de dimensiones.

### Declaración
```sql
-- Dos sintaxis equivalentes (la segunda es estándar SQL para 1D)
CREATE TABLE empleado (
    id           INT PRIMARY KEY,
    nombre       VARCHAR(100),
    paga_mensual NUMERIC(8,2)[12],          -- array 1D de 12 elementos
    paga_mensual_trienio NUMERIC(8,2)[3][12] -- array 2D de 3x12
);

-- Equivalente con ARRAY (estándar SQL, solo 1D)
paga_mensual NUMERIC(8,2) ARRAY[12]
```

**Nota**: los índices de dimensión (el `[12]`) son solo documentación — PostgreSQL no los verifica.

### Inserción
```sql
-- Con llaves (sintaxis de string)
INSERT INTO empleado VALUES (
    1, 'Miguel Ángel Pintor', 'Via del Corso', '14', '23452', 'Roma',
    '{23000,23500,23800,23800,23500,22900,23200,46000,23100,23700,23500,46100}',
    ARRAY[
        ARRAY[23900, 12534, 22354, 43233, 12324, 12324, 12134, 21313, 23245, 13346, 23523, 23246],
        ARRAY[21233, 12135, 23234, 23342, 21342, 23412, 12341, 12352, 21345, 12355, 34521, 52445]
    ]
);
```

### Consultas sobre arrays
```sql
-- Elemento por índice (índices comienzan en 1)
SELECT id, nombre, paga_mensual[7] FROM empleado;

-- Elemento de array 2D [fila][columna]
SELECT id, nombre, paga_mensual_trienio[2][3] FROM empleado;

-- Rango (slice)
SELECT id, nombre, paga_mensual[4:8] FROM empleado;

-- Filtro sobre elemento del array
SELECT id, nombre, paga_mensual
FROM empleado
WHERE paga_mensual[8] > 13423;
```

### UNNEST — convertir array en tabla
```sql
-- Cada elemento del array se convierte en una fila
SELECT id, nombre, paga
FROM empleado, unnest(paga_mensual) AS pagas(paga);

-- Con filtro y agregación
SELECT id, nombre, sum(paga) AS paga_anual
FROM empleado, unnest(paga_mensual) AS pagas(paga)
WHERE paga > 234
GROUP BY id, nombre
HAVING sum(paga) > 123223;

-- WITH ORDINALITY: obtener el índice de cada elemento
SELECT id, nombre, paga, mes
FROM empleado,
     unnest(paga_mensual) WITH ORDINALITY AS pagas(paga, mes)
WHERE id = 1
ORDER BY mes DESC;

-- Array 2D con UNNEST y cálculo de año/mes desde posición lineal
SELECT id, nombre, paga,
       (mestotal-1)/12+1 AS ano,
       (mestotal-1)%12+1 AS mes
FROM empleado,
     unnest(paga_mensual_trienio) WITH ORDINALITY AS pagas(paga, mestotal)
WHERE id = 1
ORDER BY ano, mes;
```

### Generar arrays
```sql
-- ARRAY desde subconsulta
SELECT id, nombre,
       ARRAY(SELECT nombre FROM empleado e2
             WHERE e1.localidad = e2.localidad
             ORDER BY id) AS mismalocalidad
FROM empleado e1
WHERE id = 1;

-- array_agg: función de agregado que construye un array
SELECT localidad, array_agg(nombre) AS empleados
FROM empleado
GROUP BY localidad;

-- Con orden dentro del array
SELECT localidad, array_agg(nombre ORDER BY id) AS pagas
FROM empleado
GROUP BY localidad;
```

---

## Tipos de dato compuestos (ROW / CREATE TYPE)

### Tipo ROW en consultas
```sql
-- Se puede construir un ROW en la cláusula SELECT
SELECT id, nombre,
       (calle, numero, cp, localidad) AS direccion,
       paga_mensual
FROM empleado;
```

### Limitación: columnas de tabla no pueden ser ROW sin CREATE TYPE
```sql
-- ESTO FALLA: no existe tipo para direccion
CREATE TABLE empleado2 AS
SELECT id, nombre, (calle, numero, cp, localidad) AS direccion, paga_mensual
FROM empleado;
```

### Solución: CREATE TYPE
```sql
DROP TYPE IF EXISTS tipo_direccion;
CREATE TYPE tipo_direccion AS (
    calle     VARCHAR(255),
    numero    VARCHAR(5),
    cp        VARCHAR(5),
    localidad VARCHAR(100)
);

CREATE TABLE empleado2 (
    id           INT PRIMARY KEY,
    nombre       VARCHAR(100),
    direccion    tipo_direccion,       -- columna de tipo compuesto
    paga_mensual NUMERIC(8,2)[12]
);

-- Insertar con CAST
INSERT INTO empleado2
SELECT id, nombre,
       CAST((calle, numero, cp, localidad) AS tipo_direccion),
       paga_mensual
FROM empleado;
```

### Array de tipos compuestos
```sql
-- Un array cuyos elementos son objetos compuestos
SELECT localidad,
       array_agg((calle, numero, cp, localidad)) AS direcciones
FROM empleado
GROUP BY localidad;
```

---

## Ejemplo completo: Tormentas

### Modelo de datos
Tablas originales:
- `tormentas(temporada, nombre, inicio, fin, viento_max, viento_medio)`
- `regiones(nombre_region, pais, ...)`
- `regiones_afectadas(temporada, nombre_tormenta, pais, region)`

### Crear tipo compuesto para países
```sql
CREATE TYPE pais AS (
    nombre   VARCHAR(100),
    regiones VARCHAR(100) ARRAY
);
```

### Crear tabla desnormalizada (agregada)
```sql
DROP TABLE IF EXISTS tormentas1;
CREATE TABLE tormentas1 (
    temporada    INT,
    nombre       VARCHAR(100),
    inicio       TIMESTAMP,
    fin          TIMESTAMP,
    viento_max   FLOAT,
    viento_medio FLOAT,
    paises       pais ARRAY    -- array de objetos compuestos
);
```

### Insertar datos de forma agregada (de tormentas + regiones_afectadas)
```sql
INSERT INTO tormentas1
SELECT *,
    ARRAY(
        SELECT CAST((ra.pais, array_agg(ra.region)) AS pais)
        FROM regiones_afectadas ra
        WHERE t.temporada = ra.temporada
          AND t.nombre = ra.nombre
        GROUP BY ra.pais
    ) AS paises
FROM tormentas t;
```

### Consultas sobre la tabla agregada vs. versión normalizada

**a. Países afectados por KATRINA 2005**
```sql
-- Con tabla agregada (más simple)
SELECT pais
FROM tormentas1 t,
     unnest(t.paises) AS pais(pais, regiones)
WHERE nombre = 'KATRINA' AND temporada = 2005;

-- Con tablas normalizadas (más verboso)
SELECT DISTINCT ra.pais
FROM tormentas t, regiones_afectadas ra
WHERE t.nombre = ra.nombre AND t.temporada = ra.temporada
  AND t.temporada = 2005 AND t.nombre = 'KATRINA';
```

**b. Pares (país, región) afectados por KATRINA 2005**
```sql
-- Con tabla agregada
SELECT pais, region
FROM tormentas1 t,
     unnest(t.paises) AS pais(pais, regiones),
     unnest(pais.regiones) AS region(region)
WHERE nombre = 'KATRINA' AND temporada = 2005;

-- Con tablas normalizadas
SELECT ra.pais, ra.region
FROM tormentas t, regiones_afectadas ra
WHERE t.nombre = ra.nombre AND t.temporada = ra.temporada
  AND t.temporada = 2005 AND t.nombre = 'KATRINA';
```

**c. Tormentas que afectaron a Florida, United States**
```sql
-- Opción 1: unnest en FROM
SELECT t.*
FROM tormentas1 t,
     unnest(t.paises) AS pais(pais, regiones),
     unnest(pais.regiones) AS region(region)
WHERE pais = 'United States' AND region = 'Florida';

-- Opción 2: subconsulta en WHERE con IN
SELECT *
FROM tormentas1
WHERE ('United States', 'Florida') IN (
    SELECT pais, region
    FROM unnest(paises) AS pais(pais, regiones),
         unnest(regiones) AS region(region)
);
```

---

## Gestión de datos JSON en PostgreSQL

PostgreSQL tiene soporte nativo para JSON como tipo de columna.

### Operadores JSON
| Operador | Descripción | Resultado |
|----------|-------------|-----------|
| `->` | Acceder a campo (devuelve JSON) | `generos->'name'` → `"Action"` |
| `->>` | Acceder a campo (devuelve texto) | `generos->>'name'` → `Action` |
| `json_array_elements(col)` | Expandir array JSON a filas | Una fila por elemento |

### Ejemplo: empresas productoras
```sql
SELECT empresa->>'name' AS productora
FROM peliculas,
     json_array_elements(empresas_produccion) AS e(empresa)
WHERE presupuesto = (SELECT MAX(presupuesto) FROM peliculas);
```

### Ejemplo: reparto de película más cara
```sql
SELECT ereparto->>'name' AS actriz,
       ereparto->>'character' AS personaje
FROM peliculas p, creditos c,
     json_array_elements(c.reparto) AS r(ereparto)
WHERE p.id = c.pelicula
  AND p.presupuesto = (SELECT MAX(presupuesto) FROM peliculas)
ORDER BY CAST(ereparto->>'order' AS INTEGER);
```

### Ejemplo: películas de Tarantino
```sql
SELECT p.titulo, p.fecha_emision, p.presupuesto
FROM peliculas p, creditos c,
     json_array_elements(c.personal) AS personal(empleado)
WHERE p.id = c.pelicula
  AND empleado->>'job' = 'Director'
  AND empleado->>'name' = 'Quentin Tarantino'
ORDER BY p.fecha_emision;
```

### Ejemplo: array de actores con json_agg
```sql
SELECT p.titulo,
       json_agg(
           json_build_object('nombre', ereparto->>'name', 'personaje', ereparto->>'character')
           ORDER BY CAST(ereparto->>'order' AS INTEGER)
       ) AS reparto
FROM peliculas p, creditos c,
     json_array_elements(c.reparto) AS reparto(ereparto)
WHERE p.id = c.pelicula
GROUP BY p.titulo, p.presupuesto
ORDER BY p.presupuesto DESC
LIMIT 10;
```

Ver también: [[bd-objeto-relacionales]], [[agregados]], [[04-bd-objeto-relacionales]]
