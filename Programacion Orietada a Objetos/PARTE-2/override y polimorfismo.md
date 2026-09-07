# ¿Qué es OVERRIDE?
Cuando una clse hija hereda de una clase padre, puede redefinir o sobreescribir los métodos de la clase padre.

# ¿Qué es POLIMORFISMO?
La capacidad de objetos de diferentes clases para responder a la misma operacion o mensjae de manera diferente.
**Esto permite que el codigo sea mas flexible y se reutilice de manera mas efectiva.**

# La clase Object en TypeScript
Todos los objetos heredan implicitqamente la clase <Object>. 
Por defecto cuando llamamos a `toString()` en cualquier objeto, obtenemos algo como [object Object], que no es muy útil, pero podemos **sobreescribir**.

# `toString()` por defecto:
```typescript
// archivo: vehiculo-basico.ts
export class VehiculoBasico {
  private marca: string;
  private modelo: string;

  constructor(marca: string, modelo: string) {
    this.marca = marca;
    this.modelo = modelo;
  }

  public getMarca(): string { return this.marca; }
  public getModelo(): string { return this.modelo; }

  // NO sobrescribimos toString() - usará el de Object
}
```

# `toString()` sobreescrito:
```typescript
// archivo: vehiculo-mejorado.ts
export class VehiculoMejorado {
  private marca: string;
  private modelo: string;

  constructor(marca: string, modelo: string) {
    this.marca = marca;
    this.modelo = modelo;
  }

  // SÍ sobrescribimos toString() - nuestra propia implementación
  public toString(): string {
    return `${this.marca} ${this.modelo}`;
  }

  public getMarca(): string { return this.marca; }
  public getModelo(): string { return this.modelo; }
}
```

# Ejemplo de `toString()` mas complejo y con diferentes implementaciones:
```typescript
// archivo: persona-toString.ts
export class Persona {
  private nombre: string;
  private edad: number;
  private email: string;

  constructor(nombre: string, edad: number, email: string) {
    this.nombre = nombre;
    this.edad = edad;
    this.email = email;
  }

  // Sobrescribimos toString() para mostrar información útil
  public toString(): string {
    return `${this.nombre} (${this.edad} años) - ${this.email}`;
  }

  // Método adicional para diferentes formatos
  public toStringCorto(): string {
    return `${this.nombre}, ${this.edad} años`;
  }

  public toStringFormal(): string {
    return `Sr./Sra. ${this.nombre}`;
  }

  public getNombre(): string { return this.nombre; }
  public getEdad(): number { return this.edad; }
  public getEmail(): string { return this.email; }
}
```

# Ejemplo práctico de Polimorfismo
Supongamos que tenemos una clase <Impresora> que tiene un metodo llamado `imprimirEnPantalla(objeto: any)`. Lo que hace es recibir cualquier objeto e internamente llamar al `toString()`.
Luego crearemos diferentes clases como <Bicicleta> ,<Auto> , <Persona> , etc. Cada una implementará su propio método `toString()` de manera diferente.

```typescript
// archivo: impresora.ts
export class Impresora {
  public imprimirEnPantalla(objeto: any): void {
    console.log(objeto.toString());
  }
  // Nota: objeto.toString() es equivalente a objeto.toString()
  // TypeScript automáticamente llamará al método toString() del objeto
}
```

```typescript
// archivo: bicicleta.ts
export class Bicicleta {
  private marca: string;
  private modelo: string;

  constructor(marca: string = "", modelo: string = "") {
    this.marca = marca;
    this.modelo = modelo;
  }

  // Sobrescribimos el método toString() para nuestra clase
  public toString(): string {
    return `Soy una bicicleta ${this.marca} modelo ${this.modelo}`;
  }

  public getMarca(): string { return this.marca; }
  public setMarca(marca: string): void { this.marca = marca; }
  public getModelo(): string { return this.modelo; }
  public setModelo(modelo: string): void { this.modelo = modelo; }
}
```

```typescript
// archivo: auto.ts
export class Auto {
  private marca: string;
  private año: number;

  constructor(marca: string = "", año: number = 0) {
    this.marca = marca;
    this.año = año;
  }

  // Cada clase tiene su propia implementación de toString()
  public toString(): string {
    return `Auto ${this.marca} del año ${this.año}`;
  }

  public getMarca(): string { return this.marca; }
  public setMarca(marca: string): void { this.marca = marca; }
  public getAño(): number { return this.año; }
  public setAño(año: number): void { this.año = año; }
}
```

### ¿Qué imprime esto?
```typescript
// archivo: auto.ts
export class Auto {
  private marca: string;
  private año: number;

  constructor(marca: string = "", año: number = 0) {
    this.marca = marca;
    this.año = año;
  }

  // Cada clase tiene su propia implementación de toString()
  public toString(): string {
    return `Auto ${this.marca} del año ${this.año}`;
  }

  public getMarca(): string { return this.marca; }
  public setMarca(marca: string): void { this.marca = marca; }
  public getAño(): number { return this.año; }
  public setAño(año: number): void { this.año = año; }
}
```

#### RESPUESTA
Soy una bicicleta Trek modelo Mountain
Auto Toyota del año 2020
[object Object]