# Pruebas Unitarias (Units Tests): La Base de la Pirámide

**Analogía**: Probar un solo ladrillo de LEGO para asegurar que tiene el color y forma correctos antes de construir.

- **Propósito**: Verificar que la **unidad mas pequeña y aislada** de código funciona como se espera. Una "unidad" suele ser una función, un método de una clase o un componente de UI en completo aislamiento.
- **Características**: 
    - **Aisladas**: no dependen de otras partes del sistema como bases de datos, redes o el DOM. Las dependencias externas se "simulan" (mocking).
    - **Rápidas**: Se ejecutan en milisegundos. Puedes tener miles de ellas y ejecutarlas en segundos.
    - **Abundantes**: Forman la gran mayoría  de tus pruebas.
