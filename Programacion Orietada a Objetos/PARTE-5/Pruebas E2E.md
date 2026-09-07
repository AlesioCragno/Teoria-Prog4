# Pruebas de Extremo a Extremo (End-to-End o E2E): La Cima de la Pirámide

**Analogía**: Construir el castillo de LEGO completo y asegurarse de que se parece a la imagen de la caja.

- **Propósito**: Verificar un **flujo de usuario completo** a través de la aplicación, simulando el comportamiento de un usuario real. Es la prueba definitiva de que el sistema funciona como un todo.
- **Características**:
    - **Completas**: Involucran toda la pila tecnológica: el fronted en un navegador real, el backend, la base de datos, servicios externos, etc.
    - **Lentas y costosas**: Son las más lentas de ejecutar (pueden tardar unos minutos o segundos por pruebas) y las más frágiles (pueden fallas por problemas de red o cambios menores en la UI).
    - **Escasas**:
        - Automatizar un navegador (con herramientas como Cypress o Playwright) para que:
            1. Abra la página de inicio de sesión.
            2. Rellene el formulario con un usuario y contraseña.
            3. Haga click en "Iniciar Sesión".
            4. Verifique que es redirigido al panel de control y que su nombre de usuario aparece en pantalla.
