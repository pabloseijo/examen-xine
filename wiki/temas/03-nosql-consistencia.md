---
title: NoSQL — Consistencia
type: tema
fuentes: [NoSQL3.Consistencia.pdf]
última_actualización: 2026-05-07
---

# NoSQL — Consistencia

## Teorema CAP

### Las tres propiedades
- **C (Consistency)**: Todos los nodos ven los mismos datos al mismo tiempo. Cualquier lectura devuelve el valor más reciente escrito.
- **A (Availability)**: Cada petición recibida por un nodo que no falla DEBE ser respondida. *(Definición exacta del slide, en rojo — importante para el examen)*
- **P (Partition tolerance)**: El sistema sigue funcionando aunque haya particiones de red (nodos que no pueden comunicarse entre sí).

### El teorema
En un sistema distribuido, **no se pueden garantizar las tres propiedades simultáneamente**. Ante una partición de red, hay que elegir entre consistencia y disponibilidad.

### Las tres combinaciones posibles

| Tipo | Descripción | Ejemplo |
|------|-------------|---------|
| **CA** | Sin tolerancia a particiones. Solo posible en un **sistema centralizado** (un solo nodo). | BD relacionales tradicionales |
| **CP** | Ante partición, el sistema prefiere ser consistente. Algunos nodos pueden volverse inaccesibles. | HBase, Zookeeper, MongoDB (por defecto) |
| **AP** | Ante partición, el sistema sigue respondiendo pero puede dar datos obsoletos. | Cassandra, CouchDB, Amazon Dynamo |

**Conclusión práctica**: En un cluster real, siempre hay riesgo de partición de red, por lo que solo podemos elegir entre CP y AP. Los sistemas CA son solo sistemas de un nodo.

### Ejemplo: reserva de habitación de hotel
- **CP**: Si hay partición, el sistema rechaza reservas (no puede confirmar disponibilidad). Consistente pero baja disponibilidad.
- **AP**: Si hay partición, el sistema permite reservas en ambos lados. Puede haber overbooking. Disponible pero inconsistente.

### Ejemplo: carro de compra (Amazon Dynamo)
- El carro de compra siempre permite añadir items (máxima disponibilidad, AP)
- Si hay conflicto de versiones, se muestran ambas versiones al usuario en el checkout
- El proceso final une los carros: el conocimiento del dominio resuelve el conflicto

---

## Consistencia de lectura

### ¿Cuándo importa?
No todas las aplicaciones necesitan el mismo nivel de consistencia:
- **Inversión en bolsa**: precio del activo → inconsistencia puede ser catastrófica
- **Noticia de blog**: contador de visitas → consistencia eventual es aceptable

### Ventana de inconsistencia
El tiempo durante el cual un nodo secundario puede tener datos obsoletos. La ventana depende de:
- Velocidad de red
- Frecuencia de sincronización
- Carga del sistema

### Relajar consistencia para mejorar rendimiento
La consistencia fuerte requiere coordinación entre nodos (costosa). Relajar consistencia permite:
- Menor latencia
- Mayor throughput
- Mayor disponibilidad

---

## BASE vs ACID

### ACID (Bases de datos relacionales)
- **Atomicity**: la transacción se completa o se deshace entera
- **Consistency**: la BD pasa de un estado válido a otro estado válido
- **Isolation**: las transacciones concurrentes no se interfieren
- **Durability**: una vez confirmada, la transacción persiste

### BASE (NoSQL)
- **B**asically **A**vailable: el sistema garantiza disponibilidad (según CAP)
- **S**oft State: el estado del sistema puede cambiar con el tiempo incluso sin nuevas escrituras (por propagación asíncrona)
- **E**ventual Consistency: el sistema llegará a ser consistente cuando deje de recibir actualizaciones

### Compromiso ACID-BASE
No es binario: existe un espectro. Se puede elegir el nivel de consistencia apropiado para cada operación según el dominio.

---

## Compromiso consistencia-latencia

- Más nodos participantes en una operación = más consistencia pero mayor latencia
- La disponibilidad puede verse como el límite máximo de latencia tolerable: si la latencia supera ese límite, la operación es "no disponible"

---

## Durabilidad de replicación

### Escritura simple vs. escritura replicada
- **Escritura simple**: confirmar al cliente en cuanto escribe en un nodo. Mayor rendimiento, menor durabilidad.
- **Escritura replicada**: confirmar solo después de que N réplicas hayan persistido el dato. Mayor durabilidad, menor rendimiento y disponibilidad.

### Trade-off
Esperar a más réplicas antes de confirmar:
- Mejora durabilidad (más copias existen antes de confirmar)
- Empeora rendimiento (más tiempo de espera)
- Empeora disponibilidad (si alguna réplica falla, la escritura falla)

---

## Quorums

### Concepto
Un quórum es el número mínimo de nodos que deben participar en una operación para que sea válida.

Parámetros:
- **N**: factor de replicación (número de copias)
- **W**: quórum de escritura (mínimo de nodos que deben confirmar una escritura)
- **R**: quórum de lectura (mínimo de nodos que deben responder a una lectura)

### Fórmulas

**Consistencia de escritura** (evitar conflictos entre escrituras concurrentes):
```
W > N/2
```
Solo puede haber una escritura con mayoría → no pueden coexistir dos versiones con quórum

**Consistencia de lectura** (garantizar leer la escritura más reciente):
```
R + W > N
```
La intersección entre el conjunto de escritura y el conjunto de lectura es siempre no vacía → al menos un nodo en la lectura tiene la versión más reciente.

### Factor de replicación N=3 (el más común)

| W | R | Característica |
|---|---|----------------|
| 3 | 1 | Escrituras lentas (esperar 3 nodos). Lecturas ultrarrápidas y consistentes. |
| 2 | 2 | Equilibrado. R+W=4>3. Consistente. |
| 1 | 3 | Escrituras ultrarrápidas. Lecturas lentas. Consistente. |
| 1 | 1 | Máximo rendimiento. Sin consistencia garantizada. Eventual. |

### Ajuste según caso de uso
- **Lecturas consistentes y rápidas**: W alto (3), R bajo (1) → escrituras lentas
- **Escrituras rápidas con consistencia baja**: W bajo (1), N bajo → riesgo de inconsistencia

---

## Versiones y control de concurrencia

### Optimistic Offline Lock (Bloqueo optimista)
- No bloquear recursos durante la lectura
- Al escribir, verificar que nadie más haya modificado el dato (mediante número de versión)
- Si hay conflicto: rechazar la escritura y pedir al cliente que reintente

### Marcas de versión
Para detectar si un dato ha cambiado entre lectura y escritura:
- **Contador**: entero que se incrementa con cada modificación
- **Timestamp**: marca temporal (riesgo si los relojes no están sincronizados)
- **Hash del contenido**: detecta cambios, pero no orden
- **Secuencias aleatorias**: UUID generado en cada escritura
- **CouchDB**: combina contador + hash (e.g., `3-abcdef1234`)

---

## Vector Stamp (Vectores de versión)

### Estructura
Un vector con un contador por nodo del cluster. Ejemplo con 3 nodos:
```
[nodo1: 3, nodo2: 5, nodo3: 6]
```

### Cómo evoluciona
1. Estado inicial: `[0, 0, 0]`
2. Nodo1 escribe: `[1, 0, 0]`
3. Nodo2 escribe: `[1, 1, 0]`
4. Si solo el nodo2 actualiza de nuevo: `[1, 2, 0]` → `[3, 6, 6]` se convierte en el estado

### Comparación de versiones
El vector A es **más reciente que B** si y solo si:
- Todos los contadores de A son >= los de B, Y
- Al menos un contador de A es estrictamente > el de B

### Conflicto escritura-escritura
Si dos vectores tienen valores mayores en posiciones distintas:
```
A: [3, 5, 6]
B: [3, 6, 5]
```
→ A tiene 6 en pos3 pero B tiene 6 en pos2: **CONFLICTO**. Ninguno es más reciente que el otro.

### Limitación
El Vector Stamp **detecta** inconsistencias pero **no las resuelve**. La resolución debe hacerla la aplicación o el sistema con otra estrategia (merge, last-write-wins, etc.).

Ver también: [[teorema-cap]], [[consistencia-eventual]], [[replicacion]], [[acid-vs-base]]
