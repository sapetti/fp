# Programación funcional con Javascript. Parte III

En esta tercera entrega, daremos un paso a un lado y comentaremos algunos métodos que posee Array en JS y que comparten ciertas características con los principios que hemos visto sobre FP. También veremos algún otro ejemplo con Ramda para demostrar sus beneficios en distintas situaciones.


## Map

[Map](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/map) es un concepto muy sencillo, aplica la función que se pasa como argumento sobre cada uno de sus elementos y al final devuelve __un array nuevo__ con los resultados, no muta el array existente, y eso encaja con las directrices que hemos ido viendo de FP. Veamos un pequeño ejemplo.

```javascript
const letters = ['a', 'b', 'c', 'd']
const toUpperCaseLetters = letters.map(item => item.toUpperCase())

letters // ['a', 'b', 'c',  'd']
toUpperCaseLetters // ['A', 'B', 'C', 'D']
```

Como se puede ver arriba, a *map* le pasamos una función que será llamada para cada uno de los ítems que contiene el array. Vemos en el ejemplo que iteramos sobre el array, y para cada una de las letras contenidas en el array le aplicamos la transformación (*toUpperCase*) y devolvemos el nuevo valor. Finalmente como digo, un nuevo array se genera. Si vemos finalmente los arrays, el original mantiene sus valores iniciales (no lo hemos mutado) y el resultado final es uno nuevo que contiene los valores después de la transformación.

La función que le pasamos a *map*, recibe como argumento el ítem en cada iteración pero también puede recibir index y el array sobre el que estamos mapeando para aquellos casos en los que necesitemos información adicional durante la ejecución.

```javascript
const values = [1, 2, 3, 4, 5]
const mappedValues = values.map((item, index, arr) => index === 0              ? 'head' :
                                                      index + 1 !== arr.length ? 'body'
                                                                               : 'tail')

mappedValues // ['head', 'body', 'body', 'body', 'tail']
```

En el ejemplo vemos que si el index es el primero devolvemos head, si el ultimo tail y para el resto body.

También existe [*forEach*](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/forEach), pero a diferencia de *map* que hace el “mapping“ entre el valor de original y el final a través de la función que se pasa como argumento, *forEach* solo “procesa” los elementos a través de la función, no altera los elementos ni devuelve nada. Se suele aconsejar que para denotar una transformación usaremos *map* y para ejecutar algún proceso usaremos *forEach*, siempre pensando en quien pueda leer más tarde el código, podemos ser nosotros mismos un tiempo después del desarrollo, entienda rápidamente la intención que tiene esa sentencia.


##Filter

[*Filter*](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/filter) nos indica con su nombre que filtra los elementos del array y como *map* devuelve __un nuevo array__. Al igual que *map* recibe una función que será ejecuta para cada uno de los elementos en el array, pero en este caso debe evaluar a true o false para indicar si el elemento debe pasar el filtro o no.

```javascript
[1, 2, 3, 4, 5, 6, 7, 8, 9].filter(n => n % 2 === 0) // [2, 4, 6, 8]
```

En ejemplo filtramos el array y devolveremos los números pares en un nuevo array. La función que recibe *filter* también puede recibir el index y el array que estamos filtrando.


## Reduce

[*Reduce*](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/reduce) nos permite aplicar una función a todos los elementos del array pero, en vez de devolver un array, podemos devolver un valor de cualquier tipo (array incluido). *Reduce* a diferencia de las anteriores recibe dos argumentos, la función a aplicar sobre los ítems y el valor inicial que será del tipo esperado como retorno.
La función que se pasa como argumento recibe también dos argumentos, el valor acumulado por operaciones previas y el elemento sobre el que estamos iterando.

```javascript
[1, 2, 3, 4, 5, 6, 7, 8, 9].reduce((acc, cur) => acc + cur, 0) // 45

['a', 1, 'b', 2, 'c', 3].reduce((acc, cur) => acc + cur, '') // a1b2c3

['one', 'two', 'three'].reduce((acc, cur, index, arr) => {
    acc[cur] = index
    return acc
}, {}) // { one: 0, three: 2, two: 1 }

[
    {'name': 'Axl Rose', 'instrument': 'vocals'},
    {'name': 'Slash', 'instrument': 'guitar'},
    {'name': 'Izzy Stradlin', 'instrument': 'guitar'},
    {'name': 'Steven Adler', 'instrument': 'drums'},
    {'name': 'Duff McKagan', 'instrument': 'bass guitar'},
]
.reduce((acc, cur, index, arr) => acc.concat(cur.name), [])
// ['Axl Rose', 'Slash', 'Izzy Stradin', 'Steven Adler', 'Duff McKagan']
```

Como se puede observar, la reducción que realizamos __no solo da como resultado arrays__. En el primer ejemplo hemos sumado los elementos del array, en el segundo los hemos concatenado para crear un string, en el tercero hemos creado un objeto nuevo y en el último hemos creado un array de strings a partir de un array de objetos.
Ya podéis imaginar el potencial que demuestra este método. Es tal, que con *reduce* podríamos hacer lo mismo que con *map*, *filter* o ambos en una única función. Es más si tenemos que realizar varias operaciones (*map*, *filter*...), deberíamos usar *reduce* para evitar recorrer varias veces el array.

Aquí como antes iteramos una array de objetos, filtramos por el tipo de instrumento y solo añadimos al array final el nombre de aquellos que sean guitarristas.

```javascript
[
    {'name': 'Axl Rose', 'instrument': 'vocals'},
    {'name': 'Slash', 'instrument': 'guitar'},
    {'name': 'Izzy Stradlin', 'instrument': 'guitar'},
    {'name': 'Steven Adler', 'instrument': 'drums'},
    {'name': 'Duff McKagan', 'instrument': 'bass guitar'},
]
.reduce((acc, cur) => cur.instrument === 'guitar' ? acc.concat(cur.name) : acc, [])
// ['Slash', 'Izzy Stradlin']
```

Como pequeño resumen sobre dos de los tres métodos os dejo una imagen donde se ve clara la finalidad de *map* y *reduce*. Con un array de alimentos, aplicamos *map* y realizamos una transformación sobre cada uno de ellos, en nuestro caso los partimos/troceamos, y luego aplicamos *reduce* para de una lista de alimentos crear algo nuevo, en nuestro caso un sándwich. Quizás falta en la imagen aplicar *filter*, y que se quitase algún alimento de la lista sobre el resultado final, pero como ejemplo está bastante bien.

![Map-Reduce](https://github.com/sapetti/fp/blob/master/serie-fp/images/map-reduce.png)


## Otros metodos de Array

Vistos los tres anteriores deberíamos por lo menos mencionar otros que coinciden con la filosofía de FP.
* [of](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/of): Añadido con ES6, crea un nuevo array a partir de los valores enviados como argumentos.
* [concat](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/concat): Como ya hemos visto en algun ejemplo concatena dos arrays creando uno nuevo.
* [slice](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/slice): Recibe dos argumentos, index de inicio e index de fin. Con ellos realiza una copia parcial del array original. El index de inicio se incluye en la copia, el de fin no. No lo debemos confundir con [splice](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/splice), que si modifica el array original.

```javascript
[1, 2, 3, 4, 5].slice(0, 5) // [1, 2, 3, 4, 5]
[1, 2, 3, 4, 5].slice(1, 5) // [2, 3, 4, 5]
[1, 2, 3, 4, 5].slice(1, 4) // [2, 3, 4]
[1, 2, 3, 4, 5].slice(0, 1) // [1]
[1, 2, 3, 4, 5].slice(1, 2) // [2]
[1, 2, 3, 4, 5].slice(4, 5) // [5]
[1, 2, 3, 4, 5].slice(0, 0) // []
[1, 2, 3, 4, 5].slice(5, 5) // []
```

Seguro que no estoy mencionando todos aquellos que siguen las directrices de FP, pero es tan solo por ir viendo más opciones a la hora de trabajar con arrays.

Alguno estará pensando que pasa si quiere usar alguno de los métodos impuros disponibles, siempre podemos realizar una copia y aplicarlos en ella.

```javascript
const unsorted = [2, 1, 5, 3, 4]
[].concat(unsorted).sort() // [1, 2, 3, 4, 5]
[].concat(unsorted).sort().reverse() // [5, 4, 3, 2, 1]
unsorted // [2, 1, 5, 3, 4]
```

En el ejemplo usamos concat con un array nuevo para crear una copia y luego usar otras operaciones como [sort](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/sort) o [reverse](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Array/reverse).

## Encadenar operaciones

Lo acabamos de ver en el ejemplo anterior, pero no lo hemos mencionado, ya que con muchas de estas operaciones devolvemos arrays podemos ir concatenando unas con otras.

```javascript
const isALondonBand = band => band.origin === 'London'
const isActive = band => band.yearsActive.includes('present')
const addGuitaristQuote = band => Object.assign(
                                        {},
                                        band,
                                        { quote: `${band.members.guitarist} is the greatest guitarist active in London` }
                                    )

const bands = [metallica, ledZeppelin, theWho, gunsAndRoses]

bands.filter(isALondonBand) // First Class functions
    .map(addGuitaristQuote)
    .reduce((acc, band) => isActive(band) ? band.quote : '')
    // Pete Townshend is the greatest guitarist active in London!!

bands.filter(isActive)
    .sort((a, b) => Object.keys(a.members).length < Object.keys(b.members).length ? -1 : 1 ) // Filter creo un array nuevo
    .reverse() // Ya podriamos ordenarlo de esta manera en el sort, pero es tan solo un ejemplo
    .slice(0, 1) // Obtenemos la primera banda
    .reduce((acc, band) => acc.concat(Object.values(band.members)), []) // Ya que solo queremos los miembros,
                                                                        // los añadimos al array de retorno
    .reduce((acc, member) => acc + " " + member, 'The components of the band with more musicians are:')
    // The components of the band with more musicians are: Axl Rose Slash Izzy Stradlin Duff McKagan Steven Adler

```

Arriba vemos 2 ejemplos:
* En el primero filtramos por aquellas bandas que estén en activo, le añadimos un texto a cada uno de los objetos con key ‘quote’ y finalmente dentro reduce filtramos por la banda si está en activo y devolvemos el valor que hay en quote (extrayéndolo del array). Este ejemplo no esta tan mal, es bastante plano y legible, vamos aplicando pequeñas funciones reutilizables.
* El segundo, algo menos claro, filtra por grupos en activo, ordena ascendentemente por el número de miembros, invierte el orden, obtiene el primero de la lista que tendrá el mayor número de miembros, ya que queremos obtener un listado de miembros los añadimos a un array vacío (no podemos aplicar *map*, ya que nos devolvería un array conteniendo otro, algo como [[members]]), sobre este array de miembros aplicamos una reducción para crear la frase con todos sus miembros.

En estos ejemplos hemos usado algunas funcionalidades de Object, puedes ver su utilidad aquí:
* [Object.keys](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Object/keys)
* [Object.values](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Object/values)
* [Object.assign](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Object/assign)

Obviamente son solo ejemplos y podrían ser de mayor utilidad, pero espero que sirva como ejemplo para ver que podemos aplicar operaciones en cadena. Una cosa a tener en cuenta es que si, por ejemplo en el primer caso no hubiese ninguna banda de Londres, el resultado sería una cadena vacía, que va trabajando sobre arrays vacíos y el valor inicial en reduce es ‘’ por lo que sería el valor de retorno.

Como contra decir que perdemos point-free, ya que de primeras estamos haciendo referencia al array de origen (bands), y en cada una de las operaciones a su predecesor con el punto (.filter, .map, .reduce…). En el siguiente post veremos cómo *Ramda* puede ayudarnos.


## Resumen

En la entrada de hoy hemos visto unas utilidades de las que está provisto *Array* en JavaScript. Como podemos realizar operaciones complejas con ellos sin necesidad de añadir bucles e incluso sin condiciones cómo *if*. De nuevo, con piezas pequeñas realizamos funcionalidades más complejas. En la próxima entrada veremos algo más de Ramda.
