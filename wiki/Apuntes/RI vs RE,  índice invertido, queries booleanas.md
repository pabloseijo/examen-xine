
## RE vs RI

Enronces empezamos por definir que es recuperazón de información y recuperación estructurada. 

Por una parte, la recuperación estructurada es buscar datos organizados en tablas a través, de por ejemplo, consultas sql, por lo que el resultado es exacto. Es decir, la búsqueda o está o no está. 

Por otra parte, la recuperación de información se hace sobre texto no estructurado, como puede ser una página web, o un pdf, de tal manera que la consulta es lenguaje natural o booleana. Por lo que el resultado no es tanto si está o no está, si no un ranking por relevancia.

La diferencia clave es que para la RE los resultados son exactos, mientras que en RI dos usuarios pueden hacer la misma query y necesitar documentos distintos. 


## Modelo booleano: Matriz de incidencia

La matriz de incidencia es una idea simple, y no más que una tabla donde las filas son los términos, las columnas los documentos y los valores 0 o 1 en función de si aparecen o no. 

Un ejemplo podría ser el siguiente:


|       | Doc1 | Doc2 | Doc3 |
| ----- | ---- | ---- | ---- |
| gato  | 1    | 0    | 1    |
| perro | 0    | 1    | 1    |
| casa  | 1    | 1    | 0    |

De esta manera, sabemos que la palabra gato aparece en el documento 1 y 3 pero no en el dos. ¿Pero cual es el problema de esto? Si tenemos 1 millón de documentos y medio millón de términos, la matriz tendría 500 mil millines de celdas, casi todas a 0, que es del todo inviable. 

## Índice invertido
Esto es lo que usamos como solución al problema que surgía con la matriz de incidencia, y es que en lugar de guardar todos los 0, solo guardamos donde aparece cada término. 

Para ello usamos un diccionario, que será la lista de todos los términos ordenada alfabéticamente. 

Por otra parte cada término tendrá una **posting list** que es básicamente una lista de los docIDs de los documentos en los que aparece, ordenada de menor a mayor. 

Sería algo así:

gato -> [1,3]
perro -> [2,3]
casa -> [1,2]

¿Por qué ordenada por DocID? Para poder hacer la intersección de manera eficiente


## AND Query - merge algorithm

Básicamente es hacer una operación and para ver en que archivos salen los dos términos. 

Por ejemplo, para gato and perro:

1. Obtenemos la posting list de gato: [1,3]
2. Obtenemos la posting list de perro: [2,3]
3. Ponemos un puntero en cada uno, de tal manera que avanzamos al que apunte al menor de los términos

```
`gato:  [1, 3]`
		 ↑ 
`perro: [2, 3]`
         ↑

`1 < 2 → avanza gato`
`gato:  [1, 3]`
			↑
`perro: [2, 3]`
         ↑
         
`3 > 2 → avanza perro`
`gato:  [1, 3]`
			↑
`perro: [2, 3]`
	        ↑         

`3 = 3 → añadir 3 al resultado, avanzar ambos`
```

El resultado de esta operación es básicamente los documentos en los cuales aparecen los dos términos, que en este caso es [3].

Hay dos operaciones más:
**OR**: igual pero añades al resultado cuando cualquiera de los dos punteros coincide o cuando avanzas el menor. Usando el merge, es básicamente la unión de las dos lsitas

**NOT**: complemento — los docIDs que no están en la posting list del término. No usa el algortimo, son todos los archivos que no salen en las posting list. 

## 5. Optimización para AND con 3+ términos

Si tienes `sistema AND operativo AND Windows` no da igual el orden en que interseques.

**Regla**: ordena los términos por **document frequency (df) ascendente** — el que aparece en menos documentos, primero.

¿Por qué? La primera intersección produce un resultado intermedio más pequeño, lo que hace que las siguientes intersecciones sean más rápidas.

Ejemplo: df(Windows)=1000, df(sistema)=50000, df(operativo)=30000 → Orden óptimo: Windows ∩ operativo ∩ sistema → Primera intersección ya descarta la mayoría de documentos

## Ejercicios — hazlos antes de seguir

Dado este corpus:

- **Doc1**: "el gato come pescado"
- **Doc2**: "el perro come carne"
- **Doc3**: "el gato y el perro juegan"
- **Doc4**: "la carne de pescado es cara"

**Ejercicio 1**: Construye el índice invertido (ignora stopwords: "el", "la", "y", "de", "es"). ¿Qué posting list tiene cada término?

gato -> [1,2]
come -> [1,2]
pescado -> [1,4]
perro -> [2,3]
carne ->[2, 4]
juegan -> [3]
cara -> [4]

**Ejercicio 2**: Resuelve paso a paso con el merge algorithm: `gato AND perro`

Obtenemos las posting list de cada uno de los términos:
gato->[1,2]
perro->[2,3]

ponemos un puntero en cada una de ellas 

```
gato->[1,2]
       ↑
perro->[2,3]
		↑
		
1<2 por lo que avanzamos gato

gato->[1,2]
         ↑
perro->[2,3]
		↑
		
2=2 metemos el 2 en la lista de salida [2]

Avanzamos los dos
gato->[1,2]
	         ↑
perro->[2,3]
		  ↑

Se acabó la lista de gato, por lo que el resultado es [2]

```


**Ejercicio 3**: Resuelve: `gato AND pescado AND perro`. ¿En qué orden procesarías los términos para optimizarlo? ¿Por qué?

Está vacío porque no hay ninguna frase en la que salgan los tres

**Ejercicio 4** (conceptual): Si "el" no fuera stopword y estuviera en el índice, ¿en qué parte del índice tendría más impacto su posting list — en el diccionario o en el espacio de postings? ¿Por qué?
En el diccionario, tan solo añadiría una entrada más, mientras que en el espacio de postings, al salir en otdas las frases, añade tantas entradas como frases hay, por lo que tendría más impacto. 