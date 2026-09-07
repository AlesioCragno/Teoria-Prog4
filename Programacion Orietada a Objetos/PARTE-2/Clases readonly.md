# ¿Qué significa readonly?
En TypeScript, podemos modificar `readonly` para crear propiedades que **solo pueden ser asignadas durante la inicializacion o en el constructor**.
Una vez asignadas, no pueden ser modificadas.

Esto es muy util para crear propiedades que deben mantenerse constantes durante toda la vida del objeto, como identificadores únicos, fechas de creación, etc.

# ¿Cuándo usar readonly?
Usamos `readonly` cuando: 
- Queremos asegurar que ciertos valores no cambien despues de la creación del objeto.
- Necesitamos identificadores unicos (ID´s, DNI's, códigos, etc).
- Queremos fechas de creación/registro inmutables.
- Tenemos configuraciones que no deben modificarse.
- Queremos arrays o objetos que no deben ser reasignados.

# Sintaxis de `readonly`:

```typescript
// archivo: producto.ts
export class Producto {
  public readonly id: string;           // ID único, no cambia nunca
  public readonly fechaCreacion: Date;  // Fecha de creación, no cambia nunca
  public readonly categoria: string;    // Categoría, no cambia una vez asignada
  
  private nombre: string;               // Nombre sí puede cambiar
  private precio: number;               // Precio sí puede cambiar

  constructor(categoria: string, nombre: string, precio: number) {
    this.id = this.generarId();         // Solo se puede asignar aquí
    this.fechaCreacion = new Date();    // Solo se puede asignar aquí
    this.categoria = categoria;         // Solo se puede asignar aquí
    this.nombre = nombre;
    this.precio = precio;
  }

  private generarId(): string {
    return `PROD_${Math.random().toString(36).substring(2, 15).toUpperCase()}`;
  }

  // Métodos getter para readonly properties - NO necesitan setter
  public getId(): string { return this.id; }
  public getFechaCreacion(): Date { return this.fechaCreacion; }
  public getCategoria(): string { return this.categoria; }

  // Métodos getter y setter para propiedades modificables
  public getNombre(): string { return this.nombre; }
  public setNombre(nombre: string): void { this.nombre = nombre; }
  
  public getPrecio(): number { return this.precio; }
  public setPrecio(precio: number): void { 
    if (precio > 0) {
      this.precio = precio;
    }
  }

  public toString(): string {
    return `${this.nombre} (ID: ${this.id}) - ${this.precio} - Categoría: ${this.categoria}`;
  }
}
```

# Arrays `readonly`:

```typescript
// archivo: configuracion.ts
export class Configuracion {
  // Array readonly - no se puede reasignar, pero su contenido puede leerse
  private readonly configuracionesValidas: readonly string[] = [
    "desarrollo",
    "produccion", 
    "testing",
    "staging"
  ];
  
  // Configuración actual sí puede cambiar
  private configuracionActual: string;

  constructor() {
    this.configuracionActual = "desarrollo"; // configuración por defecto
  }

  public esConfiguracionValida(config: string): boolean {
    return this.configuracionesValidas.includes(config);
  }

  public setConfiguracion(config: string): boolean {
    if (this.esConfiguracionValida(config)) {
      this.configuracionActual = config;
      console.log(`Configuración cambiada a: ${config}`);
      return true;
    }
    console.log(`Error: '${config}' no es una configuración válida`);
    return false;
  }

  public getConfiguracion(): string {
    return this.configuracionActual;
  }

  // Devolver copia readonly del array para evitar modificaciones externas
  public getConfiguracionesValidas(): readonly string[] {
    return this.configuracionesValidas;
  }

  public mostrarConfiguracionesDisponibles(): void {
    console.log("Configuraciones disponibles:");
    this.configuracionesValidas.forEach((config, index) => {
      const marca = config === this.configuracionActual ? " ← ACTUAL" : "";
      console.log(`  ${index + 1}. ${config}${marca}`);
    });
  }
}
```

## Ejemplo práctico: Sistema de Usuarios con readonly

```typescript
// archivo: usuario.ts
export class Usuario {
  public readonly id: string;
  public readonly fechaRegistro: Date;
  public readonly tipoUsuario: string;
  private readonly rolesPermitidos: readonly string[] = [
    "invitado", 
    "usuario", 
    "moderador", 
    "admin"
  ];
  
  private nombre: string;
  private email: string;
  private rolActual: string;
  private activo: boolean;

  constructor(nombre: string, email: string, tipoUsuario: string = "usuario") {
    this.id = this.generarId();
    this.fechaRegistro = new Date();
    this.tipoUsuario = tipoUsuario;
    this.nombre = nombre;
    this.email = email;
    this.rolActual = "invitado"; // rol inicial
    this.activo = true;
  }

  private generarId(): string {
    const timestamp = Date.now().toString(36);
    const random = Math.random().toString(36).substring(2, 8);
    return `USER_${timestamp}_${random}`.toUpperCase();
  }

  public cambiarRol(nuevoRol: string): boolean {
    if (!this.rolesPermitidos.includes(nuevoRol)) {
      console.log(`Error: '${nuevoRol}' no es un rol válido`);
      return false;
    }

    if (!this.activo) {
      console.log("Error: No se puede cambiar el rol de un usuario inactivo");
      return false;
    }

    const rolAnterior = this.rolActual;
    this.rolActual = nuevoRol;
    console.log(`Rol de ${this.nombre} cambiado de '${rolAnterior}' a '${nuevoRol}'`);
    return true;
  }

  public desactivar(): void {
    this.activo = false;
    this.rolActual = "invitado"; // Al desactivar, vuelve a invitado
    console.log(`Usuario ${this.nombre} desactivado`);
  }

  public reactivar(): void {
    this.activo = true;
    console.log(`Usuario ${this.nombre} reactivado`);
  }

  public puedeAccederA(recurso: string): boolean {
    if (!this.activo) return false;

    switch (this.rolActual) {
      case "invitado":
        return recurso === "pagina_publica";
      case "usuario":
        return ["pagina_publica", "perfil", "configuracion"].includes(recurso);
      case "moderador":
        return !["panel_admin", "configuracion_sistema"].includes(recurso);
      case "admin":
        return true;
      default:
        return false;
    }
  }

  // Getters para propiedades readonly
  public getId(): string { return this.id; }
  public getFechaRegistro(): Date { return this.fechaRegistro; }
  public getTipoUsuario(): string { return this.tipoUsuario; }
  
  // Getters y setters para propiedades modificables
  public getNombre(): string { return this.nombre; }
  public setNombre(nombre: string): void { this.nombre = nombre; }
  
  public getEmail(): string { return this.email; }
  public setEmail(email: string): void { this.email = email; }
  
  public getRolActual(): string { return this.rolActual; }
  public isActivo(): boolean { return this.activo; }

  public toString(): string {
    const estado = this.activo ? "Activo" : "Inactivo";
    return `${this.nombre} (${this.rolActual}) - ${estado} - Registrado: ${this.fechaRegistro.toLocaleDateString()}`;
  }
}
```

```typescript
// archivo: main.ts - Ejemplo de uso de readonly
import { Usuario } from "./usuario";
import { Producto } from "./producto";
import { Configuracion } from "./configuracion";

console.log("=== EJEMPLO DE PROPIEDADES READONLY ===");

// Crear productos
const producto1 = new Producto("Electrónicos", "Laptop Gaming", 1200);
const producto2 = new Producto("Libros", "TypeScript Avanzado", 45);

console.log("\nProductos creados:");
console.log(producto1.toString());
console.log(producto2.toString());

// Las propiedades readonly no se pueden modificar
console.log(`\nID del producto 1: ${producto1.getId()}`);
// producto1.id = "NUEVO_ID"; // ❌ Error: Cannot assign to 'id' because it is a read-only property

// Pero las propiedades normales sí
producto1.setNombre("Super Laptop Gaming");
producto1.setPrecio(1100);
console.log(`Producto modificado: ${producto1.toString()}`);

// Configuración con arrays readonly
console.log("\n=== CONFIGURACIÓN DEL SISTEMA ===");
const config = new Configuracion();
config.mostrarConfiguracionesDisponibles();

config.setConfiguracion("produccion");
config.setConfiguracion("invalida"); // Esto fallará
config.mostrarConfiguracionesDisponibles();

// Usuarios con propiedades readonly
console.log("\n=== SISTEMA DE USUARIOS ===");
const usuario1 = new Usuario("Juan Pérez", "juan@email.com", "usuario");
const usuario2 = new Usuario("Ana Admin", "ana@email.com", "admin");

console.log(usuario1.toString());
console.log(usuario2.toString());

// IDs únicos generados automáticamente
console.log(`\nID único de Juan: ${usuario1.getId()}`);
console.log(`ID único de Ana: ${usuario2.getId()}`);

// Cambiar roles (permitido)
usuario1.cambiarRol("moderador");
usuario1.cambiarRol("admin");

// Intentar cambiar propiedades readonly (no permitido)
// usuario1.id = "NUEVO_ID";           // ❌ Error de compilación
// usuario1.fechaRegistro = new Date(); // ❌ Error de compilación

console.log("\nEstado final:");
console.log(usuario1.toString());
console.log(usuario2.toString());
```