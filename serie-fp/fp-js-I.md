# Programación funcional con Javascript. Parte I

Cualquiera que haya leído algo acerca de **React**, **Redux** u otras tecnologías habrá visto que se hacen referencias a conceptos como Pure Functions, Side Effects o Higher Order Functions. Mi andadura con la programación funcional comenzó por ese mismo motivo: "¿Que es una función pura? ¿Por qué debo usarlas en Redux?".

Todos estos conceptos están relacionados con la **programación funcional**. Pese a no ser un concepto nuevo, en los últimos tiempos se puede ver como hay vientos de cambio que apuntan directamente hacia este nuevo paradigma, como por ejemplo Java 8 o Redux y sus reducers con funciones puras.

La programación funcional es un **paradigma de programación declarativa**, es decir, una forma de afrontar el mismo problema con un punto de vista diferente; cuyo objetivo es centrarse en describir qué hacer, y no en cómo hacerlo.

Quizá el mejor ejemplo de programación declarativa es SQL, donde decimos qué queremos obtener con las consultas y no en qué manera hay que hacerlo. Uno de los principales beneficios es la legibilidad, al centrarse en qué pasos hay que hacer y no en cómo hacerlos, eliminando el código repetitivo que no aporta valor al producto; y que, junto con alguno de los conceptos que veremos, nos permitirá obtener un código menos propenso al error, facilitar los test unitarios y convertir en predecible su comportamiento.

En este primer post, un poco más teórico, veremos algunos conceptos y buenas prácticas que se siguen en el mundo de programación funcional. Ésta será la base sobre la que, poco a poco, iremos añadiendo piezas para poder comprender mejor este paradigma.

Para mejor legibilidad durante esta serie estaré usando las arrow functions de ES6. Para más info sobre estas funciones, [aquí](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Funciones/Arrow_functions).

## First Class Functions

En JavaScript las funciones son valores, como cualquier otro tipo de dato. Por ejemplo, pueden ser almacenadas en variables, objetos o arrays, así como ser pasadas como argumentos a otras funciones.

```javascript
var plus1 = a => a + 1;

plus1(2); // 3

var myArray = [0, 1, 'a', null, plus1];
myArray[4](2); // 3

var obj = {
    increment: plus1
};
obj.increment(2); // 3

function incrementByFn(fnArg, num) {
    return fnArg(num);
}
incrementByFn(plus1, 2); // 3
```

Vale la pena destacar en este punto que a veces, por prisa o falta de atención, se pueden ver el siguiente tipo de expresiones:

```javascript
var incNum = num => {
    return plus1(num1);
}
```
 
La función plus1 ya recibe un número como argumento, así que podríamos eliminar un nivel y dejar de esta manera la expresión:

```javascript
var incNum = plus1;
```
 
Como por ejemplo con las [Promesas de ES6](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Promise)

```javascript
getArtists
.then( artists => {
    return parseArtistsResponse(artists);
}
```

Podríamos reemplazarlo por

```javascript
getArtists
.then(parseArtistsResponse);
```
 
Otro caso común, que se ve mucho, es con las llamadas AJAX. Puede que resulte excesivo visto aquí como ejemplo, pero podemos encontrar código como el siguiente:

```javascript
var getArtists = function(parseFn) {
    return artistsAjaxCall(function(json) {
        return parseFn(json);
    });
}
```

No es complejo, pero vamos a revisarlo de nuevo ya que este punto nos será de utilidad para lo que viene por delante:

La función artistsAjaxCall recibe como argumento una función que, a su vez, recibirá un argumento (json). En este caso parseFn ya es una función que espera un argumento (json), así que podemos eliminar ese nivel y este sería el resultado:
 
```javascript
var getArtists = function(parseFn) {
    return artistsAjaxCall(parseFn);
}
```

Ahora tenemos getArtists, una función que espera una función como argumento (callback) para hacer la llamada de AJAX. De nuevo artistsAjaxCall ya es una función que recibe otra función (callback) como argumento. Así que eliminamos nivel y así quedaría el resultado:

```javascript
var getArtists = artistsAjaxCall;
```
 
Mucho más simple, ¿no? Como digo, es un punto importante para lo que vendrá más adelante.

Más info sobre First Class Functions [aquí](https://developer.mozilla.org/en-US/docs/Glossary/First-class_Function).


## Pure functions

Una función pura es aquella que en sucesivas llamadas para el mismo input, genera siempre el mismo output (es determinista). Además, no tiene efectos secundarios o side effects. Entendiendo esto último como un cambio en el sistema o una visible interacción con fuentes externas. Ejemplos de side effect serían: cambiar los valores a los argumentos de entrada o variables externas a nuestra función, leer de disco, hacer llamadas http, acceder a base de datos, hacer logging en pantalla o fichero, leer el input del usuario, obtener la fecha actual... es decir, un cambio que nos limita a no hacer nada no relacionado con la propia función. Obviamente, un programa que no haga ninguna de las acciones consideradas impuras no aporta utilidad alguna, por ello, no es que se vayan a eliminar pero sí que favoreceremos el uso de funciones puras. En programación funcional se tiende a acotar el uso de side effects a puntos determinados, de forma que buscar el origen del error puede ser mucho más sencillo y el comportamiento del programa ser más predecible.

Vamos a ver algunos ejemplos de funciones puras e impuras:
 
```javascript
// Impure
var x = 1;
var y = 2;
...
var add = () => x + y;
...
add(); // 3?


// Pure
var add = (x, y) => x + y;
...
add(1, 2); // 3
```
 
```javascript
// Impure
var getDayOfWeek = () => {
    var d = new Date();
    return d.getDate();
}
getDayOfWeek();

// Pure
var getDayOfWeek = d => {
    return d.getDate();
}
getDayOfWeek(new Date());

```
 
```javascript
//Impure
var someValue = [0];
...
var impureFn = varByRef => {
    ...
    varByRef.push(1); // Estamos alterando una variable. No podemos  saber si el valor que se ha modificado se esta
    // usando en otra parte del programa y estamos alterando el resultado esperado (causando algun efecto secundario)
    ...
}
...
impureFn(someValue);
// someValue ahora es [0, 1]!


//Pure
var someValue = [0];
...
var pureFn = varByRef => {
    ...
    var brandNewArray = varByRef.concat(1); // Concat a diferencia de push devuelve un nuevo array
    ... 
    return brandNewArray;
}
...
var returnedArray = pureFn(someValue);
// someValue sigue siendo [0]
// returnedArray es [0, 1]
```
Estos son algunos ejemplos, pero como podéis imaginar, a la larga resultan más sencillos y menos propensos a errores.

## Referential Transparency

Además, todas las funciones puras nos dan transparencia referencial, es decir, que una función siendo invocada con el mismo input puede ser intercambiada por su output directamente sin alterar el funcionamiento del programa.

```javascript
var add = (a, b) => a + b;
add(add(1, 3), add(6, 2)); // 12
add(4, add(6, 2)); // 12
add(4, 8); // 12
12
```

Por tanto, es más sencillo de interpretar tanto para los desarrolladores como para el compilador; nos da una mayor sencillez para testing; y puede ser un punto en el que podemos realizar micro optimizaciones usando la memorizacion (guardar el resultado para los mismos argumentos de entrada de una función, para que no se tenga que volver a ejecutar).

## Shared state

O como le llaman por internet: "the root of evil". Siguiendo sobre lo ya comentado en las funciones puras, el estado compartido es algo a erradicar de nuestras aplicaciones ya que hace que el comportamiento de las aplicaciones no sea determinista. Por lo tanto, provocan que sean más difíciles de comprender, de testear y solucionar posibles errores que vayan surgiendo.

De la mano con lo que vamos comentando, está el término inmutable (Inmutability). Los datos deben ser inmutables para evitar shared state, side effects, impure functions... Todo lo que hagamos será sobre copias y no sobre los datos compartidos con más partes de la aplicación.

## Higher Order Functions

Ya hemos visto que las funciones en Javascript son valores, podemos enviarlas como argumentos a otras funciones o podrían ser el valor de retorno. Pues eso mismo define las Higher Order Functions: funciones que reciben funciones como argumentos y/o devuelven funciones como valor de retorno.
 
```javascript
var ajaxCall = cb => {
    ...
    cb();
}

var thingsFactory = () => {
    ...
    return function() {
        ...
    };
}
```

## Resumen

Antes del resumen, me gustaría destacar otros puntos para siguientes posts en la serie:

* Las funciones deben devolver siempre un valor.
* “Las funciones deberían hacer una sola tarea y hacerla bien” – Robert C. Martin
* En lo posible, debemos evitar reasignar valores a variables. En matemáticas, si X es igual a 2, siempre tendrá ese valor. No se hacen reasignaciones.
* Arity es el número de argumentos que recibe una función.
* Nullary, cuando no toma ningún argumento.
* En esta entrada hemos revisado algunos conceptos que tendremos presente en el resto de la serie.

A modo de resumen, podemos decir que debemos intentar que en nuestras aplicaciones predominen las funciones puras y eviten los datos compartidos. Esto nos aporta múltiples beneficios: la predictibilidad, la sencillez para el testing o la facilidad para encontrar errores. Hemos visto que las funciones se pueden usar como cualquier otro tipo en Javascript, es decir, las podemos pasar como argumentos, almacenarlas en variables u otras estructuras, como arrays u objetos, o enviarlas como valor de retorno.

[Programación funcional con Javascript. Parte I](http://innovationlabs.softtek.co/programaci%C3%B3n-funcional-con-javascript-i)