# ¿Qué es una Clase Abstracta?
Una **clase abstracta** es una clase especial que **no puede ser instanciada directamente**.
Su propósito es servir como plantilla o molde para otras clases que la hereden.

Pueden contener: 
- **Métodos concretos** que las clases hijas pueden usar directamente, 
- **Métodos abstractos** que las hijas deben implementar obligatoriamente y
- **Atributos** que todas las clases hijas compartirán.

# ¿Por qué usar Clases Abstractas?
Imaginemos que queremos crear un sistema para diferentes figuras geometricas y todas tienen caracteristicas en común.
Pero cada figura calcula su área y perímetro de manera diferente. Las clases abstractas nos permiten definir la estructura.

# Sintáxis de Clases Abstractas en TypeScript
```typescript
// archivo: figura.ts
export abstract class Figura {
  protected color: string;  // Atributo común a todas las figuras

  constructor(color: string) {
    this.color = color;
  }

  // Método concreto (ya implementado) que todas las figuras pueden usar
  public getColor(): string {
    return this.color;
  }

  public setColor(color: string): void {
    this.color = color;
  }

  // Método concreto que usa los métodos abstractos
  public mostrarInformacion(): void {
    console.log(this.toString());
    console.log(`Color: ${this.color}`);
    console.log(`Área: ${this.calcularArea()}`);
    console.log(`Perímetro: ${this.calcularPerimetro()}`);
  }

  // Métodos abstractos (solo la firma) que DEBEN ser implementados por las clases hijas
  public abstract calcularArea(): number;
  public abstract calcularPerimetro(): number;
  public abstract toString(): string;
}
```

La clase se declara con `abstract class`
Los métodos abstractos se declaran con `abstract` y no tienen implementación.
Los métodos concretos si tienen implementacion y pueden usar métodos abstractos.

# Implementando clases hijas
Ahora creamos clases hijas que hereden de <Figura>:

```typescript
// archivo: circulo.ts
import { Figura } from "./figura";

export class Circulo extends Figura {
  private radio: number;

  constructor(color: string, radio: number) {
    super(color);  // Llamamos al constructor de la clase padre
    this.radio = radio;
  }

  // OBLIGATORIO: implementar todos los métodos abstractos
  public calcularArea(): number {
    return Math.PI * this.radio * this.radio;
  }

  public calcularPerimetro(): number {
    return 2 * Math.PI * this.radio;
  }

  public toString(): string {
    return `Círculo de radio ${this.radio}`;
  }

  // Métodos específicos del círculo
  public getRadio(): number { return this.radio; }
  public setRadio(radio: number): void { this.radio = radio; }

  public getDiametro(): number {
    return this.radio * 2;
  }
}
```

```typescript
// archivo: rectangulo.ts
import { Figura } from "./figura";

export class Rectangulo extends Figura {
  private ancho: number;
  private alto: number;

  constructor(color: string, ancho: number, alto: number) {
    super(color);
    this.ancho = ancho;
    this.alto = alto;
  }

  // OBLIGATORIO: implementar todos los métodos abstractos
  public calcularArea(): number {
    return this.ancho * this.alto;
  }

  public calcularPerimetro(): number {
    return 2 * (this.ancho + this.alto);
  }

  public toString(): string {
    return `Rectángulo de ${this.ancho} x ${this.alto}`;
  }

  // Métodos específicos del rectángulo
  public getAncho(): number { return this.ancho; }
  public setAncho(ancho: number): void { this.ancho = ancho; }
  public getAlto(): number { return this.alto; }
  public setAlto(alto: number): void { this.alto = alto; }

  public esCuadrado(): boolean {
    return this.ancho === this.alto;
  }
}
```

# Uso de Clases Abstractas y Polimorfismo
```typescript
// archivo: main.ts
import { Figura } from "./figura";
import { Circulo } from "./circulo";
import { Rectangulo } from "./rectangulo";
import { Triangulo } from "./triangulo";

// ❌ Esto NO se puede hacer - no se puede instanciar una clase abstracta
// const figura = new Figura("rojo");  // Error de compilación

// ✅ Esto SÍ se puede hacer - instanciar las clases hijas
const circulo = new Circulo("azul", 5);
const rectangulo = new Rectangulo("verde", 4, 6);
const triangulo = new Triangulo("rojo", 3, 4, 5);

// ✅ Polimorfismo con clases abstractas
const figuras: Figura[] = [circulo, rectangulo, triangulo];

console.log("=== MOSTRANDO TODAS LAS FIGURAS ===");
figuras.forEach((figura, index) => {
  console.log(`\nFigura ${index + 1}:`);
  figura.mostrarInformacion();  // Cada figura se comporta diferente
});

// ✅ Podemos usar métodos específicos si hacemos casting
console.log("\n=== MÉTODOS ESPECÍFICOS ===");
if (circulo instanceof Circulo) {
  console.log(`El círculo tiene un diámetro de ${circulo.getDiametro()}`);
}

if (rectangulo instanceof Rectangulo) {
  console.log(`¿Es cuadrado? ${rectangulo.esCuadrado()}`);
}

if (triangulo instanceof Triangulo) {
  console.log(`¿Es equilátero? ${triangulo.esEquilatero()}`);
}
```