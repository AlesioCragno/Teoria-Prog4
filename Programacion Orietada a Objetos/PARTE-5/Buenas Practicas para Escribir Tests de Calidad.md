# Buenas Prácticas para Escribir Tests de Calidad
Escribir pruebas no es solo cuestión de aumentar la cobertura; es cuestión de crear una red de seguridad que te dé **confianza** para refactorizar y añadir nuevas funcionalidades. Una suite de pruebas de baja calidad puede convertirse en una carga en lugar de una ayuda. Aquí se presentan tres principios fundamentales para mantener tus pruebas robustas y mantenibles.

## 1. Regla de Hierro: Determinismo y Repetibilidad
Un test debe comportarse como un experimento científico: dadas las mismas condiciones de entrada, **siempre debe producir el mismo resultado**.

- **Principio**: Si ejecutas la suite de pruebas 100 veces sin cambiar el código, las 100 veces debe pasar. Si una prueba falla, debe ser porque se ha introducido un bug real, no por azar.
- **¿Por qué es crucial?** La confianza es la moneda de cambio de una suite de pruebas. Si los tests fallan de forma intermitente, el equipo empezará a ignorarlos ("*Ah, es solo el test frágil de siempre, ejecútalo de nuevo*"). Una suite de pruebas en la que no se confía es una suite de pruebas inútil.

## 2. Claridad y Enfoque: Un Test, Una Idea
Cada bloque `it(...)` debe ser fácil de leer, fácil de entender y debe probar una sola cosa.

- **Principio**: Un test debe verificar un único comportamiento o resultado. Su nombre debe describir claramente ese comportamiento, siguiendo un patrón como "debería [hacer algo] cuando [ocurre una condición]".
- **¿Por qué es crucial?** Cuando un test falla, su nombre debería decirte exactamente qué se rompió sin necesidad de leer el código. Si un test llamado `it('prueba el servicio de usuario')` falla, no tienes ni idea de cuál de las 10 aserciones que contiene es la que falló.

**Estructura Recomendada: Arrange-Act-Assert (AAA)**

| Fase | Propósito | Ejemplo |
|---|---|---|
| **Arrange (Preparar)** | Configura el estado inicial y las dependencias (mocks, datos de entrada). | `const usuario = { esAdmin: false };` |
| **Act (Actuar)** | Ejecuta la función o el método que quieres probar. | `const puedeAcceder = puedeAccederAlPanel(usuario);` |
| **Assert (Verificar)** | Comprueba que el resultado es el esperado. | `expect(puedeAcceder).toBe(false);` |

```typescript
// Mal nombre, múltiples ideas
it('funciona', () => {
  expect(sumar(2, 2)).toBe(4);
  expect(sumar(-1, 1)).toBe(0);
});

// Buen nombre, una sola idea, estructura AAA
it('debería devolver 0 cuando se suman un número positivo y su negativo', () => {
  // Arrange
  const a = -5;
  const b = 5;

  // Act
  const resultado = sumar(a, b);

  // Assert
  expect(resultado).toBe(0);
});
```