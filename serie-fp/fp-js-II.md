# Programación funcional con Javascript. Parte I

Tras ver algunos conceptos y buenas prácticas sobre programación funcional en el primer [post](https://github.com/sapetti/fp/blob/master/serie-fp/fp-js-I.md), en esta segunda entrega veremos algo más de código y cómo poder empezar a aplicarlo en nuestros proyectos. Además comenzaremos a ver algunas ventajas que ofrece [Ramda](http://ramdajs.com/) para trabajar en proyectos con Programación Funcional (de ahora en adelante FP)


## Recursion

Otra de las cosas que podrás leer sobre FP, es que no se usan loops. En FP se prefiere usar recursividad antes que bucles como for. La recursividad se da cuando una función se llama a sí misma, iterando hasta alcanzar una condición base y devolviendo el valor acumulado durante todas las llamadas.

```javascript
var finalCountdown = (num, arr = []) => {
    return num === 0 ? arr // En caso de que se alcance la condición base, devolvemos el resultado final
                    : finalCountdown(num-1, arr.concat(num)) // Sino seguimos llamando a la misma función con los nuevos parámetros 
} 
finalCountdown(10) // [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
```

Las funciones recursivas ya se suelen escribir de una manera pura, por lo que encajan con la filosofía de FP y son sencillas de testear. Además, encajaran bien cuando veamos la composición.

Un par de apuntes sobre recursividad en JS:
* Los motores de JS no están optimizados para usar recursividad. Esto quiere decir que dependiendo del caso puede no ser seguro usar recursividad y que podamos tener errores sobrepasando el stack de llamadas.
![Recursion](https://github.com/sapetti/fp/blob/master/serie-fp/images/recursion.png)
* [No todos los motores soportan el mismo stack de llamadas.](http://2ality.com/2014/04/call-stack-size.html) 
* Hay métodos alternativos como las funciones trampolín para solventar estas limitaciones y poder trabajar de manera segura.
![Trampoline](https://github.com/sapetti/fp/blob/master/serie-fp/images/trampoline.png)
* ES6 ha incluido [Tail Call Optimization](https://kangax.github.io/compat-table/es6/) pero aún no está totalmente implementado.

## Partial Application

La aplicación parcial consiste en dividir el arity (los argumentos de entrada de una función) según nos convenga e ir devolviendo funciones hasta que completemos todos esos argumentos y se ejecute totalmente la función. Veamos un ejemplo:

```javascript
var add = x => y => x + y

var add5 = add(5) // add devuelve una función

add5(10) // 15
```

Partimos add que tiene un arity de 2, en una Higher Order Function que recibe un argumento y devuelve una funcion (la funcion que asignamos a *add5* en el ejemplo) a la que parcialmente se aplica la variable *x* (5). La funcion que devolvemos desde *add* y que asignamos a *add5* es un [Closure](https://developer.mozilla.org/es/docs/Web/JavaScript/Closures) (una función recuerda el contexto en el que ha sido creada) por lo que cuando la llamamos solo tenemos que pasarle el valor de y para completar la ejecución y, sea cual sea el valor de y, añadirle 5. También podríamos llamar a la función *add* y pasarle los parámetros de la siguiente manera.

```javascript
add(5)(10) // 15
```

Obteniendo así el mismo resultado.

Veamos un ejemplo más completo:

```javascript
var pickPathProp = parentProp => childProp => obj => obj[parentProp][childProp]
var pickMembers = pickPathProp('members')
var pickSinger = pickMembers('singer')

var theWho = {
    yearsActive: '1964-1982, 1989, 1996-present',
    origin: 'London',
    members: {
        singer: 'Roger Daltrey',
        guitarist: 'Pete Townshend',
        bassGuitarist: 'John Entwistle',
        drummer: 'Keith Moon'
    }
}

var metallica = {
    yearsActive: '1981-present',
    origin: 'Los Angeles',
    members: {
        singer: 'James Hetfield',
        guitarist: 'Kirk Hammett',
        bassGuitarist: 'Robert Trujillo',
        drummer: 'Lars Ulrich'
    }
}

pickSinger(theWho) // Roger Daltrey
pickSinger(metallica) // James Hetfield
```

Revisemos el código, en este caso creamos la función *pickPathProp* que recibirá un argumento (1) y devolverá una función, que a su vez recibirá un argumento (2) y devolverá otra función, que a su vez recibirá otro argumento (3) y devolverá por fin el resultado, que será el valor que este dentro del objeto siguiendo el path de propiedades indicado por lo argumentos. Luego podemos ver como creamos 2 funciones, *pickMembers* devolverá una función pero ya hemos establecido *parentProp*, y *pickSinger* estableciendo *childProp* pero seguimos teniendo una función que espera el *obj*. Cuando llamamos *pickSinger* pasándole un objeto nos devuelve el valor siguiendo el path *obj.members.singer*.

Sin duda es válido, pero hay algo que ya puede empezar a ser incómodo (imaginemos un arity mayor). ¿Qué pasa si queremos llamar a la función de una sola vez? Tenemos que encadenar todas las llamadas, cada una con sus paréntesis. ¿Y si quisiésemos pasar 2 argumentos en un momento dado en vez de uno? Veamos ambos casos:

```javascript
pickPathProp('members')('guitarist')(metallica) // Kirk Hammett

pickPathProp('members', 'bassGuitarist')(theWho) // [Function]
```

En el último caso vemos como nos devuelve una función y no un resultado. Así funciona JS, si llamas a una función con más o menos parámetros de los esperado, esta función se ejecuta igual. En nuestro caso no es el resultado esperado. Esto lo soluciona con la técnica de Currying que veremos a continuación.

Pero antes de proseguir, decir que es aquí donde, para mí, empieza a haber un punto de inflexión. Usar las mismas funciones pero de otra manera, rompiendo bloques de código en piezas más pequeñas pudiéndolas reusar más fácilmente.


## Curry

Una técnica que, después de comenzar a usarse, piensas que debería estar presente en todos los lenguajes. Siguiendo lo visto con la aplicación parcial, Curry nos permite descomponer una función en sucesivas hasta que los argumentos de entrada hayan sido completados antes de ejecutarla. Para poder usar curry en JS, voy a usar [la función del mismo nombre](http://ramdajs.com/docs/#curry) incluida en la librería Ramda (de la que ya iremos viendo otras funcionalidades).

```javascript
var R = require('ramda')

var addFiveNumbers = R.curry((a, b, c, d, e) => a + b + c + d + e)

addFiveNumbers(11, 12, 13, 14, 15) // 65
addFiveNumbers(11, 12, 13, 14)(15) // 65
addFiveNumbers(11, 12, 13)(14, 15) // 65
addFiveNumbers(11, 12)(13)(14, 15) // 65
addFiveNumbers(11)(12, 13)(14)(15) // 65
//...
```

Para usar curry (una Higher Order Function), importamos la librería de Ramda y usamos la función curry sobre la implementación de la función. Cómo podéis ver, el resultado es siempre el mismo sin importar la manera en la que pasemos los argumentos. Mientras no pasemos todos los argumentos, se nos devolverá una función y una vez todos los argumentos se han pasado, nuestra función se ejecuta y nos devuelve el resultado.

Veamos de nuevo el ejemplo anterior, ahora usando la función curry.

```javascript
var pickPathProp = R.curry((parentProp, childProp, obj) => obj[parentProp][childProp])
var pickMembers = pickPathProp('members')
var pickSinger = pickMembers('singer')

pickSinger(theWho) // 'Roger Daltrey'
pickSinger(metallica) // 'James Hetfield'

pickPathProp('members')('guitarist')(metallica) // Kirk Hammett
pickPathProp('members', 'bassGuitarist')(theWho) // John Entwistle
pickPathProp('members')('drummer', metallica) // Lars Ulrich
```

Ahora podemos llamar la función *pickPathProp* de distintas maneras y sigue el funcionamiento que esperamos.

Por sí solo puede parecer que curry no aporta demasiado y que no es más que otra utilidad que quizás usemos en algún momento puntual. Pero veamos cómo nos puede ayudar con otras técnicas de la FP.


## Function Composition

Cuando empecé a leer sobre FP, algunos artículos comentaban que era similar al Lego, que juntando pequeñas piezas, sin valor por si solas, podías terminar construyendo estructuras con cualquier forma. Con la Composición de funciones es sin duda lo que podemos conseguir, juntar pequeñas funciones que terminan construyendo una aplicación o funcionalidad completa.

Veamos un ejemplo, digamos que tenemos unas pequeñas funciones (puras) que reusamos por nuestro código y las queremos ir aplicando en cadena a algún objeto.

```javascript
var capitalizeWord = word => word.toUpperCase()
var awesomize = word => `${word} is awesome`
var emphasise = phrase => `${phrase}!!!`
```

No van a salvar el mundo, pero bien nos valen de ejemplo. Y ahora veamos la implementación usando todas ellas en una función para generar el resultado esperado.

```javascript
var ledZeppelin = {
    yearsActive: '1968-1980',
    origin: 'London',
    members: {
        singer: 'Robert Plant',
        guitarist: 'Jimmy Page',
        bassGuitarist: 'John Paul Jones',
        drummer: 'John Bonham'
    }
}

var awesomizeSinger = band => {
    var singer = pickSinger(band)
    var cSinger = capitalizeWord(singer)
    var acSinger = awesomize(cSinger)
    var eacSinger = emphasise(acSinger)
    return eacSinger
}

awesomizeSinger(ledZeppelin) // ROBERT PLANT is awesome!!!
```

Aunque funcione no queda demasiado bien. Incluso usando una única variable que al final retornemos con todos los cambios. También podríamos cambiarlo por algo (no menos terrible) como lo siguiente.

```javascript
var awesomizeSinger = band => emphasise( awesomize( capitalizeWord( pickSinger(band))))
```

Funciona y por lo menos se reduce a una línea, pero tampoco es elegante y menos si los nombres de las funciones empiezan a crecer o si aumentamos el número de funciones. Aquí es donde entra en juego Function Composition, es una técnica por la cual aplicamos el resultado de una función a la siguiente, y el resultado de este a la siguiente y asi sucesivamente hasta completar todas las funciones en la lista. Vamos a verlo con un ejemplo, para facilitar las cosas usare la función [compose](http://ramdajs.com/docs/#compose) de Ramda. Veamos como modificamos el ejemplo anterior para usarla.

```javascript
var awesomizeSinger = band => emphasise( awesomize( capitalizeWord( pickSinger(band))))
var awesomizeSinger = R.compose(emphasise, awesomize, capitalizeWord, pickSinger)
```

Arriba la función antigua, con todos los paréntesis. Abajo nuestra nueva versión usando compose de Ramda. Compose se lee de derecha a izquierda, por lo que la última función en la línea es la primera en ejecutarse. Además compose nos devuelve una función, por lo que la podemos llamar cómo hacíamos antes para ejecutarla.

```
Result <- … <- function3 <- function2 <- function1 <- Arguments
```

Esta función (*awesomizeSinger*) envía los argumentos que se pasen a la primera función (*pickSinger*) ejecutándola, y su valor de retorno se pasa a la siguiente función (*capitaleWord*) y así con el resto de funciones, devolviéndonos el mismo resultado que antes. Merece la pena destacar que nos libramos de declarar variables (es point-free, no damos información sobre sus argumentos), de casi todos los paréntesis e incluso se construye en una única línea. Ahora es más sencillo de leer, razonar e incluso modificar. Ahora tiene sentido la analogía con el juego Lego. En este punto vemos que, al tener en las funciones argumentos de entrada y datos de retorno en la salida, los datos se van moviendo a lo largo de la aplicación siguiendo un flujo. Esto lo podríamos comparar también con una cinta de procesado donde cada máquina tiene una única finalidad y se va modificando poco a poco la materia hasta obtener un producto totalmente procesado.

Para aquellos a los que no convenza que las funciones se tengan que escribir/leer de derecha a izquierda, hay que decir que Ramda también tiene la función [pipe](http://ramdajs.com/docs/#pipe), que va de izquierda a derecha. Por no añadir complejidad seguiremos usando compose y además aprovecharemos para afianzar que se ejecuta de derecha a izquierda.

Seguro que también habréis pensado que ahora la recursividad encaja a la perfección con este modelo. Introducimos una iteración con tan solo llamar una función dentro de compose.

Es obvio que aún tenemos algunas limitaciones, como por ejemplo que las funciones solo devuelven un valor y puede que nuestras implementaciones requieran más de uno. Ya os podéis suponer como se solucionan algunos de esos casos, usando funciones a las que hayamos aplicado curry o aplicación parcial.

```javascript
var tagString = R.curry((tag, str) => `<${tag}>${str}`)
var emphasiseNTimes = times => phrase => phrase + '!'.repeat(times)

var awesomizeStrongedSinger = R.compose(emphasiseNTimes(4), awesomize, tagString('strong'), capitalizeWord, pickSinger)
var awesomizeUnderlinedSinger = R.compose(emphasiseNTimes(2), awesomize, tagString('u'), capitalizeWord, pickSinger)

awesomizeStrongedSinger(ledZeppelin) // ROBERT PLANT is awesome!!!!
awesomizeUnderlinedSinger(metallica) // JAMES HETFIELD is awesome!!
```

Si bien normalmente se suele poner en primera posición el argumento a modificar, en FP se suele poner al final para así poder configurar los argumentos previos y, aprovechando la técnica de currying, solo ejecutarse una vez que se envié el dato a tratar. Ramda, que aplica curry a todas sus funciones, sigue este patrón lo cual hace que sea una de las más elegidas dentro del mundo de la programación funcional. Veamos por ejemplo path.

```javascript
R.path(['members', 'singer'])(theWho) // Roger Daltrey
R.path(['members', 'guitarist'])(metallica) // Kirk Hammett
```

Aún nos queda que ver cómo solucionar otros problemas como por ejemplo, ¿qué pasa si queremos aplicar la función que devuelve compose a un array, capturar errores, validaciones o tenemos que hacer tareas asíncronas?


## Resumen

En esta entrada hemos visto que no necesitamos bucles para realizar tareas repetitivas, con la recursividad solo deberemos centrarnos en una condición base y en cómo realizar el cálculo. Hemos visto cómo podemos dividir las funciones para ir aplicando parcialmente los argumentos hasta que se completen y ejecutar la función, lo que nos permite realizar pequeñas configuraciones a nivel de código o incluso podríamos ir pasando una función a otras mientras le aplicamos parcialmente los argumentos. Esto nos abre un abanico de posibilidades y no tienes por qué usarlo, pero está bien saber que siempre tendrás esa opción. Y finalmente como componer funciones a través de otras más pequeñas a modo de lego, haciendo que tanto los cambios como los test se simplifiquen.
