# Aprovechando el Compilador: Pruebas de Tipos (Type-Aware Testing)
Antes de escribir una sola prueba en Vitest, ya tienes la herramienta de testing más fundamental y de más alto rendimiento a tu disposición: **el propio compilador de TypeScript**. Las "pruebas de tipos" no son pruebas en el sentido tradicional; son la garantía de que tu código se adhiere a los contratos que has definido, eliminando una categoría entera de errores antes de que el programa se ejecute.

# Los Dos Mundos del Testing: Compilación vs. Ejecución
Es crucial no confundir estos dos tipos de validación. Son dos capas de seguridad complementarias, no excluyentes.

1. **Mundo de Tipos (Compile-Time): El Plano Arquitectónico**
    - **Herramienta**: El compilador de TypeScript(`tsc`).
    - **¿Qué verifica?**:La **corección estructural** de tu código.¿Estás pasando un `string` a una función que espera un `number`? ¿Estás accediendo a una propiedad que no existe? ¿Has olvidado un `case` en un `switch` exhaustivo? 
    - **¿Cuándo ocurre?**: Antes de que se genere el JavaScript, durante el desarrollo o en un paso de CI. El código nunca llega a ejecutarse si hay un error de tipo.
2. **Mundo de Ejecución (Runtime): El Edificio en Funcionamiento**
    - **Herramienta**: Un framework de testing como Vitest o Jest.
    - **¿Qué verifica?**: El **comportamiento lógico** de tu código. Si llamas a `sumar(2, 2)`, ¿devuelve `4` o devuelve `5`? ¿Al hacer clic en un botón, se actualiza el estado correctamente?
    - **¿Cuándo ocurre?**: Después de que el código ha sido compilado a JavaScript y se está ejecutando en un entorno (como Node.js).

# La Práctica: `tsc --noEmit` en Integración Continua (CI):
La forma estandar de ejecutar tus "pruebas de tipos" es con este comando:

```bash
tsc --noEmit

```

- `tsc`: Invoca al compilador de TypeScript.
- `--noEmit`: Le dice al compilador: "Analiza todo el proyecto y busca errores de tipo, pero **no generes ningun archivo JavaScript**".

Este comando es increíblemente rápido y es la primera barrera de calidad que deberías poner en tu pipeline de CI. Si `tsc --noEmit` falla, significa que hay un error de tipo, y no tiene sentido gastar tiempo y recursos en ejecutar las pruebas de Vitest, que son más lentas.

| Característica | Pruebas de Tipos (Compile-time) | Pruebas de Ejecución (Runtime) |
|---|---|---|
| **Herramienta** | `tsc` | `Vitest`, `Jest`, etc. |
| **Propósito** | Validar la corrección estructural y de tipos. | Validar la lógica y el comportamiento del código. |
| **Ejemplo de error** | `const x: number = "hola";` | `function sumar(a, b) { return a - b; }` |
| **Cuándo se ejecuta** | Antes de la ejecución. | Durante la ejecución del código. |

En resumen: Piensa en `tsc --noEmit` como la prueba unitaria más rápida y barata que puedes ejecutar. Si pasa, ganas confianza en que tu código es estructuralmente sólido, y entonces puedes pasar a Vitest para verificar que también se comporta como esperas.