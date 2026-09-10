# GDD — Autobattler PvE (nombre provisorio: "Contramarcha")

Versión 0.2 — incorpora ambientación, vista, estructura de run y presentación
definidas. Sujeto a revisión durante prototipado.

Engine: Godot 4.7 / GDScript.
Duración objetivo: 6-8 semanas.
Rol que demuestra: systems design / balance / diseño de encuentros.

---

## 1. Contexto de género

El autobattler nació con Dota Auto Chess (2019) y se consolidó con Teamfight
Tactics y Hearthstone Battlegrounds. Su forma canónica es PvP: fase de tienda,
fase de combate automático, ocho jugadores, eliminación.

En los últimos años el género se corrió hacia el single-player (Hadean Tactics,
Necroking, He Is Coming) y hacia variantes que abandonan la grilla de sinergias
clásica:

- **Mechabellum** — despliegue con presupuesto sobre un campo abierto, sistema de
  counters denso, upgrades entre rondas. El interés está en la predicción y el
  posicionamiento, no en juntar copias de unidades.
- **Backpack Battles** — la decisión es espacial y de inventario, no de roster.
  Hecho en Godot, dato relevante como precedente técnico.

Este proyecto se posiciona en la línea de Mechabellum, adaptada a single-player,
a fantasía medieval y a un alcance de prototipo.

## 2. Definición de género y límite duro

**Esto es un autobattler.** El jugador toma todas sus decisiones antes del
combate y no interviene durante la ejecución.

Este límite se declara en el documento porque durante el diseño apareció
repetidamente la tentación de dar órdenes por turno, elegir objetivos durante la
pelea o mover piezas con patrones tipo ajedrez. Todo eso convierte el proyecto en
un táctico por turnos, que es otro género, con otro alcance y con la IA como
sistema central.

La razón de mantener el límite es de alcance: en un autobattler PvE el enemigo no
necesita inteligencia, porque es una composición fija escrita a mano. Eso libera
todo el presupuesto de tiempo para balanceo y diseño de encuentros, que es lo que
el proyecto quiere demostrar.

Cualquier idea que requiera que el jugador tome una decisión después de confirmar
el despliegue va a la lista de v2.

## 3. Pitch

Un autobattler de un solo jugador de fantasía medieval, presentado como un
wargame de mesa. Escalás una campaña de doce encuentros contra ejércitos
enemigos diseñados a mano. Antes de cada batalla ves exactamente qué te espera
del otro lado y tenés oro para reclutar y posicionar tus piezas. Después soltás
la batalla y las miniaturas se resuelven solas.

No hay tienda aleatoria, no hay matchmaking, no hay otros jugadores.

## 4. Objetivo del proyecto (portfolio)

Demostrar:

- Diseño de un sistema de counters legible y no trivial.
- Balanceo numérico documentado de un roster completo, respaldado por datos.
- Diseño de encuentros: una curva de dificultad construida a mano.
- Arquitectura de simulación determinista y depurable.

Criterio de éxito: un jugador debe poder perder una batalla, entender por qué
perdió, y ganarla al reintentarla cambiando su despliegue.

## 5. Decisión de diseño central

**Información completa antes del combate.**

En los autobattlers PvP clásicos, buena parte de la tensión viene de no saber
contra qué composición vas a chocar. Acá se elimina esa incertidumbre a
propósito: el jugador ve el ejército enemigo y su posicionamiento antes de
desplegar.

Razones:

- Convierte cada batalla en un puzzle con solución, no en una apuesta.
- Hace que el sistema de counters sea el contenido del juego, no un modificador.
- Elimina la necesidad de matchmaking, backend y balance PvP.
- Permite diseñar la curva de dificultad batalla por batalla.

Este es el punto a defender en el README, porque es la decisión que diferencia el
proyecto de un clon de TFT.

## 6. Loop

Loop de batalla:
1. Se muestra el ejército enemigo y su despliegue.
2. El jugador recibe oro y recluta unidades, colocándolas en su mitad del tablero.
3. Puede reposicionar y rearmar libremente hasta confirmar.
4. Combate automático. Sin intervención de ningún tipo.
5. Resultado: victoria o derrota.

Loop de campaña:
- Doce batallas en secuencia, con oro creciente.
- Tras cada victoria, el jugador elige una de tres recompensas.
- El jugador tiene un número limitado de vidas por campaña (ver 7.3).

## 7. Sistemas

### 7.1 Tablero

Vista top-down con profundidad real: el tablero se ve desde arriba, pero las
miniaturas están paradas y se ven en tres cuartos, al estilo de Into the Breach.
Esto resuelve la tensión entre legibilidad de silueta y profundidad táctica.

Tablero rectangular dividido en dos mitades. Grilla de despliegue de baja
resolución, por ejemplo 10x6 por lado. La profundidad es real: hay retaguardia,
hay flancos, y rodear es una maniobra posible.

Resolución base fija y baja, con snapping de píxel. Definir en la primera semana:
cambiarlo después obliga a rehacer todos los sprites.

Sin obstáculos ni terreno en la v1.

### 7.2 Roster

Ocho unidades. No más. Cada una define un rol claro y ninguna es una versión
mejor de otra.

| Unidad | Rol | Función | Contrarresta | Es contrarrestada por |
|---|---|---|---|---|
| Milicianos | Enjambre | Muchos, baratos, frágiles | Objetivo único | Área |
| Cañón de asedio | Área | Daño en zona, lento, corto alcance efectivo | Enjambre | Alcance largo |
| Arqueros | Alcance largo | Alto daño desde atrás, muy frágiles | Área, tanque | Flanqueo |
| Jinetes | Flanqueo | Rápidos, rodean hacia la retaguardia | Alcance largo | Enjambre |
| Guardia pesada | Tanque | Mucha vida, poco daño, absorbe | Objetivo único | Daño porcentual |
| Caballero | Objetivo único | Daño concentrado alto | Tanque | Enjambre |
| Clériga | Soporte | Cura o escuda en área | — | Flanqueo |
| Asesino | Antisoporte | Prioriza soporte y unidades caras | Soporte | Tanque |

Regla de diseño: el grafo de counters no debe ser un ciclo simple. Debe tener
suficientes aristas como para que la respuesta a un ejército no sea obvia de un
vistazo, pero sí deducible.

Los nombres son provisorios y sirven sobre todo para que el rol se entienda sin
consultar la tabla.

### 7.3 Estructura de campaña y vidas

El jugador empieza la campaña con tres vidas. Perder una batalla consume una
vida y permite reintentarla. Al quedarse sin vidas, la campaña termina y se
reinicia desde el principio.

Esta es una decisión deliberada de que la derrota tenga peso.

**Punto de revisión declarado:** esta regla tensiona con el criterio de éxito de
la sección 4. Un jugador que no entendió un counter puede perder la campaña sin
haber aprendido nada, y un evaluador de portfolio probablemente juegue veinte
minutos. La regla se mantiene para el primer build, pero es lo primero que hay
que revisar tras el primer playtest con alguien externo. Si el testeador
abandona antes de la batalla seis, la causa es casi seguro esta.

Mitigaciones a tener listas por si hace falta: recuperar una vida cada cuatro
victorias, o agregar un modo escaramuza donde cualquier batalla ya vista se puede
reintentar libremente fuera de la campaña. El modo escaramuza tiene además valor
de portfolio, porque le permite al evaluador saltar a la batalla 11 sin jugar las
diez anteriores.

### 7.4 Oro y reclutamiento

Cada unidad tiene un costo en oro. El oro por batalla crece según una curva
definida en datos. El jugador puede reclutar cualquier combinación que entre en
el presupuesto; no hay límite de tipos ni requisito de diversidad.

**Tope duro: 18 unidades por bando en el tablero.** Este número no es estético,
es un límite de legibilidad y de rendimiento de la simulación. Los milicianos son
unidades individuales, así que un ejército de enjambre llega al tope rápido, y
eso es parte del trade-off del rol: mucha presencia, poca flexibilidad.

Restricción deliberada: el ejército se rearma desde cero en cada batalla. No hay
ejército persistente. Esto obliga a resolver cada puzzle en sus propios términos y
evita que una build ganadora arrastre toda la campaña.

### 7.5 Comportamiento de unidad en combate

Cada unidad, por tick de simulación:

1. Si tiene un objetivo válido en rango, ataca respetando su cadencia.
2. Si no, avanza hacia su objetivo prioritario según su regla de movimiento.

**Reglas de targeting** (parámetro de datos, no código por unidad): más cercano,
más lejano, menor vida, mayor costo, soporte primero.

**Reglas de movimiento** (también en datos). Acá es donde las unidades se
diferencian sin salir del combate automático:

- *Avance frontal*: va derecho hacia adelante. Milicianos, guardia pesada.
- *Avance con retención*: avanza hasta tener un objetivo en rango y se detiene.
  Arqueros, cañón.
- *Flanqueo*: se desplaza primero hacia el borde del tablero y después avanza,
  buscando la retaguardia enemiga. Jinetes.
- *Infiltración*: ignora el frente y va directo hacia su objetivo prioritario
  atravesando el campo. Asesino.
- *Seguimiento*: se mantiene detrás de la unidad aliada más cercana. Clériga.

Esto le da a cada pieza una personalidad de movimiento visible, que era la
intención detrás de la idea de patrones tipo ajedrez, pero sin romper el combate
automático ni obligar a órdenes por turno.

Sin pathfinding complejo: movimiento directo con separación básica entre
unidades.

### 7.6 Determinismo

La simulación debe ser determinista: mismo despliegue y misma semilla, mismo
resultado. No es un lujo, es lo que permite balancear.

Consecuencias de implementación:
- Simulación por ticks fijos, desacoplada del framerate.
- Toda aleatoriedad pasa por un generador con semilla explícita.
- La visualización lee el estado de la simulación, no al revés.

Beneficio de portfolio: permite construir un simulador headless que corre miles
de combates para validar el balance. Es el entregable que más impresiona y es la
razón principal por la que el proyecto sigue siendo un autobattler y no un
táctico.

**Es la decisión técnica más importante del proyecto y no se puede retrofitear.**
Semana uno o nunca.

### 7.7 Recompensas entre batallas

Tras cada victoria, elegir una de tres cartas de recompensa:

- Aumento de oro para las batallas siguientes.
- Mejora de una unidad concreta: más vida, más daño, más alcance.
- Modificador global: por ejemplo, todas tus unidades empiezan con escudo.

Pool chico, 15 a 20 recompensas. Cada una debe cambiar qué ejércitos son
viables, no solo subir un número.

## 8. Diseño de encuentros

Las doce batallas están diseñadas a mano y ordenadas con intención:

- Batallas 1-3: enseñan un counter cada una. Ejército enemigo monotipo.
- Batallas 4-7: dos tipos que exigen dividir la respuesta.
- Batallas 8-10: posicionamiento aprovechado, por ejemplo arqueros protegidos
  detrás de guardia pesada, o jinetes esperando en un flanco.
- Batallas 11-12: composiciones que castigan las respuestas aprendidas y obligan
  a reevaluar.

Cada batalla se documenta con: qué enseña, cuál es la solución esperada, y qué
soluciones alternativas se consideran válidas.

Entregable de portfolio: ese documento de encuentros vale tanto como el juego.

## 9. Interfaz

- Panel del ejército enemigo con conteo por tipo, siempre visible.
- Panel de reclutamiento con costos y oro restante.
- Tooltip de unidad con estadísticas, regla de targeting, regla de movimiento, y
  a qué contrarresta.
- Contador de vidas de campaña siempre visible.
- Durante el combate: barras de vida y nada más.
- Al terminar: resumen de daño hecho y recibido por tipo de unidad.

El resumen post-batalla es un requisito, no un extra. Es lo que le permite al
jugador entender por qué perdió. Sin él el juego es opaco y las vidas limitadas
se vuelven insoportables.

## 10. Arte y presentación

### 10.1 Marco visual: miniaturas de wargame sobre un tablero

El juego se presenta explícitamente como un wargame de mesa de fantasía medieval:
el campo es un tablero, las unidades son miniaturas pintadas, y el combate son
piezas que se empujan, se tambalean y se caen.

No es una concesión al presupuesto de arte, es una decisión de diseño:

- **Elimina la necesidad de animación de personaje.** Una miniatura no camina ni
  ataca; se desliza y choca. Todo el feedback se resuelve con transformaciones de
  posición, rotación y escala, sin dibujar un frame extra.
- **Es coherente con el género.** El autobattler nació del ajedrez automático y su
  decisión central es la colocación de piezas.
- **Justifica la falta de control durante el combate.** Soltar las piezas y
  mirarlas resolverse es exactamente lo que hace el jugador.

Alternativas descartadas: cartas (rectángulos idénticos, ilegibles en masa,
arrastran expectativa de mazo y mano) e íconos planos (baratos pero el campo
queda como un diagrama y se pierde la sensación de combate).

### 10.2 Producción de sprites

- **Un sprite por unidad**, sin frames de animación. Miniatura en tres cuartos
  sobre su base circular.
- **Un solo tamaño de lienzo** para todas, por ejemplo 32x32. Las diferencias de
  escala se sugieren con la silueta y la altura ocupada, no con lienzos distintos.
- Total realista: 8 sprites, más una variante de paleta para el bando enemigo
  aplicada por shader.
- Base visible: ancla la pieza al tablero y ayuda a leer la casilla exacta.
- Los milicianos, al ser unidades individuales y numerosas, se dibujan más chicos
  y con menos contraste que el resto. Es lo que evita que el tablero se convierta
  en una mancha.

### 10.3 Vocabulario de feedback por transformación

Todo el movimiento del juego sale de estas herramientas, ninguna de las cuales
requiere arte nuevo:

| Evento | Expresión visual |
|---|---|
| Avance | Deslizamiento con leve balanceo, como pieza empujada por una mano |
| Ataque cuerpo a cuerpo | Empujón brusco hacia el objetivo y regreso a posición |
| Ataque a distancia | Retroceso leve de la pieza más un proyectil simple |
| Impacto recibido | Retroceso, flash blanco y sacudida corta |
| Daño acumulado | Inclinación progresiva; una unidad malherida se ve tambaleante |
| Muerte | La pieza cae de costado y se desvanece |
| Despliegue | La pieza baja desde arriba y aterriza con un pequeño rebote |

La inclinación por daño acumulado es el detalle más valioso: comunica el estado
de la batalla de un vistazo sin depender de barras de vida.

### 10.4 Legibilidad

Requisito de diseño, no objetivo estético.

- **Silueta primero.** Cada rol debe ser reconocible en negro sobre blanco. Si dos
  unidades comparten silueta, una de las dos está mal diseñada. Hacer este test
  con cada sprite apenas se dibuja.
- **Paleta chica y compartida**, 16 a 24 colores para todo el juego.
- **Distinción de bando por tinte**, mismo sprite con paleta desplazada.
- Barras de vida legibles a resolución base, complementarias a la inclinación.

### 10.5 Tablero y entorno

Superficie de tablero con textura simple y grilla sutil, visible durante el
despliegue y atenuada durante el combate. Uno o dos tableros como máximo. Sin
parallax, sin props animados, sin clima.

Opcional y al final de la lista: sugerir el marco de mesa con detalles fijos en
los bordes, como un borde de madera o fichas de puntuación.

### 10.6 Nota para el README

El enfoque se explica como decisión de diseño, no como limitación: el proyecto
invierte su tiempo en simulación, counters y balanceo, y adopta una presentación
de wargame de mesa porque refuerza la lectura de las unidades como piezas
colocables. Aclarar qué assets son propios y cuáles no.

## 11. Datos y balanceo

Todo en archivos de datos:

- Definición de unidades: costo, vida, daño, cadencia, alcance, velocidad, regla
  de targeting, regla de movimiento, modificadores de daño por tipo enemigo.
- Matriz de counters, multiplicadores de daño tipo contra tipo.
- Curva de oro por batalla.
- Pool de recompensas.
- Definición de las doce batallas.

Herramienta acompañante: un runner headless que simula N combates de cada batalla
contra varias composiciones candidatas y reporta tasas de victoria. Con eso el
documento de balance deja de ser una opinión y pasa a ser evidencia.

## 12. Fuera de alcance (v1)

- Cualquier intervención del jugador durante el combate.
- Órdenes por turno, patrones de movimiento discretos, elección de objetivo en
  vivo. Todo eso es otro género.
- Multijugador, matchmaking, backend.
- Tienda aleatoria y economía de reroll.
- Unidades que suben de nivel juntando copias.
- Generación procedural de batallas.
- Objetos, equipamiento, inventario.
- Más de un roster o facción.
- Meta-progresión entre campañas, desbloqueos.
- Narrativa.
- Obstáculos y terreno en el tablero.

## 13. Hitos sugeridos

- Semana 1: simulación determinista por ticks con dos unidades peleando. Sin arte.
- Semana 2: roster de ocho unidades, matriz de counters, reglas de targeting y de
  movimiento.
- Semana 3: fase de reclutamiento y despliegue sobre grilla, con oro.
- Semana 4: runner headless de balance, primera pasada de tuning.
- Semana 5: las doce batallas, recompensas y estructura de campaña con vidas.
- Semana 6: arte de las ocho miniaturas, tablero, vocabulario de feedback.
- Semana 7: interfaz, resumen post-batalla, balanceo con datos del runner,
  primer playtest externo.
- Semana 8: revisión de la regla de vidas según el playtest, documento de
  encuentros, README, video.

## 14. Riesgos conocidos

- **El determinismo es difícil de retrofitear.** Si no se implementa desde el
  primer día, después cuesta muchísimo. Semana uno.
- **El balance es el proyecto.** Es tentador seguir agregando unidades en vez de
  tunear las ocho que hay. Ocho es un techo duro; toda idea de unidad nueva va a
  v2.
- **La deriva de género es el riesgo de diseño principal.** Ya pasó una vez
  durante la conceptualización. Cada vez que aparezca la idea de dar órdenes
  durante el combate, releer la sección 2.
- **Las vidas limitadas pueden expulsar al evaluador.** Ver 7.3. Tener las
  mitigaciones listas antes del primer playtest, no después.
- **El arte puede crecer sin que te des cuenta.** La tentación de agregar un ciclo
  de caminata, después uno de ataque, después uno de muerte por unidad, es la
  forma más común de perder un mes. Si aparece la necesidad de un frame nuevo, la
  respuesta correcta casi siempre es una transformación más.
- **El marco de mesa tiene que ser consistente.** Si algunas unidades se ven como
  piezas y otras como personajes animados, el juego pasa a parecer inacabado en
  vez de estilizado. Es todo o nada.
- **Legibilidad con enjambres.** Los milicianos como unidades individuales son un
  riesgo real de saturación visual. El tope de 18 y el tratamiento de contraste de
  10.2 son las mitigaciones; verificarlas en pantalla apenas haya sprites.

---

# PARTE II — Diseño técnico y producción

Versión 0.3. Asume las convenciones de `00_CONVENCIONES_comunes.md`.

## 15. Arquitectura

Tres capas, con dependencias en una sola dirección.

```
src/
├── core/
│   ├── unit_definition.gd        # Resource: stats de una unidad
│   ├── counter_matrix.gd         # Resource: multiplicadores tipo vs tipo
│   ├── encounter_definition.gd   # Resource: ejército enemigo de una batalla
│   ├── reward_definition.gd      # Resource: una recompensa
│   ├── campaign_config.gd        # Resource: curva de oro, vidas, orden
│   └── registry.gd               # autoload: carga y expone todo por id
├── sim/
│   ├── battle_sim.gd             # motor de simulación, sin nodos visuales
│   ├── sim_unit.gd               # estado de una unidad en simulación
│   ├── targeting.gd              # implementación de reglas de targeting
│   ├── movement.gd               # implementación de reglas de movimiento
│   ├── damage.gd                 # cálculo de daño con counters
│   └── sim_events.gd             # cola de eventos que emite la simulación
├── presentation/
│   ├── battle_view.gd            # consume eventos, dibuja el tablero
│   ├── piece_view.gd             # una miniatura: transformaciones y feedback
│   └── vfx/
├── ui/
│   ├── deploy_screen.gd
│   ├── reward_screen.gd
│   └── debug_overlay.gd
└── tools/
    └── headless_runner.gd        # corre simulaciones sin ventana
```

**`sim/` no importa nada de `presentation/` ni de `ui/`.** Esta regla es la que
hace posible el runner headless. Si se rompe, el proyecto pierde su mejor
entregable.

### 15.1 Contrato entre simulación y presentación

La simulación no mueve sprites. Produce una **cola de eventos** que la
presentación consume y traduce a transformaciones.

```gdscript
# sim_events.gd
enum EventType {
    UNIT_SPAWNED,
    UNIT_MOVED,
    UNIT_ATTACKED,
    UNIT_DAMAGED,
    UNIT_DIED,
    BATTLE_ENDED,
}
```

Cada evento lleva el tick en que ocurrió, los ids involucrados y los datos
mínimos. La presentación puede reproducir la batalla a cualquier velocidad, o no
reproducirla en absoluto, que es lo que hace el runner headless.

Beneficio adicional: permite implementar repetición de batalla y velocidad x2 sin
tocar la simulación.

### 15.2 Bucle de simulación

```
tick fijo: 30 ticks por segundo lógico

por tick:
  1. recolectar unidades vivas, en orden estable por id
  2. para cada unidad:
     a. resolver objetivo según su regla de targeting
     b. si el objetivo está en rango y el cooldown lo permite: atacar
     c. si no: mover según su regla de movimiento
  3. aplicar daño acumulado del tick (resolución simultánea)
  4. resolver muertes
  5. verificar condición de fin
```

**Punto de diseño**: el daño se aplica al final del tick, no durante. Esto evita
que el orden de iteración afecte el resultado y permite que dos unidades se maten
mutuamente en el mismo tick, lo cual es correcto y deseable.

Límite de seguridad: si la batalla supera N ticks sin resolverse, se declara
empate y se registra. Un empate es un bug de balance, no un resultado válido.

## 16. Esquemas de datos

### 16.1 UnitDefinition

| Campo | Tipo | Notas |
|---|---|---|
| `id` | StringName | Estable, nunca cambia |
| `display_name` | String | Localizable |
| `role` | enum | SWARM, AOE, RANGED, FLANKER, TANK, SINGLE_TARGET, SUPPORT, ANTI_SUPPORT |
| `cost` | int | Oro |
| `max_health` | float | |
| `damage` | float | Por golpe |
| `attack_interval` | float | Segundos entre ataques |
| `range_cells` | float | Alcance en celdas |
| `move_speed` | float | Celdas por segundo |
| `targeting_rule` | enum | NEAREST, FARTHEST, LOWEST_HP, HIGHEST_COST, SUPPORT_FIRST |
| `movement_rule` | enum | FRONTAL, HOLD, FLANK, INFILTRATE, FOLLOW |
| `aoe_radius` | float | 0 si es objetivo único |
| `sprite` | Texture2D | |
| `armor` | float | Reducción plana; 0 en la mayoría |
| `percent_damage` | bool | Si el daño escala con vida máxima del objetivo |

### 16.2 CounterMatrix

Matriz 8x8 de multiplicadores de daño, rol atacante contra rol defensor.
Valores por defecto 1.0, con las excepciones que definen el sistema.

Recomendación fuerte: **la matriz debe tener pocas entradas distintas de 1.0.**
Si todo interactúa con todo, nada es legible. Apuntar a entre 10 y 14 celdas
modificadas sobre 64.

### 16.3 EncounterDefinition

| Campo | Tipo |
|---|---|
| `id` | StringName |
| `index` | int (1-12) |
| `enemy_units` | Array de `{unit_id, cell}` |
| `player_gold` | int |
| `design_note` | String multilínea: qué enseña esta batalla |

El campo `design_note` vive en el dato, no en un documento aparte, para que no se
desincronice. El documento de encuentros se genera a partir de estos campos.

### 16.4 Formato de guardado de campaña

```json
{
  "format_version": 1,
  "current_encounter": 5,
  "lives_remaining": 2,
  "rewards_taken": ["gold_boost_1", "archer_range"],
  "rng_seed": 918273
}
```

## 17. Números de arranque

Estos valores son un punto de partida para la primera sesión de tuning, no
definiciones. Se esperan cambios grandes.

Unidad de referencia: **Miliciano**, costo 1.

| Unidad | Costo | Vida | Daño | Intervalo | Alcance | Velocidad |
|---|---|---|---|---|---|---|
| Milicianos | 1 | 40 | 6 | 1.0 | 1 | 1.2 |
| Cañón de asedio | 5 | 90 | 20 (área 1.5) | 2.5 | 4 | 0.5 |
| Arqueros | 3 | 45 | 12 | 1.2 | 6 | 0.9 |
| Jinetes | 4 | 110 | 14 | 1.0 | 1 | 2.2 |
| Guardia pesada | 4 | 320 | 7 | 1.4 | 1 | 0.7 |
| Caballero | 5 | 160 | 38 | 1.6 | 1 | 1.0 |
| Clériga | 4 | 70 | cura 14 (área 2) | 2.0 | 3 | 0.9 |
| Asesino | 4 | 85 | 30 | 1.1 | 1 | 1.8 |

Multiplicadores de counter sugeridos para la primera pasada:

- Área contra enjambre: x1.0 base, pero el daño en área golpea a varios; el
  counter emerge de la geometría, no de un multiplicador. **Preferir counters
  emergentes a multiplicadores explícitos siempre que se pueda.**
- Objetivo único contra tanque: x1.6
- Enjambre contra objetivo único: emergente (saturación de objetivos).
- Alcance largo contra área: emergente (alcance superior).
- Flanqueo contra alcance largo: x1.4
- Enjambre contra flanqueo: emergente (bloqueo por número).
- Antisoporte contra soporte: x2.0
- Tanque contra antisoporte: x0.6 recibido

**Principio de diseño**: un counter que emerge de la geometría y el
comportamiento es más satisfactorio y más legible que uno impuesto por una tabla.
La matriz existe para reforzar relaciones que ya se sienten, no para crearlas de
la nada.

Curva de oro por batalla: `8 + (indice - 1) * 3`, es decir de 8 a 41.
Ajustar tras la primera medición del runner.

## 18. Runner headless de balance

Es el entregable técnico diferencial del proyecto.

### 18.1 Qué hace

Corre desde línea de comandos, sin ventana:

```
godot --headless --script res://src/tools/headless_runner.gd -- \
      --encounters all --trials 500 --out results.csv
```

Para cada batalla, prueba un conjunto de composiciones candidatas del jugador y
reporta:

- Tasa de victoria por composición.
- Duración media de la batalla en ticks.
- Unidades supervivientes promedio.
- Daño hecho y recibido por rol.
- Empates detectados.

### 18.2 Generación de composiciones candidatas

No hace falta una IA. Alcanza con:

- **Composiciones monotipo**: todo el oro en una sola unidad. Ocho candidatas.
  Si alguna monotipo gana más del 60% de las batallas, esa unidad está rota.
- **Composiciones de dos tipos** en proporciones fijas.
- **Composiciones diseñadas a mano** que representen la "solución esperada" de
  cada batalla.

### 18.3 Cómo se lee el resultado

Criterios de balance saludable:

- Ninguna composición monotipo supera el 60% de victorias globales.
- La solución esperada de cada batalla gana entre el 70% y el 95%. Menos de 70%
  es demasiado azarosa; más de 95% es trivial.
- Ninguna unidad tiene tasa de aparición cero en composiciones ganadoras.
- Ninguna unidad aparece en más del 80% de las composiciones ganadoras.

Estos números van a la bitácora de balance con fecha y cambios aplicados.

## 19. Plan de pruebas

Tests automatizados, en orden de escritura:

1. **Determinismo**: misma semilla y despliegue, hash de estado final idéntico
   en dos corridas.
2. **Daño**: aplicación correcta de multiplicadores de counter y armadura.
3. **Targeting**: cada regla elige el objetivo esperado en escenarios armados.
4. **Movimiento**: FLANK llega al borde antes de avanzar; INFILTRATE ignora
   unidades intermedias; HOLD se detiene al tener objetivo en rango.
5. **Terminación**: ninguna batalla del set de 12 supera el límite de ticks en
   1000 corridas con composiciones variadas.
6. **Serialización**: guardar y cargar campaña preserva el estado.

## 20. Backlog por hito

### Hito 1 — Simulación (semana 1)
- [ ] Estructura de repo y convenciones aplicadas
- [ ] `UnitDefinition` como Resource con dos unidades de prueba
- [ ] Bucle de tick fijo con acumulador
- [ ] RNG con semilla inyectada
- [ ] Targeting NEAREST
- [ ] Movimiento FRONTAL
- [ ] Cálculo de daño y muerte
- [ ] Cola de eventos de simulación
- [ ] Test de determinismo pasando
- **Terminado cuando**: dos ejércitos de prueba pelean en consola, sin gráficos,
  y la misma semilla da el mismo resultado dos veces.

### Hito 2 — Roster y reglas (semana 2)
- [ ] Las ocho unidades definidas como Resources
- [ ] Las cinco reglas de targeting
- [ ] Las cinco reglas de movimiento
- [ ] Daño en área
- [ ] Curación de soporte
- [ ] `CounterMatrix` aplicada en el cálculo de daño
- [ ] Tests de targeting y movimiento pasando
- **Terminado cuando**: cualquier combinación de las ocho unidades pelea
  correctamente y las reglas de movimiento son distinguibles en el log.

### Hito 3 — Despliegue (semana 3)
- [ ] Grilla de despliegue y colocación
- [ ] Economía de oro y reclutamiento
- [ ] Tope de 18 unidades
- [ ] Visualización mínima del tablero, con cuadrados de color
- [ ] Reproducción de la cola de eventos en pantalla
- **Terminado cuando**: se puede armar un ejército con el mouse y ver la batalla
  resolverse, aunque sea feo.

### Hito 4 — Balance (semana 4)
- [ ] Runner headless funcionando
- [ ] Exportación de definiciones a CSV y reimportación
- [ ] Primera medición completa
- [ ] Primera iteración de tuning documentada en `docs/balance.md`
- **Terminado cuando**: existe un CSV de resultados y una entrada de bitácora con
  un cambio justificado por datos.

### Hito 5 — Campaña (semana 5)
- [ ] Las doce `EncounterDefinition`
- [ ] Pool de 15-20 recompensas
- [ ] Pantalla de recompensa
- [ ] Vidas, derrota y reinicio de campaña
- [ ] Guardado y carga
- [ ] Modo escaramuza (acceso directo a batallas vistas)
- **Terminado cuando**: se puede jugar una campaña completa de principio a fin.

### Hito 6 — Arte (semana 6)
- [ ] Ocho sprites de miniatura en tres cuartos
- [ ] Test de silueta pasado para las ocho
- [ ] Shader de tinte por bando
- [ ] Tablero
- [ ] Vocabulario completo de feedback por transformación
- [ ] Efectos de impacto y muerte
- [ ] Audio mínimo
- **Terminado cuando**: un espectador distingue los ocho roles sin leer la UI.

### Hito 7 — Interfaz y pulido (semana 7)
- [ ] Panel de ejército enemigo
- [ ] Panel de reclutamiento con tooltips completos
- [ ] Resumen post-batalla por rol
- [ ] Overlay de debug
- [ ] Velocidad x2 y repetición
- [ ] Primer playtest externo, tres personas
- **Terminado cuando**: un tester entiende por qué perdió sin que se lo expliques.

### Hito 8 — Cierre (semana 8)
- [ ] Revisión de la regla de vidas según playtest
- [ ] Segunda y tercera iteración de balance documentadas
- [ ] `docs/encounters.md` generado
- [ ] Build web publicada en itch.io
- [ ] README, GIF, video con overlay de debug
- [ ] Postmortem
- **Terminado cuando**: el checklist de la sección 17 del documento de
  convenciones está completo.

## 21. Registro de riesgos

| Riesgo | Impacto | Prob. | Mitigación | Señal temprana |
|---|---|---|---|---|
| Determinismo no implementado a tiempo | Crítico | Baja | Semana 1, test primero | El test de hash no existe al cerrar el hito 1 |
| Deriva de género hacia táctico | Alto | Alta | Sección 2 del GDD, releer ante cada idea nueva | Aparece la palabra "orden" o "turno" en el backlog |
| Balance no converge | Alto | Media | Runner desde semana 4, no desde la 7 | Una monotipo gana más del 70% en la primera medición |
| Saturación visual con enjambres | Medio | Media | Tope de 18, contraste reducido | El tablero se lee mal con 15 milicianos |
| Vidas limitadas expulsan al evaluador | Medio | Alta | Modo escaramuza, mitigaciones listas | El tester abandona antes de la batalla 6 |
| Alcance del arte crece | Medio | Media | Un sprite por unidad, sin excepciones | Aparece la palabra "animación" en el backlog |
| Empates en simulación | Bajo | Media | Límite de ticks y registro | El runner reporta empates |

## 22. Preguntas abiertas

Pendientes de definir antes o durante el hito correspondiente:

- Nombre definitivo del proyecto.
- ¿La curación de la clériga puede revivir? (recomendación: no, complica el
  determinismo y la lectura)
- ¿Las recompensas se acumulan sin límite o hay un tope?
- ¿El modo escaramuza permite oro libre o el oro de la batalla original?
- Ambientación visual concreta: ¿miniaturas pintadas realistas, o estilo pieza de
  ajedrez tallada? Afecta el trabajo de sprite.

---

# PARTE III — Estructura de campaña y recompensas

Versión 0.4. Reemplaza la estructura lineal de la sección 6 y amplía la 7.7.

## 23. Mapa de campaña ramificado

### 23.1 Estructura

**Ocho niveles de profundidad, tres rutas.** El jugador avanza de un nivel al
siguiente eligiendo entre los nodos disponibles.

```
            N1
         /  |  \
       N2  N2  N2        <- nivel 2, tres opciones
        \  /|\  /
       N3  N3  N3
          ...
            N8            <- batalla final, única
```

- Nivel 1: nodo único, batalla introductoria común a todos.
- Niveles 2 a 7: tres nodos por nivel. El jugador ve el ejército enemigo de cada
  uno antes de elegir.
- Nivel 8: nodo único, batalla final.

**Total de batallas a diseñar: 20** (1 + 6x3 + 1). Una partida atraviesa 8.

Esto es más trabajo que las doce lineales originales, pero cada batalla es un
archivo de datos con un ejército y una nota de diseño, no contenido caro.

### 23.2 Ver antes de elegir

El jugador ve, en el mapa, la composición enemiga de cada nodo disponible y qué
tipo de recompensa ofrece. La elección de ruta es una decisión informada, igual
que el despliegue.

Esto es coherente con la decisión central del proyecto (sección 5): información
completa antes de comprometerse.

### 23.3 El problema de la curva de enseñanza

Con rutas, no se puede garantizar que el jugador haya visto la batalla que enseña
un counter antes de encontrarse con él. La solución:

**Cada nivel enseña o exige lo mismo en sus tres nodos, con distinto sabor.**

| Nivel | Qué enseña o exige | Nodos |
|---|---|---|
| 1 | Lectura básica de composición | Único: enemigo monotipo simple |
| 2 | Un counter fundamental cada uno | Enjambre / Alcance largo / Tanque |
| 3 | El counter complementario | Área / Flanqueo / Objetivo único |
| 4 | Dos tipos combinados | Tres combinaciones distintas |
| 5 | Posicionamiento aprovechado | Retaguardia protegida / Flancos / Línea |
| 6 | Soporte y antisoporte | Tres presentaciones |
| 7 | Composiciones que castigan la respuesta obvia | Tres trampas distintas |
| 8 | Todo junto | Único: ejército completo |

**Regla de diseño**: si un nodo del nivel 4 requiere un conocimiento que solo
enseña un nodo específico del nivel 3, ese nodo está mal diseñado. Cada nivel
debe ser resoluble con lo aprendido en cualquier ruta previa.

Esto es más restrictivo que un diseño lineal y es la razón por la que el mapa
ramificado cuesta más. Vale la pena documentarlo en `docs/encounters.md` como
decisión de diseño, porque demuestra que entendés el problema.

### 23.4 Nodos élite

En cada nivel del 2 al 7, **uno de los tres nodos es élite**: ruta arriesgada.

Un nodo élite tiene:
- El mismo tipo de ejército que sus pares, pero con **modificadores aplicados** a
  las unidades enemigas.
- Una recompensa exclusiva (sección 24.3).

**Implementación barata**: un élite es una `EncounterDefinition` con un array de
`unit_modifiers`, que es la misma estructura que usa el sistema de recompensas.
No hay sistema nuevo.

```gdscript
# añadido a EncounterDefinition
@export var is_elite: bool = false
@export var enemy_modifiers: Array[ModifierDefinition] = []
@export var exclusive_reward_pool: Array[StringName] = []
```

Modificadores élite de ejemplo:
- Veterano: +30% vida a todas las unidades enemigas.
- Fanático: +25% daño, -20% vida.
- Disciplinado: las unidades enemigas ignoran el primer stagger.
- Numeroso: +3 unidades enemigas sin cambiar la composición.

### 23.5 Vidas y ruta

Las tres vidas de la sección 7.3 se mantienen. La interacción con las rutas es
interesante: perder en un nodo élite cuesta lo mismo que perder en uno normal,
así que la decisión de riesgo es real pero acotada.

Ajuste sobre 7.3: al perder una vida, el jugador **reintenta el mismo nodo**, no
vuelve al mapa. No puede esquivar un élite difícil después de verlo perder.

## 24. Sistema de recompensas

### 24.1 Principio rector

**La ruta arriesgada no da recompensas mejores, da recompensas distintas.**

Si lo exclusivo fuera simplemente más fuerte, la ruta élite dejaría de ser una
elección y pasaría a ser lo obligatorio para cualquiera que juegue bien, y la
ruta segura se volvería contenido muerto. La diferencia es de naturaleza:

- **Ruta segura**: mejoras de números. Predecibles, acumulativas, seguras.
- **Ruta élite**: modificadores globales potentes con contrapartida. Cambian cómo
  jugás, no cuánto pegás.

### 24.2 Recompensas normales (ruta segura)

Se ofrecen tres, se elige una. Pool de 12 a 15.

Categorías:
- **Oro**: +4 de oro para todas las batallas restantes.
- **Mejora de unidad**: una unidad concreta gana +20% de vida, o +15% de daño, o
  +1 de alcance, o -1 de costo.
- **Mejora de rol**: todas las unidades de un rol ganan un bonus menor.

Son aburridas a propósito. Su virtud es que no obligan a replantear el ejército.

### 24.3 Recompensas exclusivas (ruta élite)

Se ofrecen dos, se elige una. Pool de 8 a 10. Todas tienen contrapartida
explícita.

Ejemplos de arranque:

| Nombre | Beneficio | Contrapartida |
|---|---|---|
| Leva forzosa | +8 de oro por batalla | Tope de unidades baja de 18 a 14 |
| Fanatismo | +40% de daño a todas tus unidades | -30% de vida a todas |
| Formación cerrada | Tus unidades reciben -25% de daño en área | Pierden 30% de velocidad |
| Vanguardia | Tus unidades de flanqueo son gratis | Tus unidades de alcance largo cuestan el doble |
| Juramento | Una unidad elegida duplica sus estadísticas | No podés reclutar más de tres tipos distintos |
| Reservas | Empezás cada batalla con dos milicianos gratis | Los milicianos cuestan +1 |
| Máquinas de guerra | El cañón gana +2 de alcance y +50% de daño de área | La clériga no puede reclutarse más |
| Marcha forzada | Todas tus unidades ganan +40% de velocidad | -20% de vida |

**Criterio de diseño**: cada exclusiva debe hacer viable un estilo de ejército que
antes no lo era, y debe cerrar otro. Si una exclusiva es simplemente buena, está
mal diseñada.

### 24.4 Validación con el runner

El runner de balance (sección 18) debe extenderse para medir recompensas:

- Simular campañas completas con cada exclusiva tomada al inicio.
- Reportar tasa de finalización de campaña por recompensa.
- **Ninguna exclusiva debe subir la tasa de finalización más de 15 puntos ni
  bajarla más de 15.** Fuera de esa banda, está rota o es una trampa.
- Ninguna combinación de dos exclusivas debe superar el 90% de finalización.

Este análisis es material de primera para la bitácora de balance.

### 24.5 Acumulación

El jugador toma 7 recompensas por campaña (una por nivel del 2 al 8). Con tres
niveles élite máximos por ruta, puede acumular hasta 3 exclusivas.

Punto abierto: ¿se permite acumular exclusivas con contrapartidas contradictorias
(Fanatismo y Marcha forzada, ambas reduciendo vida)? Recomendación: sí, y que el
jugador se haga responsable. Las combinaciones autodestructivas son parte del
aprendizaje. Verificar con el runner que no exista una combinación
inmediatamente letal.

## 25. Impacto en el plan

Cambios respecto de la Parte II:

- **Hito 5 crece**: 20 batallas en vez de 12, más el sistema de mapa, más dos
  pools de recompensas. Estimar dos semanas, no una.
- **Hito 4 suma la extensión del runner** para simular campañas completas, no
  solo batallas sueltas.
- Duración total realista: **9-10 semanas**, no 8.

Recorte disponible si el plazo aprieta, en orden de preferencia:
1. Reducir a 6 niveles de profundidad (14 batallas).
2. Reducir el pool exclusivo a 6.
3. Volver a lineal con nodos élite opcionales.

El punto 3 es el recorte de emergencia: conserva la idea de riesgo y recompensa
sin el costo del mapa.

## 26. Riesgos actualizados

| Riesgo | Impacto | Prob. | Mitigación | Señal temprana |
|---|---|---|---|---|
| Curva de enseñanza rota por las rutas | Alto | Alta | Regla de equivalencia por nivel (23.3) | Un tester queda trabado en el nivel 4 |
| Recompensa exclusiva dominante | Alto | Alta | Banda de ±15 puntos medida con el runner | Los testers siempre eligen la misma |
| Ruta segura vuelta contenido muerto | Medio | Media | Recompensas distintas, no peores | Nadie elige nodos normales |
| 20 batallas sin diseñar a tiempo | Alto | Media | Recortes escalonados de la sección 25 | El hito 5 pasa de dos semanas |
| Combinación de exclusivas rota | Medio | Media | Simulación de pares en el runner | Una campaña se gana sin esfuerzo |
