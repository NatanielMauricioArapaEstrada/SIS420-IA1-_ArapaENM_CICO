# Racha ascendente: resumen del proyecto

Comparación de tres agentes tabulares que aprenden cuándo robar y cuándo plantarse para aumentar la probabilidad de ganar una partida.

## Juego

- Inicio: 49 puntos; meta: 82 o más; máximo: cinco rondas. Cada ronda reinicia y baraja siete cartas: 1, 3, 5, 7, K (13) y dos jokers. Primera extracción obligatoria.
- Una carta superior reemplaza la actual; plantarse suma únicamente esa carta. Una inferior termina la ronda sin sumar. Un joker inicial no descuenta; después de una numérica, resta su doble y termina la ronda.
- Se gana inmediatamente al alcanzar la meta; se pierde al llegar a cero o menos, o agotar las rondas sin alcanzarla. Recompensa final: +1 por ganar, −1 por perder.

## Construcción del código

Funciones: `jugar` (entorno), `elegir_accion` (política), `actualizar_q/h` (aprendizaje) y `experimentar/evaluar` (pruebas). Se aprende por experiencia, sin un modelo de planificación.

Estado: `(puntaje, rondas restantes incluida la actual, carta actual o None, cartas pendientes)`. La memoria usa claves `(estado, acción)`: valores Q o preferencias H y contadores de visitas. El historial permite actualizar las decisiones al terminar la partida; H añade una referencia media por estado.

## Métodos y fórmulas

**ε-greedy y UCB comparten la actualización por promedio exacto:**

`Q(s,a) ← Q(s,a) + [R − Q(s,a)] / n(s,a)`

`R`: recompensa final; `n(s,a)`: visitas, incluida la nueva experiencia. Q promedia los resultados finales observados.

- **ε-greedy:** con probabilidad ε elige una acción uniforme al azar; en caso contrario, elige la mayor Q.
- **UCB:** prueba primero las acciones sin visitas; después elige el mayor puntaje:

`Q(s,a) + c × √[ln(N(s)) / n(s,a)]`

`N(s)` suma las visitas del estado; `c` pondera el bono para acciones menos probadas. El bono no se guarda en Q.

**H: preferencias con softmax y ascenso por gradiente adaptados a partidas completas.**

`π(a|s) = exp(H(s,a) − m) / Σᵤ exp(H(s,u) − m)`

`m = máximo H`: estabiliza el cálculo sin cambiar las probabilidades. Se elige una acción según π.

`H(s,a) ← H(s,a) + α × [R − b(s)] × [1(a=A) − π(a|s)]`

`A`: acción elegida; indicador: 1 para ella, 0 para la otra; `b(s)`: recompensa media previa. Después se actualiza la referencia:

`b(s) ← b(s) + [R − b(s)] / N(s)`

Aquí N incluye la nueva visita. Se omiten decisiones obligatorias. H aprende preferencias, no Q.

## Comparación

- Parámetros constantes: ε = {0,1; 0,2; 0,4}; c = {0,5; 1; 2}; α de H = {0,05; 0,1; 0,2}.
- Semillas 42, 43 y 44: 27 entrenamientos independientes desde cero, de 200.000 partidas cada uno (5,4 millones en total).
- Evaluación: 20.000 partidas a las 0, 10.000, 50.000, 100.000 y 200.000 entrenadas; mayor Q o H, sin explorar ni actualizar.
- Gráficas promediadas entre semillas: victorias en evaluación y recompensa del entrenamiento por bloques de 2.000 partidas, incluyendo exploración.

## Resultados observados

| Método | Mejor configuración probada | Victorias finales medias |
|---|---|---:|
| UCB | c = 2 | 24,15 % |
| ε-greedy | ε = 0,4 | 23,96 % |
| H | α = 0,2 | 23,57 % |

UCB obtuvo la mayor media observada; los resultados son cercanos y corresponden a estas configuraciones y tres semillas.
