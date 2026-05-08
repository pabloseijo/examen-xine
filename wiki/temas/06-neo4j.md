---
title: Neo4J — Introducción y Cypher
type: tema
fuentes: [IntroNeo4J.pdf, Neo4J.txt, Xestión de Información non Estruturada [G4012453] [2025_2026]_ Introducción a Neo4J _ Campus Virtual.pdf]
última_actualización: 2026-05-07
---

# Neo4J — Introducción y Cypher

## Qué es Neo4J

Neo4J es la BD de grafos más popular. Almacena datos como **nodos** (entidades) y **relaciones** (aristas) entre nodos, con **propiedades** en ambos. Usa el lenguaje declarativo **Cypher** para consultas.

### Ediciones
- **Community Edition**: despliegue en una sola instancia. Funcionalidad completa: ACID, Cypher, APIs. Para aprendizaje y pequeños proyectos.
- **Enterprise Edition**: clustering, mejor rendimiento, seguridad avanzada.

### Herramientas
| Herramienta | Descripción |
|-------------|-------------|
| **Cypher Shell** | Cliente de terminal para ejecutar Cypher |
| **Browser** | Interfaz web con visualización del grafo |
| **Desktop** | IDE de desarrollo y gestión |
| **Bloom** | Exploración visual avanzada del grafo |

---

## Conceptos del modelo de datos de grafos

### Nodos
- Representan **entidades** (personas, películas, ciudades...)
- El grafo más simple tiene un solo nodo
- Pueden no tener etiqueta, o tener **múltiples etiquetas**

### Etiquetas (Labels)
- Agrupan nodos en conjuntos semánticos: `Person`, `Movie`, `Director`
- Se pueden añadir o quitar en tiempo de ejecución (pueden ser temporales: `Suspended`)
- Un nodo puede tener **0 o más** etiquetas
- Permiten operar sobre subconjuntos de nodos

```
(Person:Actor)   vs   (Movie)   vs   (Person:Director)
```

### Relaciones
- **Conectan dos nodos** (o un nodo consigo mismo — reflexivas)
- Tienen exactamente **un tipo** (ACTED_IN, DIRECTED, KNOWS...)
- Son **dirigidas**: tienen sentido (saliente `-->` o entrante `<--`)
- No es necesario duplicar la relación en ambas direcciones — se puede navegar en ambos sentidos
- Pueden ser reflexivas

### Propiedades
- Pares **clave-valor** en nodos y relaciones
- Tipos: número, string, boolean, lista, mapa
- Ejemplos: `name = 'Tom Hanks'`, `born = 1956`, `roles = ['Forrest']`

### Recorridos (Traversals)
- La forma fundamental de consultar una BD de grafos
- Consisten en seguir relaciones de nodo en nodo bajo ciertas reglas
- Ejemplo: "películas en las que actuó Tom Hanks"

### Esquema
- **Opcional** en Neo4J (no obligatorio)
- Compuesto por **índices** (mejoran rendimiento) y **restricciones** (aseguran integridad)
- ```cypher
  CREATE CONSTRAINT idPelicula FOR (p:Pelicula) REQUIRE p.id IS UNIQUE;
  CREATE INDEX FOR (p:Pelicula) ON (p.titulo);
  ```

### Convenciones de nomenclatura (Cypher distingue mayúsculas/minúsculas)
| Elemento | Convención | Ejemplo |
|----------|------------|---------|
| Etiquetas de nodo | CamelCase (mayúscula inicial) | `VehicleOwner`, `Movie` |
| Tipos de relación | MAYÚSCULAS con guión bajo | `ACTED_IN`, `OWS_VEHICLE` |
| Propiedades | lowerCamelCase (minúscula inicial) | `firstName`, `releaseYear` |

---

## Lenguaje Cypher

### Filosofía
Cypher es un lenguaje de patrones: se describe **qué** debe encontrarse en el grafo, no **cómo** buscarlo. Inspirado en la notación visual de grafos.

### Sintaxis de nodos
```cypher
()                           -- nodo anónimo
(n)                          -- variable n que itera por nodos
(:Movie)                     -- nodo con etiqueta Movie
(m:Movie)                    -- variable m para nodos Movie
(m:Movie {title: "Matrix"})  -- nodo Movie con propiedad title
```

### Sintaxis de relaciones
```cypher
--                           -- relación sin dirección específica
-->                          -- relación saliente
<--                          -- relación entrante
-[r]->                       -- variable r para relación saliente
-[:ACTED_IN]->               -- relación saliente de tipo ACTED_IN
-[r:ACTED_IN]->              -- variable r para relación ACTED_IN
-[r:ACTED_IN {roles: ["Neo"]}]->  -- con filtro de propiedad
```

### Patrones completos
```cypher
-- Un patrón conecta nodos con relaciones
(keanu:Person:Actor {name: "Keanu Reeves"})
  -[r:ACTED_IN {roles: ["Neo"]}]->
  (matrix:Movie {title: "The Matrix"})

-- Variables de patrón (captura el camino completo)
acted_in = (:Person)-[:ACTED_IN]->(:Movie)
-- Acceder a partes: nodes(acted_in), relationships(acted_in), length(acted_in)
```

### Cláusulas principales

| Cláusula | Propósito |
|----------|-----------|
| `MATCH` | Encontrar patrones en el grafo |
| `WHERE` | Filtrar resultados |
| `RETURN` | Proyectar resultados |
| `CREATE` | Crear nodos o relaciones |
| `MERGE` | Crear si no existe (upsert) |
| `SET` | Modificar propiedades |
| `DELETE` | Eliminar nodos o relaciones |
| `WITH` | Pasar resultados de una cláusula a otra (pipeline) |
| `ORDER BY` | Ordenar resultados |
| `LIMIT` | Limitar número de resultados |
| `SKIP` | Saltar N resultados |
| `UNION` | Combinar resultados de dos consultas |
| `OPTIONAL MATCH` | MATCH que no falla si no encuentra el patrón (como LEFT JOIN) |
| `CALL {}` | Subconsulta |

---

## Consultas Cypher — Ejemplos del ejercicio de evaluación

### Crear relación DIRIGE
```cypher
MATCH (p:Persona)-[t:TRABAJO_EN]->(pelicula:Pelicula)
WHERE t.trabajo = "Director"
CREATE (p)-[:DIRIGE]->(pelicula);
```

### Reparto de Star Wars ordenado por orden
```cypher
MATCH (persona:Persona)-[r:ACTUO_EN]->(pelicula:Pelicula {titulo:"Star Wars"})
RETURN r.orden AS orden, r.personaje AS personaje, persona.nombre AS actor
ORDER BY r.orden;
```

### 10 películas con mayor beneficio
```cypher
MATCH (p:Pelicula)
RETURN p.titulo AS titulo,
       p.presupuesto AS presupuesto,
       p.ingresos AS ingresos,
       (p.ingresos - p.presupuesto) AS beneficio
ORDER BY beneficio DESC
LIMIT 10;
```

### Películas de Quentin Tarantino (actor Y director) con UNION
```cypher
MATCH (p:Persona {nombre:"Quentin Tarantino"})-[:ACTUO_EN]->(pelicula:Pelicula)
RETURN pelicula.titulo, pelicula.fechaEmision, pelicula.presupuesto, pelicula.ingresos, "actuo" AS participacion
UNION
MATCH (p:Persona {nombre:"Quentin Tarantino"})-[:DIRIGE]->(pelicula:Pelicula)
RETURN pelicula.titulo, pelicula.fechaEmision, pelicula.presupuesto, pelicula.ingresos, "dirigio" AS participacion
ORDER BY fechaEmision;
```

### Personal de The Godfather por departamento
```cypher
MATCH (persona:Persona)-[t:TRABAJO_EN]->(pelicula:Pelicula {titulo:"The Godfather"})
RETURN t.departamento AS departamento,
       count(*) AS numPersonas,
       collect({nombre: persona.nombre, trabajo: t.trabajo}) AS personal
ORDER BY numPersonas DESC, departamento;
```

### Película más rentable de Spielberg (con OPTIONAL MATCH)
```cypher
MATCH (steven:Persona {nombre:"Steven Spielberg"})-[t:TRABAJO_EN]->(pelicula:Pelicula)
WITH pelicula, t.trabajo AS trabajo
ORDER BY pelicula.ingresos DESC
LIMIT 1
OPTIONAL MATCH (actor:Persona)-[:ACTUO_EN]->(pelicula)
WITH pelicula, trabajo, count(DISTINCT actor) AS numActores
OPTIONAL MATCH (trabajador:Persona)-[:TRABAJO_EN]->(pelicula)
WHERE NOT (trabajador)-[:ACTUO_EN]->(pelicula)
RETURN pelicula.titulo, trabajo, pelicula.presupuesto, pelicula.ingresos,
       numActores, count(DISTINCT trabajador) AS numTrabajadores;
```

### Actores dirigidos por directores de Marlon Brando (dos saltos)
```cypher
MATCH (:Persona {nombre:"Marlon Brando"})-[:ACTUO_EN]->(:Pelicula)<-[:DIRIGE]-(director:Persona)
MATCH (director)-[:DIRIGE]->(:Pelicula)<-[:ACTUO_EN]-(actor:Persona)
RETURN DISTINCT actor.nombre AS nombre
ORDER BY nombre;
```

### Películas con más de un director
```cypher
MATCH (director:Persona)-[:DIRIGE]->(pelicula:Pelicula)
WITH pelicula,
     collect(DISTINCT director.nombre) AS directores,
     count(DISTINCT director) AS numDirectores
WHERE numDirectores > 1
RETURN pelicula.titulo, numDirectores, directores
ORDER BY numDirectores DESC, pelicula.titulo;
```

### Múltiples roles con CALL (subconsulta)
```cypher
MATCH (persona:Persona)
CALL {
  WITH persona
  MATCH (persona)-[:ACTUO_EN]->(pelicula:Pelicula)
  RETURN pelicula, "reparto" AS rol
  UNION ALL
  WITH persona
  MATCH (persona)-[t:TRABAJO_EN]->(pelicula:Pelicula)
  RETURN pelicula, t.trabajo AS rol
}
WITH persona, pelicula, collect(rol) AS roles, count(*) AS numRoles
RETURN persona.id, persona.nombre, pelicula.titulo, numRoles, roles
ORDER BY numRoles DESC, persona.nombre, pelicula.titulo
LIMIT 10;
```

### Actores del círculo de Tarantino (2 grados de separación)
```cypher
-- Actores directos de Tarantino
MATCH (qt:Persona {nombre:"Quentin Tarantino"})-[:DIRIGE]->(:Pelicula)<-[:ACTUO_EN]-(actor1:Persona)
RETURN DISTINCT actor1.nombre AS nombre
UNION
-- Actores que coincidieron con actores de Tarantino
MATCH (qt:Persona {nombre:"Quentin Tarantino"})-[:DIRIGE]->(:Pelicula)<-[:ACTUO_EN]-(actorDirigido:Persona)
MATCH (actorDirigido)-[:ACTUO_EN]->(:Pelicula)<-[:ACTUO_EN]-(actor2:Persona)
WHERE actor2 <> actorDirigido
RETURN DISTINCT actor2.nombre AS nombre
ORDER BY nombre;
```

---

## Importación de datos via JDBC (APOC)

```cypher
WITH "jdbc:postgresql://xine-grei-pgsql/xine?user=alumnobd&password=pwalumnobd" AS url
CALL apoc.load.jdbc(url,
  "SELECT id, titulo, presupuesto, fecha_emision, ingresos, duracion
   FROM peliculas ORDER BY ingresos DESC, id LIMIT 2000") YIELD row
CREATE (p:Pelicula {
  id: row.id,
  titulo: row.titulo,
  presupuesto: row.presupuesto,
  fechaEmision: row.fecha_emision,
  ingresos: row.ingresos,
  duracion: row.duracion
});
```

Ver también: [[neo4j]], [[grafos]], [[mongodb-vs-neo4j]], [[ejercicio-evaluacion-neo4j]]
