# ¿Qué son los Arrays?
Los arrays nos permiten almacenar múltiples elementos del mismo tipo en una sola variable.
**En TypeScript los arrays son dinámicos, es decir pueden cambiar de tamaño durante la ejecución.**

# ¿Cómo se declara un Array?
`tipoDeDato[] nombreDelArray = [elemento1, elemento2, ...];`

### Ejemplos 
```typescript
// Array de números
let numeros: number[] = [10, 20, 30, 40, 50];

// Array de strings
let finDeSemana: string[] = ["sábado", "domingo"];

// Array de booleanos
let respuestas: boolean[] = [true, false, true, false];

// También se puede usar la sintaxis genérica (menos común)
let valores: Array<number> = [1, 2, 3, 4];
```

# ¿Cómo se accede a los elementos?
**Recordar que el índice del primer elemento de un Array es cero.**

```typescript
let numeros: number[] = [10, 20, 30, 40, 50];
let valor: number = numeros[1];  // valor será 20
console.log(valor);  // Imprime: 20

// Otros ejemplos
console.log(numeros[0]);  // Imprime: 10 (primer elemento)
console.log(numeros[4]);  // Imprime: 50 (último elemento)
```

## Arrays dinámicos
Los arreglos en TypeScript son dinamicos, lo que significa que podemos: agregar elementos, remover elementos, cambiar su tamaño durante su ejecucion.

```typescript
// Crear un array vacío
let frutas: string[] = [];

// Agregar elementos al final
frutas.push("manzana");
frutas.push("banana"); 
frutas.push("naranja");

console.log(frutas);  // ["manzana", "banana", "naranja"]
console.log(frutas.length);  // 3

// Remover el último elemento
let ultimaFruta = frutas.pop();
console.log(ultimaFruta);  // "naranja"
console.log(frutas);  // ["manzana", "banana"]

// Agregar elemento al principio
frutas.unshift("uva");
console.log(frutas);  // ["uva", "manzana", "banana"]

// Remover el primer elemento
let primeraFruta = frutas.shift();
console.log(primeraFruta);  // "uva"
console.log(frutas);  // ["manzana", "banana"]
```

## Arrays de Objetos

```typescript
// archivo: main.ts
import { Persona } from "./persona";

// Crear un array de personas
let personas: Persona[] = [];

// Agregar personas al array
personas.push(new Persona("Juan", new Date(1990, 5, 15)));
personas.push(new Persona("María", new Date(1985, 10, 22)));
personas.push(new Persona("Pedro", new Date(1992, 2, 8)));

console.log(`Tenemos ${personas.length} personas`);

// Recorrer el array e imprimir cada persona
for (let i = 0; i < personas.length; i++) {
  console.log(`${i + 1}. ${personas[i].toString()} - Edad: ${personas[i].getEdad()}`);
}
```

# Métodos utiles de Arrays
## length, includes, indexOf, slice, splice

```typescript
let numeros: number[] = [1, 5, 3, 8, 2, 7, 4];

// length: obtener la cantidad de elementos
console.log(numeros.length);  // 7

// includes: verificar si un elemento existe
console.log(numeros.includes(5));  // true
console.log(numeros.includes(10)); // false

// indexOf: encontrar la posición de un elemento
console.log(numeros.indexOf(8));   // 3
console.log(numeros.indexOf(10));  // -1 (no encontrado)

// slice: obtener una porción del array (sin modificar el original)
let porcion = numeros.slice(2, 5);  // del índice 2 al 4
console.log(porcion);  // [3, 8, 2]
console.log(numeros);  // [1, 5, 3, 8, 2, 7, 4] (original sin cambios)

// splice: remover elementos del array (modifica el original)
let removidos = numeros.splice(2, 2);  // desde índice 2, remover 2 elementos
console.log(removidos);  // [3, 8]
console.log(numeros);    // [1, 5, 2, 7, 4] (original modificado)
```

# Métodos avanzados de Arrays
## map, filter, find, some, every, reduce, sort

```typescript
let numeros: number[] = [1, 5, 3, 8, 2, 7];

// map: transforma cada elemento y devuelve un nuevo array
let numerosDobles = numeros.map(n => n * 2);
console.log(numerosDobles);  // [2, 10, 6, 16, 4, 14]
console.log(numeros);        // [1, 5, 3, 8, 2, 7] (original sin cambios)

// filter: filtra elementos que cumplan una condición
let numerosPares = numeros.filter(n => n % 2 === 0);
console.log(numerosPares);  // [8, 2]

// find: encuentra el primer elemento que cumple una condición
let numeroMayor5 = numeros.find(n => n > 5);
console.log(numeroMayor5);  // 8

// some: verifica si al menos un elemento cumple la condición
let hayMayorA10 = numeros.some(n => n > 10);
console.log(hayMayorA10);  // false

// every: verifica si todos los elementos cumplen la condición
let todosPositivos = numeros.every(n => n > 0);
console.log(todosPositivos);  // true

// reduce: reduce el array a un solo valor
let suma = numeros.reduce((acumulador, actual) => acumulador + actual, 0);
console.log(suma);  // 26

// sort: ordena el array (modifica el original)
let numerosOrdenados = [...numeros].sort((a, b) => a - b);  // copia y ordena
console.log(numerosOrdenados);  // [1, 2, 3, 5, 7, 8]
console.log(numeros);           // [1, 5, 3, 8, 2, 7] (original sin cambios)
```