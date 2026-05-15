---
title: Simulacro 02 — Bases de Datos (modelo 2025)
type: simulacro
fecha: 2026-05-15
temas: [CAP, replicación, write-write, modelado, sharding, MongoDB, Neo4J]
---

# Simulacro 02 — Bases de Datos (modelo 2025)

**Tiempo orientativo: 25 minutos**
**Formato**: 3 preguntas de desarrollo. Preguntas distintas al Simulacro 01.

---

## Pregunta 1

Una red social usa una base de datos distribuida con replicación peer-to-peer entre 4 nodos (N=4). Dos usuarios editan simultáneamente el mismo perfil desde nodos distintos.

**a)** ¿Qué tipo de conflicto se produce? ¿Por qué es más grave que el conflicto que aparece en replicación maestro-esclavo?

**b)** El equipo decide usar Vector Stamp para gestionar este problema. Explica cómo funciona el Vector Stamp y qué hace exactamente cuando detecta un conflicto. ¿Qué NO hace?

**c)** El sistema actualmente tiene W=2, R=2. ¿Se evitan los conflictos de escritura con esta configuración? ¿Qué habría que cambiar para evitarlos?

---

## Pregunta 2

Una empresa de e-commerce tiene esta colección MongoDB de productos:

```json
{
  "_id": "prod-001",
  "nombre": "Auriculares BT",
  "precio": 49.99,
  "categorias": ["electrónica", "audio"],
  "fabricante": {
    "nombre": "SoundCo",
    "pais": "Alemania"
  }
}
```

El equipo debate si modelar los pedidos con el producto **embebido** o **referenciado**.

**a)** ¿Cuándo es correcto embeber el producto dentro del pedido? Da un criterio concreto y explica qué ventaja operacional ofrece.

**b)** ¿Cuándo es mejor usar una referencia? ¿Qué problema concreto resuelve la referencia que el embedding no puede resolver?

**c)** El precio del producto cambia frecuentemente. ¿Afecta esto a la decisión de embedding vs referencia? Razona.

---

## Pregunta 3

Dado el siguiente bloque de comandos ejecutados para configurar un cluster MongoDB:

```javascript
mongod --configsvr --replSet rsConfig --port 27017 --dbpath /data/config
```

```javascript
mongos --configdb rsConfig/localhost:27017 --port 27020
```

```javascript
sh.addShard("rsShard1/localhost:27018")
```

**a)** ¿Qué hace el primer comando? ¿Qué significa `--configsvr` y qué rol tiene ese nodo en el cluster?

**b)** ¿Qué es el `mongos`? ¿Qué parámetro le indica dónde están los metadatos del cluster y por qué los necesita?

**c)** ¿Qué hace `sh.addShard`? ¿Por qué el shard se especifica como `"rsShard1/localhost:27018"` y no solo como `"localhost:27018"`?

---

## Soluciones

> Completa el simulacro antes de leer esto.

<details>
<summary>Ver soluciones</summary>

### P1a) Tipo de conflicto y gravedad
Se produce un conflicto **escritura-escritura (W-W)**. Es más grave que el conflicto R-W de maestro-esclavo porque:
- En maestro-esclavo el conflicto R-W es **transitorio**: cuando el secundario se sincroniza con el primario, desaparece solo.
- En P2P el conflicto W-W es **permanente**: dos versiones incompatibles del mismo dato existen en nodos distintos y no se resuelven solas. Requieren intervención explícita.

### P1b) Vector Stamp
El Vector Stamp es un array con un contador por nodo: `[nodo1: 2, nodo2: 5, nodo3: 1, nodo4: 3]`. Cada vez que un nodo escribe, incrementa su propio contador.

Para comparar dos versiones: A es más reciente que B si **todos** los contadores de A son ≥ los de B y **al menos uno** es estrictamente mayor.

Si dos versiones tienen valores mayores en posiciones distintas — `A=[3,5,1,2]` vs `B=[3,4,2,2]` — ninguna es más reciente que la otra → **conflicto detectado**.

Lo que **NO hace**: no resuelve el conflicto. Solo lo detecta. La resolución depende de la aplicación (last-write-wins, merge, exponer al usuario).

### P1c) ¿W=2 evita conflictos?
W=2, N=4. Para evitar conflictos W-W se necesita W > N/2 = 2. Como 2 no es mayor que 2 (es igual), **no se garantiza**. Podrían coexistir dos escrituras con quórum de 2 cada una usando 4 nodos en total. Habría que subir W a 3 (3 > 2).

### P2a) Cuándo embeber
Cuando el producto y el pedido **siempre se acceden juntos** y la relación es 1-a-pocos. Ventaja operacional: una sola lectura devuelve todo el documento sin necesidad de hacer una segunda consulta. Además la operación es atómica — todo el pedido incluido el producto vive en un único documento.

### P2b) Cuándo referenciar
Cuando el producto se accede de forma independiente (ej: página de detalle del producto, buscador). La referencia evita **duplicación**: si el producto está embebido en 10.000 pedidos y hay que actualizar un campo, habría que modificar 10.000 documentos. Con referencia se actualiza en un único sitio.

### P2c) Precio variable y embedding
El precio cambia frecuentemente → si está embebido en todos los pedidos, cada cambio de precio requiere actualizar todos los pedidos existentes → problemático. Sin embargo, en un pedido **ya realizado** el precio debe quedar fijo (el precio que pagó el cliente en ese momento). Conclusión: para pedidos históricos, embeber el precio en el momento de la compra es correcto y deseable. Para el catálogo de productos activo, usar referencia.

### P3a) --configsvr
Arranca un `mongod` en modo **Config Server**. Los Config Servers almacenan los **metadatos del cluster**: qué chunks existen, en qué shard está cada chunk, qué shards hay. Sin ellos el cluster no sabe dónde están los datos. Siempre forman un replica set propio (`--replSet rsConfig`).

### P3b) mongos y configdb
El `mongos` es el **router** del cluster — el punto de entrada para los clientes. No almacena datos. Recibe las consultas, consulta los Config Servers para saber en qué shard está cada dato, y redirige la operación al shard correcto. El parámetro `--configdb rsConfig/localhost:27017` le indica dónde están los Config Servers para poder leer los metadatos.

### P3c) sh.addShard y nombre del replica set
`sh.addShard` registra un nuevo shard en el cluster. Se especifica como `"rsShard1/localhost:27018"` (nombre del replica set + host) y no solo el host porque cada shard en MongoDB es internamente un **replica set**. El cluster necesita conocer el nombre del replica set para poder comunicarse con cualquiera de sus miembros, no solo con el nodo indicado — si ese nodo falla, el cluster puede contactar con los otros miembros del replica set.

</details>
