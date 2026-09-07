# Dobles de Prueba: Controlando el Entorno de Tus Tests
En el testing, a menudo necesitamos que nuestro código interactúe con otras partes del sistema (dependencias). Sin embargo, para que una prueba sea rápida, predecible y aislada, no queremos depender de una base de datos real, una API externa o incluso de un módulo complejo.

Aquí es donde entran los **Dobles de Prueba** ( *Test Doubles* ). Son objetos o funciones "impostoras" que se hacen pasar por las dependencias reales, dándonos control total sobre el entorno de la prueba. Piensa en ellos como los dobles de acción en una película: parecen reales y hacen el trabajo necesario para la escena, pero sin el riesgo del actor principal.

Vitest ofrece varias herramientas para crear estos dobles:

## Spies: El Observador
**Analogía**: Un espía o un informante que observa y toma notas de lo que sucede, pero sin interferir con la operación original.

- **¿Para qué sirven?** Para verificar **interacciones**. Quieres saber si una función fue llamada, *cuántas veces* fue llamada y con qué argumentos fue llamada, pero dejando que la función original se ejecute normalmente.
- **En Vitest**: Se crean con `vi.fn()` (para una función genérica) o `vi.spyOn(objeto, 'metodo')` para "envolver" un método existente.

```typescript
// Ejemplo: ¿Se está llamando correctamente al sistema de logging?
const logger = {
  log: (message: string) => {
    console.log(message); // Comportamiento original
  }
};

it('debería llamar al logger con el mensaje correcto', () => {
  // 1. Creamos un espía que envuelve el método `log`
  const logSpy = vi.spyOn(logger, 'log');

  // 2. Ejecutamos el código que queremos probar
  function registrarUsuario(nombre: string) {
    logger.log(`Usuario ${nombre} registrado.`);
  }
  registrarUsuario('Ana');

  // 3. Verificamos la interacción con el espía
  expect(logSpy).toHaveBeenCalledTimes(1);
  expect(logSpy).toHaveBeenCalledWith('Usuario Ana registrado.');
});
```

## Mocks: El Reemplazo
**Analogía**: Un actor impostor que reemplaza completamente al actor real. Tú le das el guion y él dirá exactamente lo que le pidas.

- **¿Para qué sirven?** Para **aislar** el código que estás probando de sus dependencias. Reemplazas un módulo o función completo con una versión controlada que devuelve los datos que tú quieras, evitando llamadas a APIs, bases de datos, etc.
- **En Vitest**: Se usa `vi.mock('ruta/al/modulo')` para reemplazar un módulo entero.

## Fake Timers: El Controlador del Tiempo
**Analogía**: Un control remoto para una máquina del tiempo. Puedes pausar, adelantar o retroceder el tiempo a tu antojo.

- **¿Para qué sirven?** Para probar código que depende del paso del tiempo (como `setTimeout` , `setInterval` o `Date` ) de forma **instantánea y predecible**, sin tener que esperar realmente.
- **En Vitest**: Se activan con `vi.useFakeTimers()` y se controlan con funciones como `vi.advanceTimeByTime()`.