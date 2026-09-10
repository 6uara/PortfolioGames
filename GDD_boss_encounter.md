# GDD base — Boss encounter (nombre provisorio: "La Sala")

Versión 0.1 — documento base, sujeto a revisión durante prototipado.
Engine: Godot 4.7 / GDScript.
Duración objetivo: 5-7 semanas.
Rol que demuestra: combat design / encounter design / technical design.

---

## 1. Pitch

Un único enfrentamiento contra un jefe en una sala cerrada, en tercera persona,
con combate cuerpo a cuerpo de tipo souls-like. Sin exploración, sin progresión,
sin inventario. El jugador aparece en la sala, pelea, muere o gana, y reinicia
al instante.

## 2. Objetivo del proyecto (portfolio)

Demostrar:

- Diseño de encuentro: telegraphs, ritmo, escalada de dificultad por fases.
- Feel de combate: timings, i-frames, cancelaciones, feedback de impacto.
- Máquinas de estado y árboles de decisión de IA.
- Capacidad de exponer y depurar sistemas de combate (herramientas de debug).

El criterio de éxito es que un jugador que conoce el género diga "esto se siente
bien y es justo".

## 3. Loop

Muerte y reintento inmediato. Sin pantallas de carga, sin menú intermedio.
El objetivo es que el jugador pueda hacer 30 intentos en 20 minutos y sentir que
mejora cada vez.

## 4. Personaje jugable

### 4.1 Acciones

- Movimiento en 8 direcciones con lock-on al jefe.
- Rodar (dodge) con i-frames.
- Ataque ligero (combo de 2-3 golpes).
- Ataque pesado (más daño, más recovery).
- Bloqueo o parry (elegir uno; recomendado parry, es más interesante de diseñar).
- Curación con cargas limitadas (3 usos por intento).

### 4.2 Stamina

Todas las acciones excepto el movimiento base consumen stamina. La stamina se
regenera con delay tras la última acción. Es el recurso que impide el spam de
rodar y el que define el ritmo del combate.

### 4.3 Números de arranque (a tunear)

Estos valores son un punto de partida, no una definición final:

- i-frames del rodar: 12 frames sobre 40 de animación total.
- Recovery del ataque pesado: 25 frames.
- Ventana de parry: 8 frames.
- Regeneración de stamina: arranca 30 frames después de la última acción.

Todo esto debe ser editable sin recompilar.

## 5. El jefe

### 5.1 Estructura de fases

Tres fases, disparadas por umbrales de vida (100-66%, 66-33%, 33-0%).

Fase 1 — Enseñanza:
- 3 ataques básicos, telegraphs largos y claros.
- Ritmo pausado, pausas generosas entre ataques.
- Función: que el jugador aprenda el vocabulario de movimientos.

Fase 2 — Combinación:
- Los ataques de fase 1 se encadenan en combos de 2-3.
- Se suman 2 ataques nuevos, uno de ellos con área.
- Telegraphs más cortos.
- Función: exigir lectura, no memoria.

Fase 3 — Presión:
- Se suma un ataque de alto compromiso con telegraph muy largo y castigo alto
  si falla (ventana de oportunidad del jugador).
- Ritmo más agresivo, menos pausas.
- Función: probar todo lo aprendido bajo presión.

### 5.2 Reglas de diseño de ataques

Cada ataque del jefe debe definir explícitamente:

- Duración de telegraph (frames antes del hitbox activo).
- Frames activos del hitbox.
- Recovery (ventana de castigo para el jugador).
- Daño.
- Distancia de alcance y ángulo.
- Si es esquivable rodando, parriable, o ambas.

Principio: todo ataque tiene una respuesta correcta y una ventana de castigo.
Nada es aleatorio en el sentido de injusto.

### 5.3 Selección de ataques (IA)

No es un random puro. El jefe elige según:

- Distancia al jugador (cerca / media / lejos define el set disponible).
- Cooldown por ataque, para evitar repetición inmediata.
- Peso por fase.
- Regla anti-frustración: no repetir el mismo ataque más de dos veces seguidas.

## 6. La sala

Un único espacio circular o rectangular, cerrado, plano. Sin obstáculos ni
cobertura, para que el combate sea puro. Tamaño suficiente para que el jugador
pueda romper distancia pero no huir indefinidamente.

## 7. Arte y animación

Punto declarado del proyecto: el arte no es propio.

- Modelo del jefe y del jugador: assets comprados o gratuitos ya riggeados.
- Animaciones: Mixamo o el pack que venga con el modelo.
- Efectos: partículas simples de Godot, sin VFX elaborado.

El tiempo se invierte en hitboxes, timings y comportamiento, no en modelado.
Esto se aclara de forma directa en el README.

Nota: elegí el modelo del jefe temprano, porque el set de animaciones disponible
condiciona qué ataques podés diseñar. No diseñes los ataques primero y busques
animaciones después.

## 8. Herramientas de debug (parte central del entregable)

Un modo debug activable en runtime que muestre:

- Hitboxes y hurtboxes con color por estado.
- Frames activos del ataque en curso.
- Estado actual de la máquina de estados del jefe y del jugador.
- Stamina y i-frames del jugador visualizados.
- Log del último ataque elegido por la IA y por qué.

Esto es lo que se graba para el video de portfolio. Un video con overlay de debug
comunica más habilidad técnica que un video de gameplay limpio.

## 9. Feedback e impacto

- Hit stop en impactos.
- Screen shake acotado.
- Indicador claro de daño recibido.
- Sonido diferenciado por tipo de ataque y por parry exitoso.

El parry exitoso debe sentirse desproporcionadamente bien. Es el momento que
vende el combate.

## 10. Fuera de alcance (v1)

- Más de un jefe.
- Armas alternativas, builds, estadísticas del jugador.
- Menús, opciones, guardado.
- Cinemáticas o narrativa.
- Multijugador.

## 11. Hitos sugeridos

- Semana 1: controlador del jugador, cámara con lock-on, rodar con i-frames.
- Semana 2: ataques del jugador, stamina, hitboxes y sistema de daño.
- Semana 3: jefe con fase 1 completa y máquina de estados.
- Semana 4: fases 2 y 3, selección de ataques por IA.
- Semana 5: herramientas de debug y feedback de impacto.
- Semana 6: tuning de timings, playtesting con terceros, video y README.

## 12. Riesgos conocidos

- El tuning de combate es donde se va el tiempo real. Reservá al menos dos
  semanas efectivas para eso; un combate técnicamente correcto pero mal tuneado
  no demuestra nada.
- Conseguir playtesters es imprescindible acá. Tu propia percepción de la
  dificultad va a estar completamente distorsionada después de 200 intentos.

---

# PARTE II — Diseño técnico y producción

Versión 0.2. Asume las convenciones de `00_CONVENCIONES_comunes.md`.

## 13. Arquitectura

```
src/
├── core/
│   ├── attack_definition.gd      # Resource: un ataque, con sus frames
│   ├── player_config.gd          # Resource: stats y timings del jugador
│   ├── boss_config.gd            # Resource: fases, pesos, cooldowns
│   └── frame_data.gd             # utilidades de conversión frames <-> segundos
├── combat/
│   ├── hitbox.gd                 # Area3D con metadata de ataque
│   ├── hurtbox.gd
│   ├── damage_resolver.gd        # única fuente de verdad del daño
│   └── invulnerability.gd        # gestión de i-frames
├── player/
│   ├── player_controller.gd
│   ├── player_state_machine.gd
│   └── states/                   # un archivo por estado
├── boss/
│   ├── boss_controller.gd
│   ├── boss_state_machine.gd
│   ├── attack_selector.gd        # la "IA": elección ponderada
│   └── states/
├── camera/
│   └── lock_on_camera.gd
└── debug/
    └── combat_debug_overlay.gd
```

### 13.1 Máquinas de estado

Patrón: **State pattern con nodos**. Cada estado es un nodo hijo de la máquina,
con `enter()`, `exit()`, `update(delta)` y `physics_update(delta)`.

Estados del jugador:
`Idle`, `Move`, `Roll`, `LightAttack1`, `LightAttack2`, `LightAttack3`,
`HeavyAttack`, `Parry`, `ParrySuccess`, `Hurt`, `Heal`, `Dead`.

Estados del jefe:
`Idle`, `Reposition`, `Telegraph`, `Active`, `Recovery`, `PhaseTransition`,
`Stagger`, `Dead`.

**Regla**: las transiciones son explícitas y viven en el estado de origen. Nada
de una función gigante de transiciones centralizada.

### 13.2 Frame data como dato

El combate souls-like es un juego de frames. Todos los timings viven en un
Resource, expresados en frames a 60 fps y convertidos internamente.

```gdscript
class_name AttackDefinition
extends Resource

@export var id: StringName
@export var animation_name: StringName
@export var startup_frames: int = 20      # telegraph
@export var active_frames: int = 6        # hitbox encendida
@export var recovery_frames: int = 24     # ventana de castigo
@export var damage: float = 30.0
@export var poise_damage: float = 10.0
@export var range_meters: float = 2.5
@export var angle_degrees: float = 90.0
@export var can_be_parried: bool = true
@export var can_be_rolled: bool = true
@export var tracks_during_startup: bool = true
@export var min_distance: float = 0.0     # rango de uso
@export var max_distance: float = 3.0
@export var cooldown_seconds: float = 3.0
@export var phase_weights: PackedFloat32Array  # peso por fase
```

**El desacople entre frame data y animación es crítico.** La animación es
decorativa; la verdad está en los frames declarados. Si la animación no coincide,
se ajusta la animación, no los datos.

### 13.3 Resolución de daño

Una sola función resuelve todo el daño del juego. Nada de aplicar daño desde
scripts de estado.

```
resolve_damage(source, target, attack_definition):
  1. si target tiene i-frames activos -> ignorar, emitir evento DODGED
  2. si target está en ventana de parry y attack.can_be_parried -> PARRIED
  3. aplicar daño, aplicar poise damage
  4. si poise acumulado supera umbral -> STAGGER
  5. emitir evento DAMAGED con todos los datos
```

Los eventos alimentan tanto el feedback visual como el overlay de debug.

## 14. Números de arranque

Todos en frames a 60 fps. Primera pasada, se esperan cambios grandes.

### 14.1 Jugador

| Parámetro | Valor |
|---|---|
| Vida | 100 |
| Stamina máxima | 100 |
| Regeneración de stamina | 30/seg tras 30 frames de inactividad |
| Rodar: total | 40 frames |
| Rodar: i-frames | del 5 al 17 (13 frames) |
| Rodar: costo | 25 stamina |
| Ataque ligero 1: startup / active / recovery | 12 / 4 / 16 |
| Ataque ligero 2 | 10 / 4 / 18 |
| Ataque ligero 3 | 14 / 5 / 28 |
| Ataque ligero: daño / costo | 18 / 20 stamina |
| Ataque pesado: startup / active / recovery | 26 / 6 / 30 |
| Ataque pesado: daño / costo | 42 / 35 stamina |
| Parry: startup / ventana / recovery | 3 / 8 / 22 |
| Parry: costo | 15 stamina |
| Curación: total / invulnerable | 70 frames / ninguno |
| Curación: cantidad / cargas | 45 / 3 |
| Ventana de cancelación a rodar tras ataque | últimos 8 frames de recovery |

### 14.2 Jefe

Vida total 1200, dividida en tres fases de 400.

**Fase 1 — Enseñanza**

| Ataque | Startup | Active | Recovery | Daño | Rango | Notas |
|---|---|---|---|---|---|---|
| Tajo horizontal | 34 | 5 | 40 | 28 | 0-3m | Parriable, rodable |
| Estocada | 30 | 4 | 46 | 32 | 2-5m | Cierra distancia |
| Pisotón | 40 | 8 | 44 | 25 | 0-2.5m | Área, no parriable |

Pausa entre ataques: 60-90 frames.

**Fase 2 — Combinación**

Se suman:

| Ataque | Startup | Active | Recovery | Daño | Rango |
|---|---|---|---|---|---|
| Barrido en giro | 36 | 10 | 52 | 35 | 0-4m |
| Salto con impacto | 48 | 6 | 50 | 40 | 4-9m |

Los tres de fase 1 bajan su startup un 15%. Se habilitan cadenas de 2-3 ataques
con 20-30 frames entre eslabones. Pausa entre cadenas: 45-70 frames.

**Fase 3 — Presión**

Se suma:

| Ataque | Startup | Active | Recovery | Daño | Rango |
|---|---|---|---|---|---|
| Golpe devastador | 90 | 8 | 90 | 75 | 0-5m |

Telegraph larguísimo y castigo enorme si falla: es la ventana de oportunidad
principal del jugador. Pausa entre cadenas: 35-55 frames.

**Principio**: la dificultad sube por reducción de pausas y encadenamiento, no
por inflar daño. Un jefe que mata de dos golpes no es difícil, es frustrante.

### 14.3 Umbral de stagger

Poise del jefe: 100, se regenera 10/seg tras 2 segundos sin recibir daño.
Un stagger abre 90 frames de castigo garantizado.

## 15. Selección de ataques

No es random puro. Algoritmo por decisión:

```
1. filtrar ataques disponibles:
   - fase actual permite el ataque (peso > 0)
   - cooldown cumplido
   - distancia al jugador dentro de [min_distance, max_distance]
   - no es el mismo ataque de las dos últimas veces
2. si no queda ninguno -> estado Reposition
3. elegir por peso, con el RNG sembrado
4. si el jugador está lejos y no hay ataques de largo alcance -> Reposition
```

**Reglas anti-frustración explícitas:**
- No repetir el mismo ataque más de dos veces seguidas.
- No encadenar dos ataques no parriables ni no rodables consecutivos.
- Tras un stagger, pausa forzada de 40 frames antes del siguiente ataque.
- Nunca iniciar un ataque si el jugador está en animación de muerte.

Todas estas reglas se logean en el overlay de debug: qué ataques estaban
disponibles, cuál se eligió y por qué. Ese log es material de video de portfolio.

## 16. Cámara y lock-on

- Lock-on con offset vertical: el jefe queda en el tercio superior de la pantalla
  para que el suelo y las telegrafías de área sean visibles.
- Distancia de cámara variable según la distancia al jefe.
- Sin colisión de cámara con paredes en v1 (la sala es abierta).
- Ruptura de lock-on solo manual.

**Advertencia**: la cámara es el sistema que más silenciosamente arruina un
combate en tercera persona. Reservar tiempo real para tunearla, no dejarla para
el final.

## 17. Feedback e impacto

| Evento | Respuesta |
|---|---|
| Golpe conectado | Hit stop 4 frames, partícula de impacto, sonido con pitch variable |
| Golpe pesado | Hit stop 7 frames, screen shake leve |
| Parry exitoso | Hit stop 12 frames, flash, sonido distintivo, cámara acerca levemente |
| Daño recibido | Viñeta roja, screen shake, sonido de dolor |
| Stagger del jefe | Hit stop 15 frames, cambio de música o silencio |
| Cambio de fase | Pausa, grito, cambio de iluminación |
| Muerte del jugador | Cámara lenta, fundido rápido, reinicio en menos de 2 segundos |

**El reinicio rápido es un requisito de diseño.** Treinta intentos en veinte
minutos es el objetivo; cualquier fricción entre la muerte y el siguiente intento
lo rompe.

## 18. Plan de pruebas

Automatizables:
1. Frame data: cada ataque activa su hitbox exactamente en el frame declarado.
2. I-frames: un ataque que impacta durante la ventana de rodar no produce daño.
3. Parry: un ataque parriable en la ventana produce PARRIED, fuera produce daño.
4. Selección: el selector nunca devuelve un ataque en cooldown ni repetido tres
   veces.
5. Fases: los umbrales de vida disparan la transición exactamente una vez.

Manuales, por checklist:
- Cada ataque del jefe es esquivable rodando en la dirección correcta.
- Cada ataque tiene una ventana de castigo real de al menos 20 frames.
- El jugador nunca muere sin haber tenido una respuesta posible.

## 19. Backlog por hito

### Hito 1 — Movimiento y cámara (semana 1)
- [ ] Estructura de repo, convenciones
- [ ] Modelo del jugador y del jefe elegidos y descargados
- [ ] Controlador de movimiento con física
- [ ] Cámara con lock-on
- [ ] Máquina de estados del jugador con Idle, Move, Roll
- [ ] Rodar con i-frames declarados en datos
- **Terminado cuando**: se puede correr, rodar y mirar al jefe, y los i-frames
  son visibles en el overlay.

### Hito 2 — Combate del jugador (semana 2)
- [ ] Sistema de hitbox y hurtbox
- [ ] Resolver de daño único
- [ ] Combo ligero de tres, ataque pesado
- [ ] Stamina
- [ ] Parry
- [ ] Curación
- [ ] Cancelaciones
- **Terminado cuando**: el jugador puede atacar a un muñeco de prueba y todos los
  timings vienen de datos.

### Hito 3 — Jefe fase 1 (semana 3)
- [ ] Máquina de estados del jefe
- [ ] Los tres ataques de fase 1 con frame data
- [ ] Selector de ataques con cooldowns y distancia
- [ ] Reposicionamiento
- [ ] Vida y muerte
- **Terminado cuando**: el jefe pelea de forma legible y se le puede ganar.

### Hito 4 — Fases 2 y 3 (semana 4)
- [ ] Transiciones de fase
- [ ] Los tres ataques nuevos
- [ ] Sistema de cadenas de ataques
- [ ] Poise y stagger
- [ ] Reglas anti-frustración
- **Terminado cuando**: el combate completo es jugable de principio a fin.

### Hito 5 — Debug y feedback (semana 5)
- [ ] Overlay con hitboxes, frames activos, estados, stamina, i-frames
- [ ] Log de decisión del selector
- [ ] Hit stop, screen shake, partículas
- [ ] Audio
- [ ] Reinicio rápido
- **Terminado cuando**: el video con overlay se puede grabar.

### Hito 6 — Tuning (semanas 6-7)
- [ ] Tres playtests externos como mínimo
- [ ] Al menos tres iteraciones documentadas en `docs/balance.md`
- [ ] Checklist manual de esquivabilidad completo
- [ ] Ajuste de cámara
- **Terminado cuando**: un tester que conoce el género lo describe como justo.

### Hito 7 — Cierre (semana 7-8)
- [ ] Build publicada
- [ ] README, GIF, video con overlay
- [ ] Postmortem
- [ ] Créditos de assets

## 20. Registro de riesgos

| Riesgo | Impacto | Prob. | Mitigación | Señal temprana |
|---|---|---|---|---|
| Animaciones disponibles no soportan los ataques diseñados | Alto | Alta | Elegir modelo y set ANTES de diseñar ataques | Falta una animación para un ataque del GDD |
| Cámara arruina el combate | Alto | Media | Tiempo reservado en hito 6, no al final | Los testers se quejan de "no veo lo que pasa" |
| Percepción de dificultad distorsionada | Alto | Alta | Playtests externos desde el hito 4 | Te parece fácil y nadie más lo pasa |
| Frame data desacoplada de la animación visible | Medio | Media | Overlay de debug desde temprano | El golpe se ve conectar y no hace daño |
| Scope creep a segundo jefe | Medio | Baja | Sección de fuera de alcance | Aparece "otro jefe" en el backlog |

## 21. Preguntas abiertas

- ¿Parry o bloqueo? El GDD recomienda parry; confirmar antes del hito 2.
- ¿El jugador tiene poise propio, o toda interrupción lo afecta?
- ¿Hay un segundo intento inmediato o el jugador vuelve a una zona previa?
  (recomendación: reinicio inmediato en la sala)
- ¿Se muestra la barra de vida del jefe? (recomendación: sí, con marcas de fase)

---

# PARTE III — Armas, sala y presentación

Versión 0.3. Incorpora la elección de arma y las decisiones pendientes.

## 22. Sistema de defensa: parry

Confirmado: **parry, sin bloqueo con escudo.**

Consecuencias que hay que respetar en todo el diseño:

- **Cada ataque del jefe debe estar clasificado como parriable o no parriable, y
  el jugador debe poder distinguirlo por el telegraph antes de comprometerse.**
  Un ataque que parece parriable y no lo es se siente injusto, y ese es el peor
  defecto posible en un combate de este tipo.
- Convención visual: los ataques no parriables llevan un destello distintivo
  durante el startup. Es la solución estándar del género y funciona.
- Sin escudo, el jugador no tiene defensa pasiva. Eso sube la dificultad de base
  y hay que compensarlo con ventanas de esquiva generosas.
- **El parry es idéntico con las tres armas**: misma ventana, mismo startup,
  mismo recovery, mismo costo de stamina. Simplifica el tuning y hace que la
  elección de arma sea puramente ofensiva.

Ventana unificada (repetida de 14.1 para referencia): startup 3, ventana 8,
recovery 22, costo 15 de stamina.

## 23. Las tres armas

Tres estilos cuerpo a cuerpo diferenciados por alcance, velocidad y compromiso.

### 23.1 Principio de diferenciación

Cada arma debe responder distinto a la misma pregunta: "el jefe acaba de
terminar un ataque, ¿qué hago con esta ventana de castigo de 40 frames?"

- Mandoble: entra un golpe pesado, o dos ligeros justos.
- Lanza: entra un golpe desde fuera del alcance de contraataque.
- Dagas: entran cuatro golpes, pero hay que estar pegado.

Si las tres responden igual, la elección es cosmética y el sistema no vale su
costo.

### 23.2 Mandoble

Alto daño, lento, alto compromiso. El arma de quien lee bien los telegraphs.

| Parámetro | Valor |
|---|---|
| Alcance | 3.0 m |
| Ligero 1: startup / active / recovery | 18 / 5 / 24 |
| Ligero 2 | 16 / 5 / 30 |
| Ligero: daño / stamina | 30 / 28 |
| Pesado: startup / active / recovery | 34 / 7 / 42 |
| Pesado: daño / stamina | 68 / 45 |
| Poise damage ligero / pesado | 18 / 40 |
| Combo | Solo dos ligeros |
| Cancelación a rodar | Últimos 6 frames |

Nota: el mandoble tiene el poise damage más alto, así que es el arma que más
rápido llega al stagger. Esa es su recompensa por el riesgo.

### 23.3 Lanza

Alcance medio-largo, velocidad media, bajo riesgo. El arma segura.

| Parámetro | Valor |
|---|---|
| Alcance | 4.2 m |
| Ligero 1: startup / active / recovery | 13 / 4 / 18 |
| Ligero 2 | 12 / 4 / 20 |
| Ligero 3 | 14 / 4 / 26 |
| Ligero: daño / stamina | 17 / 18 |
| Pesado (estocada cargada): startup / active / recovery | 28 / 5 / 34 |
| Pesado: daño / stamina | 44 / 38 |
| Poise damage ligero / pesado | 8 / 22 |
| Combo | Tres ligeros |
| Cancelación a rodar | Últimos 8 frames |

Nota de balance: la lanza es el arma que más fácil se vuelve dominante, porque
castiga desde fuera del alcance del jefe. **Compensarla con daño bajo y poise
damage bajo, no con recovery largo**, para que siga siendo cómoda pero lenta de
matar. Es el arma a vigilar en el runner de balance.

### 23.4 Dagas

Corto alcance, muy rápido, exige estar encima del jefe.

| Parámetro | Valor |
|---|---|
| Alcance | 1.8 m |
| Ligero 1: startup / active / recovery | 8 / 3 / 12 |
| Ligero 2 | 7 / 3 / 12 |
| Ligero 3 | 7 / 3 / 14 |
| Ligero 4 | 9 / 4 / 22 |
| Ligero: daño / stamina | 11 / 12 |
| Pesado (giro): startup / active / recovery | 20 / 6 / 28 |
| Pesado: daño / stamina | 34 / 32 |
| Poise damage ligero / pesado | 4 / 14 |
| Combo | Cuatro ligeros |
| Cancelación a rodar | Últimos 10 frames |
| Bonus | Rodar cuesta 20 stamina en vez de 25 |

Nota: las dagas casi nunca llegan al stagger. Su ventaja es la movilidad y la
cantidad de ventanas pequeñas que puede aprovechar.

### 23.5 Implementación

Un `WeaponDefinition` (Resource) con un array de `AttackDefinition` para el
combo ligero, uno para el pesado, y los modificadores de stamina y movilidad.

```gdscript
class_name WeaponDefinition
extends Resource

@export var id: StringName
@export var display_name: String
@export var light_combo: Array[AttackDefinition]
@export var heavy_attack: AttackDefinition
@export var range_meters: float
@export var roll_stamina_cost: int = 25
@export var animation_set: StringName
```

**La máquina de estados del jugador no cambia entre armas.** Los estados de
ataque leen el `AttackDefinition` correspondiente del arma equipada. Si el
mandoble tiene dos ligeros y las dagas cuatro, es el largo del array el que lo
define, no un estado distinto.

Esta es la parte del proyecto que mejor demuestra diseño orientado a datos:
agregar un arma es crear un Resource y conseguir animaciones, sin tocar código.

## 24. Producción de animación: protocolo

Esta es la parte cara del proyecto y la que puede descarrilarlo.

**Regla: conseguir los tres sets de animación ANTES de finalizar los movesets.**
El diseño se adapta a lo que existe, no al revés.

Checklist de la semana 1:

- [ ] Elegir el rig del jugador (Mixamo o pack comercial con rig compatible).
- [ ] Verificar que existan, para ese mismo rig: idle, correr, rodar, y ataques
      de mandoble, lanza y dagas.
- [ ] Verificar que los tres sets tengan estilo visual coherente entre sí.
- [ ] Contar cuántos frames de ataque real trae cada animación.
- [ ] Elegir el modelo del jefe con sus ataques.

**Criterio de corte**: si al final de la semana 1 no conseguiste los tres sets
en estilo coherente, reducí a dos armas. Es mejor dos armas que funcionan que
tres donde una se ve prestada de otro juego.

Alternativa válida si falla: las tres armas comparten animaciones de cuerpo y se
diferencian solo por el modelo del arma y los timings. Se ve peor pero es
honesto y funcional; declararlo en el README.

## 25. La sala

Espacio circular de unos 20 metros de diámetro, plano, cerrado, sin cobertura.

Justificación del diseño plano: con tres armas cuerpo a cuerpo y sin arma a
distancia, no hay kiting posible, así que la sala no necesita obstáculos que lo
impidan. Un espacio limpio hace legible el posicionamiento y las telegrafías de
área.

Elementos:
- Borde visible e infranqueable, con retroalimentación clara al chocarlo.
- Iluminación que cambia con la fase del jefe. Es el recurso más barato para
  comunicar escalada.
- Marcas en el piso que ayuden a juzgar distancias. Sutiles, pero presentes.

## 26. Entrada, muerte y reinicio

### 26.1 Selección de arma

Antes de entrar a la sala, una pantalla simple de elección entre las tres armas,
con las estadísticas visibles y una descripción de una línea de cada estilo.
Se puede cambiar de arma entre intentos sin fricción.

**Requisito**: cambiar de arma no debe costar más de dos clics desde la pantalla
de muerte. Un jugador que quiere probar otra arma tras morir no debe atravesar
menús.

### 26.2 Entrada

Sin cinemática. El jugador aparece en la sala, el jefe está inactivo hasta que
el jugador cruza una línea. Eso da unos segundos para orientarse.

### 26.3 Muerte

- Cámara lenta breve, fundido rápido.
- Pantalla de muerte con dos opciones: reintentar y cambiar de arma.
- **Menos de dos segundos entre la muerte y el control recuperado.** Este número
  es un requisito de diseño, no una aspiración. Treinta intentos en veinte
  minutos es el objetivo.
- Estadística visible en la pantalla de muerte: intentos, mejor porcentaje de
  vida del jefe alcanzado, fase alcanzada. Muestra progreso aunque el intento
  fallara.

### 26.4 Victoria

Pantalla con estadísticas del intento: tiempo, intentos totales, parries
exitosos, daño evitado. Opción de reintentar con otra arma, que es lo que
convierte una victoria en tres partidas más.

## 27. Impacto en el plan

Cambios respecto de la Parte II:

- **Hito 1 suma el protocolo de animación de la sección 24.** Es lo primero de
  todo, antes de escribir el controlador.
- **Hito 2 se extiende a dos semanas**: tres movesets en vez de uno.
- **Hito 6 de tuning se vuelve más pesado**: cada ataque del jefe tiene que tener
  ventana de castigo válida para las tres armas. Ese es el checklist manual
  crítico.
- Duración total realista: **8-9 semanas**, no 5-7. La elección de arma es la
  decisión que más alcance agregó al proyecto.

Nuevo checklist manual de tuning, por cada ataque del jefe:

- [ ] Esquivable rodando en la dirección correcta.
- [ ] Clasificación parriable / no parriable visible en el telegraph.
- [ ] Ventana de castigo aprovechable con mandoble (al menos un ligero).
- [ ] Ventana de castigo aprovechable con lanza sin entrar en rango de réplica.
- [ ] Ventana de castigo aprovechable con dagas (al menos dos ligeros).

## 28. Riesgos actualizados

| Riesgo | Impacto | Prob. | Mitigación | Señal temprana |
|---|---|---|---|---|
| No existen los tres sets de animación coherentes | Crítico | Alta | Protocolo semana 1, criterio de corte a dos armas | Falta un set al cerrar la semana 1 |
| La lanza domina por alcance | Alto | Alta | Daño y poise bajos; vigilar en tuning | Los testers eligen lanza y ganan más rápido |
| Un ataque del jefe es imposible de castigar con dagas | Alto | Media | Checklist manual por arma | El tester con dagas nunca gana |
| Ataque no parriable confundido con parriable | Alto | Media | Destello distintivo en el startup | El tester intenta parry y muere repetido |
| El alcance de tres armas descarrila el plazo | Alto | Media | Criterio de corte, hito 2 extendido | El hito 2 pasa de dos semanas |
