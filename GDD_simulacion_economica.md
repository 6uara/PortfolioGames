# GDD base — Proyecto de simulación económica (nombre provisorio: "Aldea")

Versión 0.1 — documento base, sujeto a revisión durante prototipado.
Engine: Godot 4.7 / GDScript.
Duración objetivo: 6-8 semanas.
Rol que demuestra: game designer / systems design.

---

## 1. Pitch

Un prototipo de gestión de asentamiento donde el jugador administra un pueblo de
20-30 aldeanos a lo largo de un ciclo de estaciones. El objetivo es sobrevivir
un año completo sin que la población muera de hambre ni de frío. No hay combate,
no hay enemigos, no hay progresión de tecnología: todo el desafío está en
equilibrar cadenas de producción con mano de obra limitada.

## 2. Objetivo del proyecto (portfolio)

Este proyecto no busca ser divertido a nivel comercial. Busca demostrar:

- Diseño de sistemas interdependientes.
- Balanceo numérico documentado y justificado.
- Capacidad de exponer datos de diseño fuera del código (tuning por archivos).
- Legibilidad de sistemas complejos para el jugador.

Todo lo que no sirva a esos objetivos se recorta.

## 3. Loop de juego

Loop macro (estacional):
1. Empieza la estación. El jugador ve el stock, la población y las necesidades.
2. Asigna aldeanos a edificios de producción.
3. Construye o demuele edificios con los recursos disponibles.
4. Avanza el tiempo (con velocidad ajustable) y observa el resultado.
5. Al final de la estación, se resuelve el consumo y se reporta el balance.

Loop micro (minuto a minuto):
- Observar cuellos de botella, reasignar trabajadores, corregir.

## 4. Sistemas

### 4.1 Recursos

Cinco recursos, no más:

| Recurso | Origen | Uso |
|---|---|---|
| Leña | Bosque | Combustible de invierno, insumo de carbón |
| Carbón | Carbonera (consume leña) | Insumo de herrería, calefacción eficiente |
| Grano | Campo (estacional) | Insumo de harina |
| Harina | Molino (consume grano) | Insumo de pan |
| Pan | Panadería (consume harina + leña) | Alimento de la población |

Regla de diseño: cada recurso debe participar en al menos dos cadenas o ser
consumido directamente por la población. Si un recurso solo sirve para una cosa,
se elimina.

### 4.2 Cadenas de producción

- Leña → Carbón (ratio a definir en balance, arranque sugerido 3:1)
- Grano → Harina → Pan (arranque sugerido 2:1 y 1:1)

Cada edificio tiene:
- Cupo de trabajadores (1 a 3).
- Tasa de producción por trabajador por unidad de tiempo.
- Insumos consumidos por unidad producida.
- Costo de construcción.

### 4.3 Población

Cada aldeano tiene:
- Estado: asignado a un edificio, o inactivo.
- Necesidad de comida: consume N unidades de pan por estación.
- Necesidad de calor: activa solo en invierno, consume leña o carbón.
- Salud: baja si una necesidad queda insatisfecha; el aldeano muere si llega a cero.

Sin árbol genealógico, sin felicidad, sin nombres propios más allá de lo cosmético.
La población crece a tasa fija si hubo excedente de comida al cierre de la estación.

### 4.4 Estaciones

Cuatro estaciones de duración igual. Efectos:

- Primavera: siembra. El campo no produce, consume grano como semilla.
- Verano: producción normal.
- Otoño: cosecha. El campo produce su rendimiento acumulado.
- Invierno: el campo no produce. Se activa la necesidad de calor.

La estacionalidad es el corazón del desafío: obliga a planificar con anticipación
en vez de reaccionar.

### 4.5 Trabajo y movimiento

Los aldeanos caminan del hogar al lugar de trabajo y al depósito. La distancia
importa: un edificio lejos del depósito produce menos efectivo.

Limitación deliberada: grilla chica (32x32 casillas máximo) y pathfinding sobre
grilla con A*. Nada de navmesh, nada de evasión entre agentes.

## 5. Interfaz

- Vista top-down o isométrica fija, con zoom y paneo.
- Panel de recursos siempre visible con stock y delta por estación.
- Panel de población: total, asignados, inactivos, en riesgo.
- Al seleccionar un edificio: trabajadores asignados, producción efectiva,
  insumo faltante si aplica.
- Alerta clara cuando una cadena está bloqueada por falta de insumo.

La legibilidad es un criterio de evaluación del proyecto, no un extra. Si el
jugador no entiende por qué se murió su pueblo, el diseño falló.

## 6. Arte y presentación

- Assets de un pack gratuito (Kenney u otro) o primitivas con colores planos.
- Paleta limitada, un color por tipo de edificio.
- Sin animación de personajes: los aldeanos pueden ser cápsulas o sprites simples.

Esto se declara explícitamente en el README del repositorio. El proyecto no
pretende demostrar habilidad artística.

## 7. Datos y balanceo

Todos los números de diseño viven fuera del código, en archivos de datos
(Resources de Godot o JSON):

- Definición de recursos.
- Definición de edificios: costos, ratios, cupos, tasas.
- Definición de consumo poblacional por estación.
- Definición de duración y modificadores de estación.

Entregable de portfolio: un documento aparte de balance donde registres al menos
tres iteraciones de tuning, qué problema tenía cada versión y qué cambiaste. Ese
documento probablemente valga más que el build para un puesto de diseño.

## 8. Condiciones de victoria y derrota

- Victoria: sobrevivir cuatro estaciones completas con población igual o mayor a
  la inicial.
- Derrota: la población llega a cero.

Sin puntaje, sin estrellas, sin ranking.

## 9. Fuera de alcance (v1)

Explícitamente excluido:
- Combate, enemigos, eventos aleatorios de saqueo.
- Comercio con otras aldeas.
- Árbol tecnológico.
- Impuestos, dinero, mercado.
- Guardado y carga de partida.
- Múltiples mapas o generación procedural.

## 10. Hitos sugeridos

- Semana 1-2: grilla, colocación de edificios, aldeanos que caminan y trabajan.
- Semana 3: cadenas de producción completas y consumo de recursos.
- Semana 4: estaciones y necesidades poblacionales.
- Semana 5: interfaz y feedback de bloqueos.
- Semana 6: balanceo, documento de tuning, pulido, README y video.

## 11. Riesgos conocidos

- El pathfinding con muchos agentes puede consumir el presupuesto de tiempo.
  Mitigación: tope duro de 30 aldeanos, grilla chica, y recalcular rutas solo
  cuando cambia la asignación.
- El scope de gestión tiende a crecer. Cada sistema nuevo que se te ocurra
  durante el desarrollo va a una lista de "v2", no al proyecto.

---

# PARTE II — Diseño técnico y producción

Versión 0.2. Asume las convenciones de `00_CONVENCIONES_comunes.md`.

## 12. Arquitectura

```
src/
├── core/
│   ├── resource_definition.gd    # Resource: un recurso
│   ├── building_definition.gd    # Resource: un edificio
│   ├── season_definition.gd      # Resource: una estación
│   ├── population_config.gd      # Resource: consumo y crecimiento
│   └── registry.gd
├── sim/
│   ├── world_sim.gd              # orquestador del tick
│   ├── grid.gd                   # grilla y ocupación
│   ├── stockpile.gd              # inventario global
│   ├── production_system.gd
│   ├── villager.gd               # estado de un aldeano
│   ├── job_system.gd             # asignación y tareas
│   ├── pathfinding.gd            # A* sobre grilla
│   └── season_system.gd
├── presentation/
├── ui/
└── tools/
    └── headless_runner.gd
```

Misma regla que en el autobattler: `sim/` no conoce `ui/`. Permite correr un año
completo de simulación sin ventana, que es como se balancea esto.

## 13. Modelo de simulación

### 13.1 Escalas de tiempo

Tres escalas anidadas:

- **Tick**: unidad mínima, 10 por segundo real a velocidad 1x. Los aldeanos se
  mueven y trabajan por tick.
- **Día**: 100 ticks. Unidad de producción y de consumo de comida.
- **Estación**: 20 días. Unidad de decisión del jugador y de resolución de
  necesidades acumuladas.

Un año son 4 estaciones, 80 días, 8000 ticks. A velocidad 1x eso son 13 minutos;
con velocidades 2x y 4x, una partida completa entra en 5-10 minutos. **Esto
importa: una partida que no entra en diez minutos no la va a terminar ningún
evaluador.**

### 13.2 Bucle de tick

```
por tick:
  1. avanzar cada aldeano según su tarea actual
     - moviéndose: avanzar por su ruta
     - trabajando: acumular progreso de producción
     - inactivo: buscar tarea
  2. completar producciones terminadas -> stockpile
  3. si terminó un día: aplicar consumo diario de comida
  4. si terminó una estación: resolver necesidades, crecimiento, cambio de estación
  5. verificar condición de fin
```

### 13.3 Sistema de trabajo

Cada aldeano tiene una tarea con estados:
`Idle`, `MovingToWork`, `Working`, `MovingToStockpile`, `Depositing`.

Un edificio con insumos disponibles y cupo libre genera trabajo. El aldeano
inactivo más cercano lo toma.

**Simplificación deliberada**: el aldeano transporta lo que produce. No hay
transportistas dedicados, ni almacenes intermedios, ni cadenas logísticas. Eso es
todo un juego aparte.

### 13.4 Efecto de la distancia

La distancia importa porque el aldeano camina. Producción efectiva por aldeano:

```
prod_efectiva = tasa_base * (tiempo_trabajo / (tiempo_trabajo + tiempo_viaje))
```

No hace falta implementar esa fórmula: emerge sola del bucle. Pero hay que
tenerla escrita para poder explicar el balance y para validar el runner.

### 13.5 Pathfinding

A* sobre grilla, con estas restricciones duras:

- Grilla máxima 32x32.
- Máximo 30 aldeanos.
- **Recalcular ruta solo al cambiar de destino**, nunca por tick.
- Caché de rutas entre pares de edificios: las rutas se repiten muchísimo.
- Sin evasión entre agentes: los aldeanos se atraviesan. Es feo y es correcto
  para el alcance.

Con esas restricciones, el pathfinding deja de ser el riesgo del proyecto.

## 14. Esquemas de datos

### 14.1 ResourceDefinition

| Campo | Tipo |
|---|---|
| `id` | StringName |
| `display_name` | String |
| `icon` | Texture2D |
| `is_food` | bool |
| `is_fuel` | bool |
| `fuel_value` | float |

### 14.2 BuildingDefinition

| Campo | Tipo | Notas |
|---|---|---|
| `id` | StringName | |
| `display_name` | String | |
| `worker_slots` | int | 1-3 |
| `inputs` | Array de `{resource_id, amount}` | Por unidad producida |
| `outputs` | Array de `{resource_id, amount}` | |
| `work_ticks` | int | Ticks de trabajo por unidad, por trabajador |
| `build_cost` | Array de `{resource_id, amount}` | |
| `footprint` | Vector2i | Celdas ocupadas |
| `active_seasons` | Array de StringName | Vacío = todas |

### 14.3 SeasonDefinition

| Campo | Tipo |
|---|---|
| `id` | StringName |
| `days` | int |
| `heat_required` | bool |
| `production_modifiers` | Diccionario `building_id -> float` |

### 14.4 PopulationConfig

| Campo | Valor de arranque |
|---|---|
| `food_per_villager_per_day` | 0.5 |
| `fuel_per_villager_per_winter_day` | 0.8 |
| `health_max` | 3 |
| `health_lost_per_unmet_need_day` | 1 |
| `health_regen_per_satisfied_day` | 1 |
| `growth_surplus_threshold` | 10 unidades de comida al cierre de estación |
| `growth_amount` | 2 aldeanos |

## 15. Números de arranque

### 15.1 Cadenas

| Edificio | Cupo | Entrada | Salida | Ticks | Costo |
|---|---|---|---|---|---|
| Leñador | 2 | — | 1 leña | 60 | 8 leña |
| Carbonera | 1 | 3 leña | 1 carbón | 100 | 15 leña |
| Campo | 3 | — | 4 grano (solo otoño) | 80 | 10 leña |
| Molino | 1 | 2 grano | 1 harina | 70 | 20 leña |
| Panadería | 2 | 1 harina + 1 leña | 2 pan | 90 | 25 leña, 5 carbón |
| Casa | — | — | +3 de capacidad poblacional | — | 12 leña |
| Depósito | — | — | almacenamiento | — | inicial |

### 15.2 Estado inicial

10 aldeanos, 40 leña, 20 grano, 15 pan, un depósito, dos casas.

### 15.3 La ecuación que hay que verificar

Para 10 aldeanos, 80 días de comida: 400 unidades de pan al año.
Con panadería produciendo 2 pan cada 90 ticks por trabajador, y 100 ticks por
día, dos trabajadores producen aproximadamente 4.4 pan por día, o 352 por año.

**Eso da negativo a propósito.** El jugador tiene que expandir. Si los números de
arranque cierran, no hay juego. Verificar esta cuenta con el runner antes de
tocar nada más.

## 16. Runner headless

Menos central que en el autobattler, pero igual de útil.

Modos:
- **Simulación pasiva**: sin intervención del jugador, con una asignación fija.
  Responde: ¿cuánto sobrevive un pueblo que no hace nada?
- **Barrido de parámetros**: varía un ratio de producción y reporta el día de
  muerte del pueblo. Detecta cadenas rotas.
- **Estrategias predefinidas**: dos o tres asignaciones que representen jugadas
  razonables. Si ninguna sobrevive, el juego es imposible; si todas sobreviven
  cómodas, es trivial.

Salida a CSV con stock por recurso por día. Graficar eso en una planilla muestra
inmediatamente dónde se rompe la cadena.

## 17. Interfaz: requisitos de legibilidad

Este proyecto vive o muere por la legibilidad, así que se especifica en detalle.

**Panel de recursos** (siempre visible): stock actual, delta por día, y
proyección de días restantes al ritmo actual. La proyección es el dato que
convierte el juego en planificable.

**Panel de población**: total, asignados por edificio, inactivos, y cuántos están
perdiendo salud.

**Al seleccionar un edificio**: trabajadores, producción efectiva real (no la
teórica), y si está bloqueado, **por qué** está bloqueado.

**Estados de alerta explícitos**, con icono y color:
- Sin insumo: el edificio no tiene qué procesar.
- Sin cupo: hay trabajo pero nadie asignado.
- Sin espacio: el depósito está lleno.
- Necesidad insatisfecha: alguien va a empezar a morirse.

**Cierre de estación**: pantalla de resumen con producido, consumido, saldo por
recurso, y cambios de población. Es el momento de aprendizaje del jugador.

Regla: si el jugador pierde y no puede señalar qué decisión lo llevó ahí, la
interfaz falló, no el jugador.

## 18. Plan de pruebas

1. Conservación de recursos: nada se crea ni se destruye fuera de producción y
   consumo. Test con contador global.
2. Determinismo: misma semilla y mismas asignaciones, mismo estado al día 80.
3. Producción: un edificio con insumos y trabajadores produce la cantidad
   esperada en el tiempo esperado.
4. Pathfinding: existe ruta entre dos puntos conectados; no existe entre
   desconectados; no hay bucles infinitos.
5. Consumo: la población consume exactamente lo esperado por día.
6. Sin fugas: 30 aldeanos durante 8000 ticks sin crecimiento de memoria.

## 19. Backlog por hito

### Hito 1 — Grilla y aldeanos (semanas 1-2)
- [ ] Estructura de repo, convenciones
- [ ] Grilla, colocación y demolición de edificios
- [ ] A* con caché de rutas
- [ ] Aldeano con máquina de tareas
- [ ] Un edificio de producción funcionando de punta a punta
- [ ] Cámara con zoom y paneo
- **Terminado cuando**: un aldeano camina al leñador, produce, lleva al depósito
  y vuelve, en bucle estable.

### Hito 2 — Cadenas (semana 3)
- [ ] Los cinco recursos y los siete edificios
- [ ] Cadenas con insumos y bloqueo por falta de insumo
- [ ] Stockpile con capacidad
- [ ] Asignación de trabajadores por interfaz
- **Terminado cuando**: la cadena grano-harina-pan funciona completa.

### Hito 3 — Estaciones y población (semana 4)
- [ ] Ciclo de cuatro estaciones
- [ ] Siembra, cosecha, invierno sin producción de campo
- [ ] Consumo de comida y de combustible
- [ ] Salud, muerte y crecimiento poblacional
- [ ] Condiciones de victoria y derrota
- **Terminado cuando**: se puede jugar un año completo y perder.

### Hito 4 — Interfaz (semana 5)
- [ ] Paneles de recursos y población con proyecciones
- [ ] Estados de alerta
- [ ] Resumen de cierre de estación
- [ ] Velocidades 1x, 2x, 4x y pausa
- **Terminado cuando**: un tester entiende por qué se le murió el pueblo.

### Hito 5 — Balance (semana 6)
- [ ] Runner headless con los tres modos
- [ ] Barrido de parámetros
- [ ] Tres iteraciones documentadas en `docs/balance.md`
- [ ] Verificación de la ecuación de la sección 15.3
- **Terminado cuando**: existe evidencia de que el juego es ganable y no trivial.

### Hito 6 — Cierre (semanas 7-8)
- [ ] Arte de assets libres, paleta consistente
- [ ] Audio ambiente mínimo
- [ ] Tres playtests externos
- [ ] Build web, README, GIF, video
- [ ] Postmortem

## 20. Registro de riesgos

| Riesgo | Impacto | Prob. | Mitigación | Señal temprana |
|---|---|---|---|---|
| Pathfinding consume el presupuesto | Alto | Alta | Topes duros: 32x32, 30 aldeanos, caché | Cae el framerate con 20 aldeanos |
| Scope creep de gestión | Alto | Alta | Lista de fuera de alcance vinculante | Aparece "comercio" o "tecnología" |
| Balance imposible o trivial | Alto | Media | Runner desde hito 5, ecuación verificada | Sobrevivir sin hacer nada |
| Ilegibilidad de estado | Alto | Media | Alertas explícitas, proyecciones | El tester no sabe qué hacer |
| Partida demasiado larga | Medio | Media | Velocidades y 80 días fijos | Una partida tarda más de 15 min |

## 21. Preguntas abiertas

- ¿El jugador puede reasignar trabajadores durante la estación o solo entre
  estaciones? (recomendación: durante, con la pausa disponible)
- ¿La demolición devuelve recursos?
- ¿Las casas limitan la población o solo la salud?
- ¿Hay un segundo año como modo libre tras ganar?

---

# PARTE III — Escenarios de aprendizaje

Versión 0.3. Amplía la sección 17 con la curva de aprendizaje.

## 22. El problema que resuelven

Un juego de gestión de cadenas tiene un momento crítico: el jugador abre la
partida, ve una grilla vacía, un panel de recursos y diez aldeanos, y no sabe
qué hacer. Si en los primeros dos minutos no entiende el bucle, abandona.

Este proyecto es especialmente vulnerable porque no tiene combate ni presión
inmediata que empuje a actuar. La única motivación es entender el sistema.

Solución: **dos escenarios cortos de aprendizaje, opcionales pero recomendados,
accesibles desde el menú principal.**

## 23. Estructura común

Los escenarios usan la misma simulación que la campaña. No hay código de tutorial
aparte: son `ScenarioDefinition` con estado inicial, edificios habilitados,
duración y condición de victoria distintos.

```gdscript
class_name ScenarioDefinition
extends Resource

@export var id: StringName
@export var display_name: String
@export var description: String
@export var starting_villagers: int
@export var starting_stock: Array[ResourceAmount]
@export var starting_buildings: Array[PlacedBuilding]
@export var available_buildings: Array[StringName]   # subconjunto del total
@export var starting_season: StringName
@export var duration_days: int
@export var victory_condition: VictoryCondition
@export var hint_sequence: Array[HintDefinition]
```

**Esto no es un sistema nuevo, es una parametrización del que ya existe.** Ese es
el argumento que justifica su costo: dos escenarios cuestan un archivo de datos
cada uno más el sistema de pistas.

### 23.1 Sistema de pistas

Pistas contextuales, no un tutorial modal que bloquee.

```gdscript
class_name HintDefinition
extends Resource

@export var id: StringName
@export var text: String
@export var trigger: TriggerType      # ON_START, ON_RESOURCE_BELOW,
                                       # ON_BUILDING_PLACED, ON_IDLE_VILLAGERS,
                                       # ON_DAY_REACHED, ON_PRODUCTION_BLOCKED
@export var trigger_value: Variant
@export var highlight_ui_element: StringName   # opcional
```

Principios:
- La pista aparece cuando la situación la hace relevante, no en un orden fijo.
- Nunca bloquea el juego. El jugador puede ignorarla.
- Una sola pista visible a la vez.
- **Las pistas describen el sistema, no dan la solución.** "Los molinos necesitan
  grano para producir" en vez de "construí un campo ahora".

Este sistema también sirve en la campaña, con un pool reducido de pistas críticas
que se puede desactivar en opciones.

## 24. Escenario 1 — Básico: leña y comida

**Objetivo pedagógico**: enseñar el bucle fundamental. Un aldeano asignado a un
edificio produce, camina, deposita. La producción tiene un ritmo y ese ritmo se
puede medir.

| Parámetro | Valor |
|---|---|
| Aldeanos | 6 |
| Stock inicial | 20 leña, 10 pan |
| Edificios pre-colocados | Depósito, dos casas |
| Edificios disponibles | Leñador, campo, molino, panadería, casa |
| Estación | Primavera, sin invierno |
| Duración | 20 días (una estación) |
| Victoria | Terminar con 30 o más de pan en stock |

Sin carbonera, sin necesidad de calor, sin estaciones múltiples. El jugador solo
tiene que entender que grano se convierte en harina y harina en pan, y que eso
lleva tiempo.

Pistas previstas:
1. Al empezar: qué son los aldeanos inactivos y cómo se asignan.
2. Al colocar el primer edificio: cómo se lee la producción efectiva.
3. Al bloquearse una producción por falta de insumo: qué significa el estado de
   alerta.
4. Al llegar al día 10: cómo leer la proyección de días restantes.

**Duración estimada de juego**: 3 a 5 minutos con velocidad 4x.

## 25. Escenario 2 — Avanzado: el invierno

**Objetivo pedagógico**: enseñar la planificación estacional, que es el corazón
del juego. Producir hoy para consumir en tres semanas.

| Parámetro | Valor |
|---|---|
| Aldeanos | 10 |
| Stock inicial | 30 leña, 20 grano, 20 pan |
| Edificios pre-colocados | Depósito, tres casas, un leñador |
| Edificios disponibles | Todos |
| Estación | Otoño |
| Duración | 40 días (otoño + invierno) |
| Victoria | Llegar al final del invierno sin muertes |

El jugador entra en otoño, cosecha, y tiene que darse cuenta de que el invierno
que viene no produce grano y además consume combustible. Es exactamente el
desafío de la campaña, comprimido y sin margen de error acumulado.

Pistas previstas:
1. Al empezar: el invierno se acerca y el campo dejará de producir.
2. Al entrar al invierno: se activa la necesidad de calor.
3. Al bajar la leña de un umbral: proyección de cuántos días de calor quedan.
4. Al morir el primer aldeano, si ocurre: qué necesidad quedó insatisfecha.

**Duración estimada de juego**: 6 a 8 minutos con velocidad 4x.

## 26. Acceso y presentación

- Menú principal con tres entradas: Escenario básico, Escenario avanzado,
  Campaña.
- Los escenarios están marcados como recomendados, no obligatorios.
- **Al iniciar la campaña sin haber completado ningún escenario, una sola
  pregunta**: "¿Primera vez? Te recomendamos el escenario básico." Con opción de
  ir o de seguir de largo. Una vez y nunca más.
- Estado de completado guardado, visible en el menú.

Advertencia sobre lo opcional: un evaluador apurado va a saltar directo a la
campaña, se va a perder, y va a cerrar el juego. Esa única pregunta al inicio es
la mitigación mínima. Si en playtest se ve que igual la saltan y se pierden,
considerar hacer obligatorio el básico.

## 27. Valor de portfolio

Los escenarios no son solo onboarding. Demuestran algo específico y evaluable:

- **Diseño orientado a datos llevado al extremo**: dos experiencias distintas sin
  una línea de código específica de cada una.
- **Diseño de curva de aprendizaje**: qué se enseña, en qué orden, y con qué
  disparadores.
- Permiten al evaluador entender el juego en cinco minutos, que es
  probablemente todo el tiempo que le va a dar.

Documentar en `docs/encounters.md` (o `docs/scenarios.md`) qué enseña cada pista
y por qué está donde está.

## 28. Impacto en el plan

- **Nuevo hito, entre el actual 4 y el 5**: sistema de pistas y los dos
  escenarios. Estimar una semana.
- El sistema de pistas se reutiliza en la campaña, así que parte del costo se
  amortiza.
- Duración total realista: **8-9 semanas**, no 6-8.

Recorte si el plazo aprieta: hacer solo el escenario básico. Es el que resuelve
el problema crítico; el avanzado es refuerzo.

## 29. Riesgos actualizados

| Riesgo | Impacto | Prob. | Mitigación | Señal temprana |
|---|---|---|---|---|
| El evaluador salta los escenarios y se pierde | Alto | Alta | Pregunta única al iniciar campaña | El tester abandona en los primeros 3 minutos |
| Las pistas se vuelven un tutorial modal molesto | Medio | Media | Nunca bloquean, una a la vez, desactivables | Los testers las cierran sin leer |
| Los escenarios se convierten en código especial | Alto | Media | Todo por `ScenarioDefinition`, sin ramas en la sim | Aparece un `if scenario_id ==` en la simulación |
| El escenario básico es aburrido | Medio | Media | Tope de 5 minutos, velocidad 4x disponible | El tester lo abandona antes de terminarlo |
