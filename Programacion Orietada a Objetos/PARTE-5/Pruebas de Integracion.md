# Pruebas de Integracion (Integration Tests): El Nivel Intermedio

**Analogía**: Probar que dos o tres ladrillos de LEGO encajan correctamente entre sí.

- **Propósito**: Verificar que **varios componentes interactúan correctamente** cuando se combinan. El objetivo es encontrar errores en las "costuras" o interfaces entre los módulos.
- **Características**:
    - **Combinadas**: Prueban un flujo corto que involucra a varios módulos.
    - **Mas lentas**: Tardan más que las pruebas unitarias porque involucran más código y, a veces, sistemas reales (como una base de datos en memoria).
    - **Menos numerosas**: Tienes menos pruebas de integración que unitarias.