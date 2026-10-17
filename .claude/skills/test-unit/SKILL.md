---
name: test-unit
description: Escribe pruebas unitarias cuando se añade o cambia una pieza que hace cuentas (cálculos, validaciones, reglas de negocio).
---

Cuándo usarme: cuando se añada o cambie una pieza que hace cuentas.

Cómo escribir las pruebas

    Lee la pieza y haz una lista de sus casos raros: vacío, cero, negativo, el límite, el formato equivocado.
    Una prueba por comportamiento, con nombre en español que diga qué comprueba.
    Nada de piezas falsas. Si hacen falta, el problema es el diseño: dilo y para.

Al terminar Corre las pruebas y pega el resultado tal cual. Si sale rojo, la prueba encontró un error de verdad: avísame. NO cambies el número esperado para que pase.