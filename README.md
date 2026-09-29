# C3 — Cerradura electrónica con código PIN (FSM Moore + Mealy)

Tarea de Arquitectura de Computadoras: diseño de una máquina de estados finitos
factorizada en dos FSMs interactuando entre sí (una Moore, una Mealy), siguiendo la
metodología de diseño de FSM vista en clase.

José Daniel Cuá Fagiani — Carné 21200

## Contenido

1. [Diseño general y factorización](docs/01-diseno-general.md)
2. [PinChecker — FSM Mealy](docs/02-pin-checker-mealy.md)
3. [LockController — FSM Moore](docs/03-lock-controller-moore.md)

## Resultados principales

| FSM | Tipo | Estados | Función |
|---|---|---|---|
| PinChecker | Mealy | 4 (S0–S3) | Compara el PIN dígito por dígito; `match`/`wrong` reaccionan en el mismo ciclo |
| LockController | Moore | 5 (Locked0/1/2, Alarm, Unlocked) | Controla el pestillo y la alarma; salidas dependen solo del estado |

Ecuaciones minimizadas y verificadas con `sympy.logic.boolalg.SOPform` (Quine-McCluskey) —
ver el detalle en cada documento de diseño.

## Estructura del repositorio

```
docs/       explicación del diseño de cada FSM, paso a paso (igual metodología que clase)
tablas/     fsm_tablas.xlsx -- tablas de transición, codificación y salida de ambas FSMs
circuitos/  archivos .circ de Logisim, verificados por línea de comandos (fase 2)
img/        capturas y diagramas (fase 2)
```

## Herramientas usadas

- **Logisim Evolution** para los circuitos digitales (fase 2), verificado con su modo de
  consola (`--tty table`).
- **sympy** (Python) para minimizar las ecuaciones de siguiente-estado y de salida por
  Quine-McCluskey, evitando errores de Karnaugh hecho a mano.
- **openpyxl** (Python) para generar las tablas de FSM en Excel.

## Video

Pendiente — enlace privado de YouTube explicando la FSM, su funcionalidad y las
decisiones de construcción (máx. 5 min, fase 3).
