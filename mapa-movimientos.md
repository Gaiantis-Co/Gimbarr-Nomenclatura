# Mapa de Movimientos

Nomenclatura de movimientos con las relaciones actuales de la base de datos:
las **líneas continuas** representan las **evoluciones** (tabla `evoluciones`) y las **variantes** se listan dentro del array del nodo base (tabla `variaciones`).

```mermaid
flowchart TB

    M4(("Dominada<br/>Requisitos[]<br/>Variantes[]"))
    M5(("Entrala de ladron<br/>Requisitos[Dominada]<br/>Variantes[]"))
    M6(("Anclar<br/>Requisitos[Entrala de ladron]<br/>Variantes[Duo, Contra]"))
    M10(("Apolo<br/>Requisitos[Anclar]<br/>Variantes[]"))
    M11(("Escuadra<br/>Requisitos[Dominada]<br/>Variantes[]"))
    M12(("Nivel<br/>Requisitos[Dominada]<br/>Variantes[]"))
    M13(("Cero<br/>Requisitos[Anclar]<br/>Variantes[]"))

    M5 --- M6
    M6 --- M10
    M4 --- M5
    M4 --- M11
    M4 --- M12
    M6 --- M13
```
