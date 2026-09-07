# El Calificador `static`
Hasta ahora, toda sla propiedades y métodos que hemos visto pertenecen a uns **instancia** de una clase.
Esto significa que para poder usarlos, primero debemos crear un objeto con `new`.

Sin embargo, a veces necesitamos propiedades o métodos que pertenezcan a la **clase en sí misma**, no a un objeto individual.
Para esto, usamos el calificador `static`.
- **Propiedades Estáticas**: son variables que son compartidas por **todas las instancias** de una clase.
Piensa en ellas como una propiedad "global" para esa clase. Son ideales para contadores o configuraciones.
- **Métodos Estáticos**: son funciones que se pueden llamar directamente desde la clase, sin necesidad de crear un objeto.
Son perfectos para crear funciones de utilidad o "métodos de fábrica" que contruyen objetos de una menra específica.

**Punto clave**: dentro de un método estático, no se puede usar `this` para referirse a una instancia, porque el método no opera sobre un objeto específico, sino sobre la clase.

## Ejemplo práctico: Contador de Objetos Creados
Queremos llevar la cuenta de cuántos objetos de una clase especifica han sido creados en total a lo largo de nmuestro programa.

```typescript
// archivo: Coche.ts
export class Coche {
  // 1. Propiedad Estática: El contador
  // `private` para que solo pueda ser modificado desde dentro de la clase.
  // Es compartido por TODOS los objetos `Coche`.
  private static contadorDeCoches: number = 0;

  // Propiedades de instancia
  public marca: string;
  public modelo: string;

  constructor(marca: string, modelo: string) {
    this.marca = marca;
    this.modelo = modelo;

    // Cada vez que el constructor es llamado, incrementamos
    // el contador ESTÁTICO de la clase.
    Coche.contadorDeCoches++;
    console.log(`Creando ${this.marca} ${this.modelo}...`);
  }

  // 2. Método Estático: Para acceder al contador
  // Un método `public static` nos permite consultar el valor
  // del contador desde fuera de la clase.
  public static obtenerCantidadDeCochesCreados(): number {
    return Coche.contadorDeCoches;
  }

  public toString(): string {
    return `${this.marca} ${this.modelo}`;
  }
}
```

```typescript
// archivo: main.ts
import { Coche } from "./Coche";

console.log("=== CONTADOR DE INSTANCIAS CON STATIC ===");

// Podemos llamar al método estático ANTES de crear cualquier objeto.
let cantidadInicial = Coche.obtenerCantidadDeCochesCreados();
console.log(`Cantidad de coches creados al inicio: ${cantidadInicial}`); // 0

console.log("\nFabricando coches...");
const coche1 = new Coche("Toyota", "Corolla");
const coche2 = new Coche("Ford", "Mustang");
const coche3 = new Coche("Honda", "Civic");

// Después de crear las instancias, el contador estático ha sido actualizado.
let cantidadActual = Coche.obtenerCantidadDeCochesCreados();
console.log(`\nCantidad de coches creados ahora: ${cantidadActual}`); // 3

console.log("\nFabricando otro coche...");
const coche4 = new Coche("Tesla", "Model 3");

let cantidadFinal = Coche.obtenerCantidadDeCochesCreados();
console.log(`\nCantidad final de coches creados: ${cantidadFinal}`); // 4
```