# Pruebas por línea de comandos

Cada archivo acá es un vector de prueba para el modo `--test-vector` de Logisim
Evolution (verificación por consola, sin abrir la interfaz gráfica). Son los mismos
vectores usados para la verificación de la sección 2 de [`../docs/04-verificacion.md`](../docs/04-verificacion.md).

## Cómo correr una prueba

```
logisim-evolution --test-vector <circuito> <archivo_de_prueba> <archivo.circ>
```

Desde la raíz del repo, con Logisim Evolution instalado:

```bash
LOGISIM="logisim-evolution"   # o la ruta completa al ejecutable

# Cada FSM por separado (todas las transiciones estado x entrada)
$LOGISIM --test-vector PinChecker     tests/pinchecker_fsm.txt      circuitos/cerradura_pin.circ
$LOGISIM --test-vector LockController tests/lockcontroller_fsm.txt  circuitos/cerradura_pin.circ

# Las dos FSMs integradas (circuito main): 4 digitos correctos abre, L re-bloquea,
# 3 fallos -> alarma, PIN correcto en alarma NO abre, Reset saca de la alarma
$LOGISIM --test-vector main           tests/integracion_completa.txt circuitos/cerradura_pin.circ
```

Todas deberían terminar con `Failed: 0`.

## Formato de los archivos

Cada archivo tiene una fila de encabezado (nombre de cada pin) y después una fila por
ciclo de reloj. `<DC>` quiere decir "no importa el valor en esa columna para esa fila"
(normalmente porque ese ciclo es el del flanco de reloj, y la salida todavía refleja el
estado anterior en ese instante exacto). La columna `<set>` agrupa filas que comparten
el mismo estado de los flip-flops (todas en 1 acá, porque cada archivo prueba una sola
secuencia continua), y `<seq>` es solo el número de fila dentro de ese grupo.
