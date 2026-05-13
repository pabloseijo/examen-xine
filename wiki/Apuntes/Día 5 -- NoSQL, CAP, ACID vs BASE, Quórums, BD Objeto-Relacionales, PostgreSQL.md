---
title: Día 5 — NoSQL, CAP, ACID vs BASE, Quórums, BD Objeto-Relacionales, PostgreSQL
type: tema
fuentes: [NoSQL1.Modelado.pdf, NoSQL2.Distribucion.pdf, NoSQL3.Consistencia.pdf, BasesDatosObjetoRelacionales.pdf]
última_actualización: 2026-05-13
---

# Día 5 — BD: NoSQL, CAP, ACID vs BASE, Quórums, BD Objeto-Relacionales, PostgreSQL

---

## 1. Por qué existe NoSQL

El modelo relacional tiene dos problemas a escala:

**Impedance mismatch**: las aplicaciones trabajan con objetos y grafos, pero las bases de datos relacionales obligan a traducir todo a tablas. Cada vez que lees o escribes tienes que convertir entre los dos mundos.

**Escala horizontal**: internet a escala requiere distribuir datos en cientos de máquinas. Los RDBMS clásicos no están diseñados para eso — escalan verticalmente (máquina más potente), no horizontalmente (más máquinas).

---

## 2. Los 4 modelos NoSQL

| Modelo | Ejemplos | Caso de uso |
|--------|----------|-------------|
| **Clave-Valor** | Redis, DynamoDB | Caché, sesiones de usuario |
| **Documentos** | MongoDB, CouchDB | Catálogos, perfiles, contenido |
| **Familias de columnas** | Cassandra, HBase | Analytics, logs, series temporales |
| **Grafos** | Neo4J, Neptune | Redes sociales, recomendaciones |

Lo que tienen en común:
- Escalan horizontalmente (más máquinas, no máquinas más potentes)
- Esquema flexible — la estructura la controla la aplicación, no la base de datos. Puedes añadir campos nuevos sin alterar todos los documentos existentes.
- Relajan la consistencia — aceptan que durante un período corto distintos nodos puedan tener valores distintos, convergiendo eventualmente al mismo valor.

---

## 3. Agregados

Un **agregado** es la unidad natural de datos que se lee y escribe de forma atómica. Es la unidad de distribución — cada agregado vive en un único nodo del cluster.

**Regla de diseño**: los datos que se acceden juntos van en el mismo agregado (embedding). Los datos que se acceden independientemente van en agregados separados con referencias.

- En MongoDB: cada documento es un agregado
- En clave-valor: cada par clave-valor es un agregado
- En grafos: NO hay agregados — las relaciones cruzan entre entidades libremente

---

## 4. Modelado: Embedding vs. Referenciado

En NoSQL no se diseña según normalización sino según **cómo se van a acceder los datos**.

**Embedding** (incrustar el subdocumento dentro del documento):
```json
{
  "_id": "pedido-123",
  "cliente": { "nombre": "Ana", "email": "ana@ex.com" },
  "total": 999
}
```
Usar cuando:
- Los datos se acceden siempre juntos
- Relación 1-a-pocos
- Necesitas atomicidad (todo en un documento)

**Referenciado** (guardar solo el ID):
```json
{
  "_id": "pedido-123",
  "cliente_id": "cliente-456",
  "total": 999
}
```
Usar cuando:
- Los datos se acceden por separado
- Relación M-a-N o 1-a-muchos sin límite
- Duplicar datos sería problemático (ej. el cliente cambia de email)

---

## 5. Esquema: relacional vs. NoSQL

Cayó en el examen 2025.

**En relacional**: el esquema es rígido. Cambiar la estructura requiere scripts de migración. Hay que mantener dos versiones: el esquema **New** (el nuevo) y el **Legacy** (el que tienen los datos viejos).

**En NoSQL**: no hay esquema fijo en la base de datos — el esquema está en la aplicación (schema-on-read). La app gestiona los cambios. Permite migración incremental — documentos nuevos con formato nuevo y viejos con formato antiguo conviven sin problema.

---

## 6. Teorema CAP

En un sistema distribuido es imposible garantizar las tres propiedades a la vez:

- **C** — Consistency: todos los nodos ven los mismos datos al mismo tiempo
- **A** — Availability: cada petición recibida por un nodo que no falla DEBE ser respondida
- **P** — Partition tolerance: el sistema sigue funcionando aunque haya una partición de red

**Sistema centralizado (una sola máquina)**: puede ser CA — no hay partición de red posible porque no hay red entre nodos.

**Sistema distribuido (cluster)**: siempre hay riesgo de partición de red → P es obligatorio → solo puedes elegir entre:
- **CP**: consistente pero puede volverse inaccesible ante una partición
- **AP**: siempre responde pero puede dar datos obsoletos

**En la práctica**: no es "elige 2 de 3" — es un compromiso continuo entre **consistencia y latencia**. Puedes relajar consistencia incluso sin partición para mejorar velocidad.

| Tipo | Ejemplos |
|------|----------|
| CA | PostgreSQL, MySQL (sin clustering) |
| CP | MongoDB, HBase, Zookeeper |
| AP | Cassandra, CouchDB, Amazon Dynamo |

---

## 7. ACID vs. BASE

**ACID** (bases de datos relacionales):
| Propiedad | Descripción |
|-----------|-------------|
| **Atomicity** | Una transacción es una unidad indivisible — o se ejecutan todas las operaciones o no se ejecuta ninguna. Si algo falla, el sistema deshace todo y vuelve al estado anterior. |
| **Consistency** | La BD pasa de un estado válido a otro estado válido |
| **Isolation** | Las transacciones concurrentes no se interfieren entre sí |
| **Durability** | Una vez confirmada, la transacción persiste aunque haya fallos |

**BASE** (NoSQL):
| Propiedad | Descripción |
|-----------|-------------|
| **Basically Available** | El sistema garantiza disponibilidad según CAP |
| **Soft State** | El estado puede cambiar con el tiempo aunque no haya escrituras nuevas (por propagación asíncrona entre nodos) |
| **Eventual Consistency** | El sistema llegará a ser consistente si deja de recibir actualizaciones |

No es binario — es un espectro. MongoDB, por ejemplo, permite transacciones multi-documento que son ACID.

---

## 8. Quórums

Mecanismo para controlar el nivel de consistencia en sistemas replicados (con N nodos réplica).

- **W** = nodos que deben confirmar una escritura
- **R** = nodos que deben responder una lectura
- **N** = total de réplicas

**Reglas**:
- `W > N/2` → garantiza que no pueden coexistir dos escrituras conflictivas (mayoría)
- `R + W > N` → garantiza que siempre lees al menos un nodo que tiene la escritura más reciente
- `R + W ≤ N` → consistencia eventual (puedes leer datos obsoletos)

---

## 9. Write-Write: estrategias de conflicto

Cuando dos nodos reciben escrituras distintas sobre el mismo dato al mismo tiempo (en replicación peer-to-peer), hay conflicto.

**Estrategia pesimista**: bloquear antes de escribir — nadie puede modificar el dato mientras otro lo está usando. Evita el conflicto pero reduce disponibilidad.

**Estrategia optimista**: dejar que ambas escrituras ocurran y detectar el conflicto después. Para detectarlo se usa el **Vector Stamp** — cada nodo lleva un contador de versiones. Si al comparar dos versiones una no es mayor que la otra en todos los campos, hay conflicto.

> Este concepto encaja mejor tras estudiar replicación (Día 6). El write-write es una consecuencia directa de la replicación peer-to-peer.

---

## 10. BD Objeto-Relacionales (SQL:1999)

Extensión del modelo relacional que añade características de orientación a objetos manteniendo SQL.

### CREATE TYPE — definir un tipo estructurado

```sql
CREATE TYPE tipo_direccion AS (
    calle     VARCHAR(255),
    numero    VARCHAR(5),
    cp        VARCHAR(5),
    localidad VARCHAR(100)
);
```

### CREATE TABLE OF — tabla basada en un tipo

```sql
CREATE TYPE empleado_t AS (
    nombre  VARCHAR(50),
    salario DECIMAL(9,2)
) INSTANTIABLE NOT FINAL;

CREATE TABLE empleados OF empleado_t;
```

`INSTANTIABLE` = se pueden crear instancias. `NOT FINAL` = puede tener subtipos.

### Herencia con UNDER

```sql
CREATE TYPE directivo_t UNDER empleado_t (
    presupuesto DECIMAL(12,2)
);

CREATE TABLE directivos OF directivo_t UNDER empleados;
```

Un directivo hereda `nombre` y `salario`, y añade `presupuesto`.

**Comportamiento**:
- SELECT en `empleados` devuelve también los directivos
- `ONLY(empleados)` devuelve solo empleados que no son directivos
- INSERT en `directivos` inserta también en `empleados`
- DELETE desde `empleados` borra en cascada hacia `directivos`

### Referencias con →

```sql
SELECT e.nombre, e.dep->nombre
FROM empleados e
WHERE e.dep->dir.localidad = 'Santiago';
```

### ONLY()

```sql
SELECT * FROM ONLY(empleados); -- excluye directivos
```

---

## 11. PostgreSQL — Agregados y tipos no relacionales

PostgreSQL es relacional pero soporta datos no normalizados: arrays, tipos compuestos y JSON.

### Arrays

```sql
CREATE TABLE empleado (
    id           INT PRIMARY KEY,
    nombre       VARCHAR(100),
    paga_mensual NUMERIC(8,2)[12]  -- 12 sueldos en una columna
);
```

Acceder a elementos (índices desde 1):
```sql
SELECT nombre, paga_mensual[7] FROM empleado;    -- mes 7
SELECT nombre, paga_mensual[4:8] FROM empleado;  -- meses 4 al 8
SELECT nombre FROM empleado WHERE paga_mensual[8] > 13000;
```

### UNNEST — convertir array en filas

Para filtrar o agregar elementos del array necesitas expandirlo a filas:

```sql
SELECT id, nombre, paga
FROM empleado, unnest(paga_mensual) AS pagas(paga);
```

**WITH ORDINALITY** — para obtener también la posición de cada elemento:

```sql
SELECT id, nombre, paga, mes
FROM empleado,
     unnest(paga_mensual) WITH ORDINALITY AS pagas(paga, mes)
WHERE id = 1
ORDER BY mes;
```

### Tipos compuestos

```sql
CREATE TYPE tipo_direccion AS (
    calle     VARCHAR(255),
    numero    VARCHAR(5),
    cp        VARCHAR(5),
    localidad VARCHAR(100)
);

CREATE TABLE empleado2 (
    id           INT PRIMARY KEY,
    nombre       VARCHAR(100),
    direccion    tipo_direccion,
    paga_mensual NUMERIC(8,2)[12]
);
```

Se pueden combinar arrays y tipos compuestos — por ejemplo, un array de objetos compuestos.

### JSON

Operadores:
- `->` — accede a un campo y devuelve JSON
- `->>` — accede a un campo y devuelve texto (para comparar o mostrar)
- `json_array_elements(col)` — expande un array JSON a filas (equivalente a UNNEST para JSON)
- `json_agg(...)` — agrupa resultados en un array JSON (lo contrario de json_array_elements)

Ejemplo — nombre de cada empresa productora:
```sql
SELECT empresa->>'name' AS productora
FROM peliculas,
     json_array_elements(empresas_produccion) AS e(empresa);
```

Ejemplo — agrupar actores de cada película en un array JSON:
```sql
SELECT p.titulo, json_agg(ereparto->>'name') AS actores
FROM peliculas p, creditos c,
     json_array_elements(c.reparto) AS r(ereparto)
WHERE p.id = c.pelicula
GROUP BY p.titulo;
```
