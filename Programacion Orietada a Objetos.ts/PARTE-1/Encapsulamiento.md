# Encapsulamiento
- Es una práctica recomendable de programación *encapsular* (ocultar) los atributos de la clase 
y los métodos que no queremos exponer a terceros utilizando el calificado `private`.
_NOTA: Exisite otros calificadores para modificar el acceso a un atributo: `public` y `protected`_

Para usar encapsulamiento de atributos debemos:
- Declarar **todos** los atributos como `private`.
- Para cada atributo, en caso de que se necesite, debemos crear un método `get<nombreAtributo>` y un método `set<nombreAtributo> `.
_`get` se usa para obtener el valor del atributo, mientras que `set` se utiliza para modificar el valor del atributo._

- Supongamos que tenemos que escribir la clase Persona en Typescript utilizando el encapsulamiento de los atributos. Para esto, declaramos los atributos como `private` , y luego escribimos los setters y getters para cada atributo:
``` typescript
export class Persona {
  private nombre: string;

  constructor(nombre: string = "") {
    this.nombre = nombre;
  }

  public getNombre(): string {
    return this.nombre;
  }

  public setNombre(nombre: string): void {
    this.nombre = nombre;
  }
}
```

_En UML tambien podemos especificar la visibilidad de cada atributo:_

Calificador	|   Visibilidad
----------------------------
-	        |   privado
+	        |   público
#	        |   protegido
~	        |   paquete