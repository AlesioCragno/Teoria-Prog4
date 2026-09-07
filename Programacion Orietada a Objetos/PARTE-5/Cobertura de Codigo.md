# Cobertura de Código: El Mapa de Tus Pruebas
La cobertura de código (*code coverage*) es una métrica que te indica qué porcentaje de tu código fuente ha sido ejecutado mientras corrían tus pruebas.

Piensa en ella no como una medida de la *calidad* de tus pruebas, sino como un **mapa de las zonas que no has probado**. La pregunta que responde no es "¿mi código funciona?", sino "**¿qué partes de mi código he probado y cuáles he ignorado?**".

# ¿Qué Mide Exactamente?
Las herramientas de cobertura, como la que viene integrada en Vitest, suelen reportar varias métricas:

| Métrica | Descripción | Ejemplo de Falla |
|---|---|---|
| **Líneas** | Porcentaje de líneas de código ejecutables que fueron ejecutadas. | Una línea dentro de un `if` que nunca se cumplió. |
| **Funciones** | Porcentaje de funciones que fueron llamadas al menos una vez. | Una función de utilidad que ninguna prueba invocó. |
| **Ramas** | Porcentaje de "caminos" lógicos que se tomaron. | Probar solo el `if` de una condición, pero nunca el `else`. |

La métrica de **ramas** es a menudo la más reveladora. Puedes tener un 100% de cobertura de líneas en una función, pero si solo probaste un camino de una bifurcación `if/else` compleja, tu cobertura de ramas será baja, indicando que hay escenarios lógicos que no has verificado.

**El Peligro de Perseguir el 100%**
Es tentador establecer umbrales muy altos (90%-100%), pero esto a menudo es contraproducente. Perseguir un número puede llevar a los equipos a escribir pruebas de baja calidad que simplemente "tocan" el código para aumentar la métrica, pero no verifican su comportamiento de forma significativa.

Esto se conoce como la **Ley de Goodhart**: "*Cuando una medida se convierte en un objetivo, deja de ser una buena medida*".

**La Estrategia Razonable:**
1. **Establece un umbral base y razonable**: Un punto de partida común es establecer un umbral de líneas entre el **60% y el 80%**. Esto actúa como una red de seguridad para evitar que se añadan grandes cantidades de código sin probar.
2. **Enfócate en la Calidad, no en la Cantidad**: Usa el reporte de cobertura no para celebrar un número alto, sino para **identificar las partes críticas de tu aplicación que tienen baja cobertura**.
3. **Prioriza**: ¿El módulo de pagos tiene un 40% de cobertura? ¡Es una señal de alerta! ¿Un script de utilidad simple tiene un 70%? Quizás no sea tan urgente.

## Ejemplo Práctico: Generando un Reporte de Cobertura con Vitest
### Paso 1: Configurar Vitest para la Cobertura
**En tu archivo `vitest.config.ts` (o  `vite.config.ts` ), añade la sección `test.coverage`:**

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    // ...otras configuraciones de test...
    coverage: {
      // El proveedor de cobertura. 'v8' es rápido y nativo.
      provider: 'v8', 
      // Reporteros para generar el informe. 'text' en consola, 'html' para un informe visual.
      reporter: ['text', 'html'],
      // Opcional: Establecer umbrales. El build fallará si no se alcanzan.
      thresholds: {
        lines: 60,
        branches: 60,
        functions: 60,
        statements: 60,
      },
    },
  },
});
```
**En tu `package.json`, añade un script para la cobertura:**
```json
{
  "scripts": {
    "test": "vitest",
    "coverage": "vitest run --coverage"
  }
}
```

- `vitest run`: Ejecuta las pruebas una vez y sale (ideal para CI).
- `--coverage`: Activa la recolección de cobertura.

### Paso 2: El Código que Vamos a Probar
Crea un archivo `src/math.ts`. Nota que la función `dividir` tiene una rama lógica (un `if`) que maneja la división por cero.

```typescript
// src/math.ts
export function sumar(a: number, b: number): number {
  return a + b;
}

export function dividir(a: number, b: number): number {
  if (b === 0) {
    throw new Error("No se puede dividir por cero");
  }
  return a / b;
}
```

### Paso 3: Las Pruebas (Incompletas)
Ahora, crea un archivo de prueba `src/math.test.ts`, pero solo probaremos el "camino feliz" de la función `dividir`.

```typescript
// src/math.test.ts
import { describe, it, expect } from 'vitest';
import { sumar, dividir } from './math';

describe('Funciones matemáticas', () => {
  it('debería sumar dos números correctamente', () => {
    expect(sumar(2, 3)).toBe(5);
  });

  // Solo probamos el caso exitoso de la división
  it('debería dividir dos números correctamente', () => {
    expect(dividir(10, 2)).toBe(5);
  });
});
```

### Paso 4: Ejecutar y Analizar el Reporte de Cobertura
Ahora, ejecuta el comando en tu terminal:

```bash

npm run coverage

```

Vitest ejecutará las pruebas y te mostrará un reporte de cobertura en la consola similar a este:

---------------------|---------|----------|---------|---------|-------------------
File                 | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s 
---------------------|---------|----------|---------|---------|-------------------
All files            |   85.71 |       50 |     100 |   85.71 |                   
 src/math.ts         |   85.71 |       50 |     100 |   85.71 | 7                 
---------------------|---------|----------|---------|---------|-------------------

**Análisis del Reporte:**
- `% Stmts` **(Statements)**: 85.71% de las declaraciones fueron ejecutadas.
- `% Branch` **(Ramas)**: **¡Solo el 50%!** Esta es la pista clave. Significa que de las 2 posibles ramas en nuestro código (el `if` y el `else` implícito), solo una fue probada.
- `% Funcs` **(Funciones)**: **100%**. Ambas funciones (`sumar` y `dividir`) fueron llamadas.
- `% Lines` **(Líneas)**: **85.71%** de las líneas fueron ejecutadas.
- `Uncovered Line #s`: `7` . Vitest nos dice exactamente qué línea no se ejecutó: la línea 7, que es `throw new Error(...)`

Además, Vitest habrá creado una carpeta `coverage/` en tu proyecto. Si abres el archivo `coverage/index.html` en un navegador, verás un reporte visual detallado que resalta en rojo las líneas no cubiertas.

### Paso 5: Mejorar las Pruebas para Alcanzar el 100%
El reporte nos ha mostrado exactamente dónde está el punto ciego de nuestras pruebas. Ahora, vamos a añadir el caso de prueba que falta en `src/math.test.ts`.

```typescript
// src/math.test.ts
import { describe, it, expect } from 'vitest';
import { sumar, dividir } from './math';

describe('Funciones matemáticas', () => {
  it('debería sumar dos números correctamente', () => {
    expect(sumar(2, 3)).toBe(5);
  });

  it('debería dividir dos números correctamente', () => {
    expect(dividir(10, 2)).toBe(5);
  });

  // AÑADIMOS ESTA NUEVA PRUEBA
  it('debería lanzar un error al dividir por cero', () => {
    // Verificamos que llamar a la función con estos argumentos lanza un error.
    expect(() => dividir(5, 0)).toThrow("No se puede dividir por cero");
  });
});
```

### Paso 6: Volver a Ejecutar la Cobertura
Ejecuta de nuevo:

```bash

npm run coverage

```

Ahora, el reporte será mucho más satisfactorio:

---------------------|---------|----------|---------|---------|-------------------
File                 | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s 
---------------------|---------|----------|---------|---------|-------------------
All files            |     100 |      100 |     100 |     100 |                   
 src/math.ts         |     100 |      100 |     100 |     100 |                   
---------------------|---------|----------|---------|---------|-------------------

¡Éxito! Al añadir la prueba para el caso de error, hemos cubierto la otra rama lógica, y ahora tenemos un 100% de cobertura, lo que nos da mucha más confianza en que nuestra función `dividir` se comporta como esperamos en todos los escenarios.