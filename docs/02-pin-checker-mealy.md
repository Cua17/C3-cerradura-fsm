# PinChecker — FSM Mealy

Sigue el mismo procedimiento de 7 pasos visto en clase (Ch3, "FSM Design Procedure").

## 1. Entradas y salidas

- Entrada: `D` (1 = el dígito que se acaba de ingresar es correcto para esta posición
  del PIN, 0 = incorrecto).
- Salidas: `match` (los 4 dígitos fueron correctos), `wrong` (este dígito falló).
- Reloj: un ciclo = un dígito ingresado (igual convención que el caracol de clase, donde
  un ciclo = un bit de la cinta).

## 2. Diagrama de transición de estados

4 estados: `S0` (0 dígitos correctos seguidos), `S1` (1), `S2` (2), `S3` (3 — el próximo
dígito correcto completa el PIN). Como es Mealy, la salida va **sobre la flecha**, no
dentro del círculo: `entrada/salida`.

Cadena de avance con `D=1` (sin salida activa todavía):

```
(S0) ──D=1/–──▶ (S1) ──D=1/–──▶ (S2) ──D=1/–──▶ (S3) ──D=1/match──▶ (S0)
```

Y desde **cualquiera** de los cuatro estados, `D=0` regresa directo a `S0` con la
salida `wrong=1` (flechas no dibujadas arriba para no saturar el diagrama, pero están
todas en la tabla de la sección 3). El estado `S0` lleva doble círculo por ser también
el estado de reset.

## 3. Tabla de transición de estados (FSM State Transition Table)

| S | D | S' | match | wrong |
|---|---|----|----|----|
| S0 | 0 | S0 | 0 | 1 |
| S0 | 1 | S1 | 0 | 0 |
| S1 | 0 | S0 | 0 | 1 |
| S1 | 1 | S2 | 0 | 0 |
| S2 | 0 | S0 | 0 | 1 |
| S2 | 1 | S3 | 0 | 0 |
| S3 | 0 | S0 | 0 | 1 |
| S3 | 1 | S0 | 1 | 0 |

No hay combinaciones "no importa": con 4 estados y 2 bits de estado se usan las 4
combinaciones exactas, sin sobrantes.

## 4. Codificación de estados

Codificación binaria (la misma que usa el profesor en el semáforo y el caracol; no
one-hot, porque con solo 4 estados binario ya da ecuaciones simples y usa la mitad de
flip-flops):

| State | Encoding (Q1 Q0) |
|---|---|
| S0 | 00 |
| S1 | 01 |
| S2 | 10 |
| S3 | 11 |

## 5. Tabla combinada de transición y salida (Mealy FSM State Transition & Output Table)

| Q1 Q0 | D | N1 N0 | match | wrong |
|---|---|---|---|---|
| 00 | 0 | 00 | 0 | 1 |
| 00 | 1 | 01 | 0 | 0 |
| 01 | 0 | 00 | 0 | 1 |
| 01 | 1 | 10 | 0 | 0 |
| 10 | 0 | 00 | 0 | 1 |
| 10 | 1 | 11 | 0 | 0 |
| 11 | 0 | 00 | 0 | 1 |
| 11 | 1 | 00 | 1 | 0 |

## 6. Ecuaciones booleanas

Minimizadas con `sympy.logic.boolalg.SOPform` (Quine-McCluskey) para no arrastrar
errores de un Karnaugh a mano; el resultado es reproducible con cualquier minimizador
o mapa de Karnaugh hecho a mano (son solo 3 variables):

$$N_1 = D \cdot (Q_1 \oplus Q_0) = D\overline{Q_1}Q_0 + DQ_1\overline{Q_0}$$
$$N_0 = D \cdot \overline{Q_0}$$
$$match = D \cdot Q_1 \cdot Q_0$$
$$wrong = \overline{D}$$

Verificación cruzada a mano (fila S3, D=1 → N1N0 debe ser 00 y match=1):
$N_1 = 1\cdot(1\oplus1) = 1\cdot 0 = 0$ ✓, $N_0 = 1\cdot\overline{1} = 0$ ✓,
$match = 1\cdot1\cdot1 = 1$ ✓.

## 7. Esquemático

Registro de estado: 2 flip-flops D (`Q1`, `Q0`), reloj compartido con `LockController`,
reset síncrono a `00`. Lógica de siguiente estado: un XOR y dos AND para `N1`, un AND
con inversor para `N0` (ver ecuaciones arriba). Lógica de salida: un AND de 3 entradas
para `match` (`D·Q1·Q0`), un inversor para `wrong`. Construido y verificado en Logisim
en [`../circuitos/`](../circuitos/) (fase 2).
