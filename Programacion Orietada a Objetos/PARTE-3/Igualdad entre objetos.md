# Igualdad entre Objetos
- **Igualdad por Referencia (Identidad)**: es lo que JavaScript/TypeScript comprueba por defecto con el operador ===.
Dos variables de objeto son iguales por referencia solo si **apuntan  exactamente al mismo objeto en memoria**. Es como tener dos llaves que abren la misma y unica puerta.
- **Igualdad por Valor (Equivalencia)**: ocurre cuando objetos, aunque sean instancias diferentes en la memoria, tienen los **mismos valores ebn sus propiedades**.
Es como tener dos llaves que abren dos puertas diferentes de dos casas que son idénticas por dentro. Por defecto, TypeScript no comprueba esto; debemos escribir nuestra propia lógica.

## Ejemplo práctico: Comparando Puntos
```typescript
// archivo: Punto.ts
export class Punto {
  private x: number;
  private y: number;

  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }

  // Método para comprobar la igualdad por VALOR
  public esIgualA(otroPunto: Punto): boolean {
    return this.x === otroPunto.x && this.y === otroPunto.y;
  }

  public toString(): string {
    return `Punto(x: ${this.x}, y: ${this.y})`;
  }
}
```

```typescript
// archivo: main.ts
import { Punto } from "./Punto";

console.log("\n=== IGUALDAD DE OBJETOS ===");

const p1 = new Punto(10, 20);
const p2 = new Punto(10, 20); // Mismos valores, pero objeto diferente
const p3 = p1;                 // Misma referencia, mismo objeto

console.log(`p1: ${p1}`);
console.log(`p2: ${p2}`);
console.log(`p3: ${p3}`);

console.log("\n--- Comprobando igualdad por Referencia (===) ---");
console.log(`¿p1 === p2? ${p1 === p2}`); // false -> Son dos objetos distintos en memoria.
console.log(`¿p1 === p3? ${p1 === p3}`); // true  -> Apuntan al mismo objeto en memoria.

console.log("\n--- Comprobando igualdad por Valor (lógica propia) ---");
console.log(`¿p1 es igual en valor a p2? ${p1.esIgualA(p2)}`); // true -> Sus propiedades internas son iguales.
```

# Copia Superficial vs. Copia Profunda
- **Copia Superficial (Shallow Copy)**: Se crea un nuevo objeto, y las propiedades de nivel superior del objeto original se copian en él.
**Problema**: Si alguna de esas propiedades es a su vez un objeto o un array, lo que se copia es la referencia a ese objeto/array anidado, no ina copia del mismo.
- **Copia Profunda (Deep Copy)**: Se crea un nuevo objeto, y se recorren **recursivamente** todas sus propiedades. Si se encuentra un objeto o array anidado, se crea tambien una copia profunda de él. Esto garantiza que el objeto copiado sea completamente independiente del original.

## Ejemplo Práctico: Copiando un Pedido

```typescript
// archivo: Cliente.ts
export class Cliente {
  public nombre: string;
  constructor(nombre: string) { this.nombre = nombre; }
}

// archivo: Pedido.ts
import { Cliente } from "./Cliente";

export class Pedido {
  public id: number;
  public cliente: Cliente; // Propiedad anidada (un objeto)

  constructor(id: number, cliente: Cliente) {
    this.id = id;
    this.cliente = cliente;
  }

  public toString(): string {
    return `Pedido ID: ${this.id}, Cliente: ${this.cliente.nombre}`;
  }
}
```

```typescript
// archivo: main.ts
import { Cliente } from "./Cliente";
import { Pedido } from "./Pedido";

console.log("\n=== COPIA SUPERFICIAL VS. PROFUNDA ===");

const clienteOriginal = new Cliente("Ana");
const pedidoOriginal = new Pedido(101, clienteOriginal);

console.log("Estado Original:");
console.log(pedidoOriginal.toString());

// --- 1. Copia Superficial (Shallow Copy) ---
// Usamos el operador de propagación (...) para una copia superficial.
const copiaSuperficial = { ...pedidoOriginal };

console.log("\nRealizando cambios en la Copia Superficial...");
// Cambiamos el nombre del cliente a través de la copia.
copiaSuperficial.cliente.nombre = "Beatriz";

console.log(`Copia Superficial: ${copiaSuperficial.id}, Cliente: ${copiaSuperficial.cliente.nombre}`);
console.log(`Pedido Original: ${pedidoOriginal.toString()}`); // ¡¡¡El original también cambió!!!
// Esto sucede porque ambos objetos (original y copia) comparten la MISMA referencia al objeto Cliente.

// --- 2. Copia Profunda (Deep Copy) ---
// Reseteamos el nombre para una demostración limpia.
clienteOriginal.nombre = "Ana";

// Usamos el "truco de JSON" para una copia profunda simple.
// NOTA: Este método tiene limitaciones (pierde funciones, fechas se convierten en strings, etc.).
// Para casos complejos, se recomienda usar una librería como Lodash.
const copiaProfunda = JSON.parse(JSON.stringify(pedidoOriginal));

console.log("\nRealizando cambios en la Copia Profunda...");
// Cambiamos el nombre del cliente a través de la copia profunda.
copiaProfunda.cliente.nombre = "Carla";

console.log(`Copia Profunda: Pedido ID: ${copiaProfunda.id}, Cliente: ${copiaProfunda.cliente.nombre}`);
console.log(`Pedido Original: ${pedidoOriginal.toString()}`); // ¡El original NO cambió!
// Esto funciona porque la copia profunda creó un objeto Pedido nuevo Y un objeto Cliente nuevo.
```