# Diseño general — Cerradura electrónica con código PIN

## El dispositivo

Una cerradura electrónica con teclado numérico. El usuario ingresa un PIN de 4 dígitos,
un dígito por ciclo de reloj (igual que el ejemplo del caracol de Alyssa P. Hacker visto
en clase, que revisa la cinta bit por bit en cada flanco). Si los 4 dígitos coinciden
con el PIN guardado, la cerradura se abre. Si un dígito falla, el intento se reinicia.
Después de 3 intentos fallidos, la cerradura entra en modo alarma y solo un reset físico
(botón de administrador) la puede sacar de ahí.

## Por qué se factoriza en dos FSMs

Siguiendo la idea de "Factoring State Machines" de clase (romper una FSM compleja en
FSMs más chicas que interactúan, como el semáforo con Parade Mode), esta cerradura se
separa en dos máquinas con responsabilidades bien distintas:

1. **`PinChecker`** (Mealy) — compara la secuencia de dígitos contra el PIN guardado,
   dígito por dígito.
2. **`LockController`** (Moore) — controla el estado físico de la cerradura (bloqueada,
   cuántos intentos fallidos lleva, alarma, o abierta).

No se hicieron una sola FSM combinada a propósito: son dos problemas distintos con
requisitos de timing distintos.

- `PinChecker` necesita reaccionar **en el mismo ciclo** en el que llega el dígito
  correcto número 4, para no perder un ciclo de reloj avisando "coincide" — por eso es
  **Mealy** (salida depende de estado *y* entrada, igual que el caracol).
- `LockController` maneja el estado físico real de una cerradura (el pestillo, la luz
  de alarma). Ese tipo de salida **no debe depender directamente de una entrada
  cualquiera** — solo del estado ya estable — porque si alguien mete un dígito
  incorrecto justo en el filo de un flanco de reloj, no queremos que el pestillo
  parpadee. Por eso es **Moore**.

Esto es exactamente el mismo criterio que usó el profesor en la *washer*: un contador
(Moore, porque su salida — cuánto dinero lleva — es un valor de estado estable) y una
lógica de arranque (Mealy, porque necesita reaccionar de inmediato a la combinación de
estado + botón de arranque).

## Interfaz entre las dos FSMs

```
                 ┌───────────────┐          ┌──────────────────┐
   D (dígito) ─▶│                │  match  │                    │──▶ unlock
                 │   PinChecker   │────────▶│   LockController   │──▶ alarm
   CLK, Reset ─▶│    (Mealy)     │  wrong  │      (Moore)       │──▶ locked
                 │                │────────▶│                    │
                 └───────────────┘          └──────────▲─────────┘
                                                         │
                                              L (botón de re-bloqueo)
                                              CLK, Reset (admin, sale de Alarm)
```

- `D` = 1 si el dígito que se acaba de ingresar es el correcto para la posición actual
  del PIN, 0 si no. (La comparación dígito-contra-PIN-guardado es un comparador
  combinacional externo a las dos FSMs — el enfoque de la tarea es el diseño de las
  FSMs, igual que el caracol de clase recibe su entrada `A` ya como un bit de
  "coincide/no coincide", sin modelar el sensor que lee la cinta.)
- `match` = pulso de `PinChecker` a `LockController`: "los 4 dígitos fueron correctos".
- `wrong` = pulso de `PinChecker` a `LockController`: "este dígito falló, se reinicia
  el intento".
- `L` = botón físico para volver a bloquear la puerta cuando está abierta.
- El `Reset` global es compartido por ambas FSMs. Sacar a `LockController` de `Alarm`
  requiere ese mismo Reset — a propósito: conceptualmente, solo alguien con acceso para
  reiniciar todo el sistema (un administrador) debería poder cancelar una alarma, no un
  botón cualquiera del teclado.

Ver el diseño de cada FSM en detalle en [`02-pin-checker-mealy.md`](02-pin-checker-mealy.md)
y [`03-lock-controller-moore.md`](03-lock-controller-moore.md).
