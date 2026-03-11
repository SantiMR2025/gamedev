Perfecto. Vamos con la **guía final de montaje en Unity**.
La idea es dejarte un orden **realista, limpio y seguro** para montar el proyecto sin que todo dependa de acordarte de 47 referencias a mano y una plegaria.

Voy a estructurarlo así:

1. **Preparación del proyecto**
2. **Tags, Layers y NavMesh**
3. **Escena Bootstrap / Managers**
4. **Prefab del Player**
5. **UI global**
6. **Prefab de enemigo**
7. **Prefab del boss**
8. **Prefabs interactivos**
9. **Montaje por nivel**
10. **Checklist de prueba**
11. **Errores típicos**

---

# 1) Preparación del proyecto

## Estructura de carpetas

Dentro de `Assets`:

```text
Assets
├── Art
│   ├── Materials
│   ├── Models
│   ├── Textures
│   └── Animations
│
├── Audio
│   ├── Music
│   ├── SFX
│   ├── Voice
│   └── Ambient
│
├── Prefabs
│   ├── Characters
│   ├── Interaction
│   ├── Puzzles
│   ├── Boss
│   ├── UI
│   └── Managers
│
├── Scenes
│   ├── Bootstrap
│   ├── MainMenu
│   ├── Level_01
│   ├── Level_02
│   ├── Level_03
│   ├── Level_04
│   ├── Final_Level
│   ├── Ending_Good
│   └── Ending_Bad
│
├── Scripts
│   ├── Core
│   ├── Player
│   ├── Enemy
│   ├── Boss
│   ├── Interaction
│   ├── Puzzle
│   ├── Audio
│   ├── Story
│   ├── UI
│   └── Debug
│
├── ScriptableObjects
│   └── TensionProfiles
│
└── UI
    ├── Fonts
    ├── Sprites
    └── Icons
```

---

# 2) Tags, Layers y NavMesh

## Tags necesarias

Crea estas tags:

```text
Player
Enemy
Boss
Interactable
Ground
```

## Layers recomendadas

Crea estas layers:

```text
Player
Enemy
Boss
Interactable
Ground
HideSpot
TriggerOnly
```

## Uso recomendado

* `Player` → jugador
* `Enemy` → enemigos normales
* `Boss` → boss
* `Interactable` → puertas, llaves, puzzles, etc.
* `Ground` → suelos navegables y detectables por pasos
* `HideSpot` → escondites
* `TriggerOnly` → volúmenes de tutoriales, boss triggers, zonas especiales

## NavMesh

Antes de probar IA:

* marca suelos caminables como **Navigation Static** o usa `NavMeshSurface`
* hornea el NavMesh
* revisa que:

  * enemigos puedan recorrer pasillos
  * no atraviesen obstáculos grandes
  * el boss tenga espacio suficiente
  * los `SearchZone` estén sobre área navegable

Si el NavMesh falla, la IA se vuelve filósofa: piensa mucho, hace poco.

---

# 3) Escena Bootstrap / Managers

Te recomiendo una escena `Bootstrap` que cargue o contenga los sistemas persistentes.

## GameObject raíz

```text
Bootstrap_Managers
```

## Hijos recomendados

```text
Bootstrap_Managers
├── CoreManagers
├── AudioManagers
├── AIDirectors
├── UIRoot
└── DebugRoot
```

## Componentes / objetos dentro

### CoreManagers

* `GameManager`
* `SaveSystem`
* `SceneLoader`
* `StoryFlags`
* `ObjectiveTracker`
* `CheckpointSystem`

### AudioManagers

* `MusicManager`
* `AmbientAudioManager`

### AIDirectors

* `TensionDirector`
* `BossSpawnRules`
* `EnemySearchCoordinator`
* `EnemySearchReservationSystem`
* `BossSpawnManager`
* `BossEncounterDirector`
* `BossTensionSpawner`

### DebugRoot

* `DebugGameplaySettings`
* `NoiseDebugMonitor`
* `GameplayDebugHotkeys`

Guarda este conjunto como prefab si quieres:

```text
PF_BootstrapManagers
```

## Recomendación

Todos estos managers deberían usar:

```csharp
DontDestroyOnLoad(gameObject);
```

o vivir en una escena bootstrap cargada antes de todo.

---

# 4) Prefab del Player

## Estructura recomendada

```text
PF_Player
├── Visual
├── CameraRoot
│   └── MainCamera
├── GroundCheck
├── HoldPoint
└── FlashlightLight
```

## Componentes en raíz

* `CharacterController`
* `PlayerMovement`
* `PlayerStamina`
* `InteractionSystem`
* `PlayerHide`
* `FlashlightController`
* `PlayerItemHolder`
* `FootstepNoiseEmitter`
* `FootstepSurfaceAudio`
* `PlayerAnimationBridge`

## Configuración clave

### CharacterController

Ajusta aprox:

* Height: `1.8`
* Radius: `0.35 - 0.45`
* Center Y: `0.9`

### PlayerMovement

Asigna:

* `controller` → CharacterController del mismo objeto
* `groundCheck` → hijo `GroundCheck`
* `groundMask` → `Ground`

### InteractionSystem

Asigna:

* `playerCamera` → `MainCamera`
* `interactMask` → layers interactuables

### PlayerItemHolder

Asigna:

* `holdPoint` → hijo `HoldPoint`

### FlashlightController

Asigna:

* `flashlightLight` → hijo `FlashlightLight`

## Muy importante

El prefab del player debe llevar:

* Tag = `Player`
* capa adecuada
* cámara funcional
* collider bien alineado

---

# 5) UI global

Te recomiendo un solo `Canvas` principal por escena jugable, o uno persistente si prefieres.

## Estructura sugerida

```text
Canvas_Gameplay
├── InteractionPrompt
├── ObjectivePanel
├── SubtitlePanel
├── TutorialPanel
├── PausePanel
├── GameOverPanel
├── LoadingPanel
├── DebugPanel
│   ├── TensionDebug
│   └── EnemyStateDebug
```

## Scripts que van aquí

* `ObjectiveUI`
* `SubtitleUI`
* `TutorialUI`
* `PauseMenuUI`
* `GameOverUI`
* `TensionDebugUI`
* `EnemyStateDebugUI`

## Referencias

Asegúrate de conectar:

* `InteractionSystem.promptText`
* `ObjectiveUI.objectiveText`
* `SubtitleUI.subtitleText`
* `TutorialUI.messageText`
* `PauseMenuUI.pausePanel`
* `GameOverUI.gameOverPanel`

## Cursor

Cuando juegues:

* juego normal → cursor bloqueado
* pausa / game over → cursor visible

---

# 6) Prefab de enemigo

## Estructura recomendada

```text
PF_Enemy_Base
├── Model
├── EyePoint
└── PatrolRoot
```

## Componentes

* `NavMeshAgent`
* `Animator`
* `EnemyAI`
* `EnemyInvestigationMemory`
* `EnemyCommunicationSystem`
* `EnemyHideSpotSearch`
* `EnemyFlankHelper`
* `TimedNoiseMemory`
* `EnemyAnimatorBridge`
* `AIDebugGizmos`
* opcional `StealthKillTarget`

## Configuración clave

### EnemyAI

Asigna:

* `player` → Player en escena o búsqueda automática
* `eyePoint` → hijo `EyePoint`
* `patrolPoints` → se rellenan por escena
* `obstructionMask` → capas de paredes/escenario

### EnemyCommunicationSystem

* `enemyMask` → layer `Enemy`

### EnemyHideSpotSearch

* `hideSpotMask` → layer `HideSpot`

### NavMeshAgent

Ajuste base razonable:

* Speed: `3.5 - 4.5`
* Angular Speed: `600 - 800`
* Acceleration: `8 - 12`
* Stopping Distance: `0.8 - 1.2`

---

# 7) Prefab del boss

## Estructura

```text
PF_Boss
├── Model
├── SpawnFXPoint
└── AttackPoint
```

## Componentes

* `NavMeshAgent`
* `Animator`
* `BossController`
* `BossAnimatorBridge`

## Configuración

* al inicio puede estar desactivado en escena si se spawnea por sistema
* `BossController.player` → Player
* `runWarningUI` → referencia al panel RUN o warning
* `stingerAudio` → audio source

## Consejo

Pon al boss en una layer separada si quieres distinguirlo de enemigos normales.

---

# 8) Prefabs interactivos

## Escondite

`PF_HideSpot_Locker`

Debe tener:

* collider accesible
* `HideSpot`
* hijos:

  * `HidePoint`
  * `ExitPoint`
  * `InspectPoint`

## Puerta normal

`PF_Door_Standard`

Debe tener:

* `Animator`
* `DoorInteract`
* triggers de animación:

  * `OpenForward`
  * `OpenBackward`

## Puerta con keycard

`PF_Door_Keycard`

Debe tener:

* `Animator`
* `KeycardDoor`
* trigger:

  * `Open`

## Checkpoint

`PF_Checkpoint`

* `BoxCollider` trigger
* `CheckpointTrigger`

## Save point

`PF_SavePoint`

* collider
* `SavePointInteract`

## Boss spawn point

`PF_BossSpawnPoint`

* `BossSpawnPoint`
* `BossSpawnPointGizmo`

---

# 9) Montaje por nivel

---

## Nivel 1

### Elementos mínimos

* `PF_Player`
* `FlashlightPickup`
* `PhoneInteract`
* 1 `PF_Checkpoint`
* `TutorialTrigger` para:

  * movimiento
  * interacción
  * linterna
* `AutoObjectiveSetter`

### Flujo

1. objetivo: encontrar linterna
2. recoger linterna
3. objetivo: responder teléfono
4. usar teléfono
5. cinemática / cambio de escena

### Revisión rápida

* teléfono no usable sin linterna
* prompt visible
* escena carga bien al terminar

---

## Nivel 2

### Elementos mínimos

* `PF_Player`
* 2–4 `PF_Enemy_Base`
* varios `PF_HideSpot_Locker`
* `PF_Puzzle_Keycard_Lv1`
* `MirrorInteract`
* 1–2 checkpoints
* `SearchZone`
* tutorial de esconderse
* tutorial de distracción

### Revisión rápida

* enemigos patrullan
* oyen objetos lanzados
* revisan escondites cercanos
* si te ven esconderte, van al armario correcto
* puzzle da keycard
* espejo cierra nivel

---

## Nivel 3

### Elementos mínimos

* `PF_Player`
* enemigos normales
* 1 enemigo especial con:

  * `StealthKillTarget`
  * `StealthKillTargetKeyDrop`
* `StealthKillUnlock`
* `WalkieTalkieInteract`
* `PF_Door_Keycard`
* `PF_Boss`
* `PF_BossSequenceTrigger`

### Revisión rápida

* desbloqueo de remate funciona
* solo puedes rematar por detrás
* enemigo especial deja llave
* boss persigue hasta punto objetivo
* transición final correcta

---

## Nivel 4

### Elementos mínimos

* `PF_Player`
* enemigos normales
* `PF_Puzzle_HiddenCode`
* `PF_FuseBox`
* `PF_GeneratorSwitch`
* `FusePickup`
* `ElevatorAccess`
* `PF_Boss`
* varios `PF_BossSpawnPoint`
* `BossSpawnRuleZone` para:

  * safe room
  * checkpoint
  * puzzle crítico
* `SearchZone`
* `TensionProfileApplier`

### Revisión rápida

* código oculto abre lo correcto
* fusibles cuentan bien
* generador activa ambient / flag
* ascensor pide energía + tarjeta
* boss no spawnea en safe room ni puzzle crítico
* tensión sube y baja con sentido

---

## Nivel final

### Elementos mínimos

* `PF_Player`
* `PF_Boss`
* `FinalChoiceManager`
* `BossFightManager`
* `BossWeakPointInteract`
* `ThirdPersonBossFightCamera`
* `BossFightStarter`
* `EndingLoader`

### Revisión rápida

* elección escapar/luchar funciona
* boss fight entra en tercera persona
* ventanas de vulnerabilidad correctas
* número de fases correcto
* final bueno / malo carga bien

---

# 10) Checklist de prueba rápida por escena

Cada vez que montes una escena, revisa esto:

## Player

* se mueve
* salta
* corre
* stamina baja y recupera
* cámara funciona
* prompt aparece

## Interacción

* puertas responden
* llaves se recogen
* keycards se guardan
* escondites funcionan
* puedes salir del escondite

## IA

* enemigos patrullan
* detectan ruido
* cambian de estado
* persiguen
* vuelven a buscar
* comparten alertas

## Boss

* puede spawnear
* no spawnea en zonas bloqueadas
* modo de persecución correcto
* música cambia al aparecer

## UI

* objetivo visible
* subtítulos visibles
* tutoriales visibles
* pausa funciona
* game over funciona

## Guardado

* guarda
* carga
* checkpoint respawnea correctamente

---

# 11) Errores típicos que revisar primero

## El enemigo no se mueve

Revisa:

* `NavMesh` horneado
* `NavMeshAgent` activo
* no está fuera del NavMesh
* `patrolPoints` asignados

## El enemigo no ve al jugador

Revisa:

* `eyePoint`
* `viewDistance`
* `viewAngle`
* paredes en medio
* tag `Player`

## El enemigo no oye ruido

Revisa:

* `NoiseSystem.Emit(...)`
* `hearingRadius`
* `hideSpotMask / enemyMask`
* que el objeto lanzado tenga collision real

## El jugador no puede interactuar

Revisa:

* `InteractionSystem.playerCamera`
* distancia de interacción
* collider del objeto
* interfaz `IInteractable`
* layer incluida en `interactMask`

## El boss no spawnea

Revisa:

* `TensionDirector.CanSpawnBoss`
* `BossSpawnRules`
* `BossSpawnManager` con puntos válidos
* distancias mín/max
* boss ya activo por error

## El checkpoint carga mal

Revisa:

* player con tag `Player`
* `CharacterController` se desactiva al reposicionar
* referencias de `InventorySystem`, `StoryFlags`, `PlayerStamina`

## La puerta no abre

Revisa:

* trigger del Animator
* Animator asignado
* llave o tarjeta correcta
* `isOpen` no bloqueado por error

---

# 12) Orden recomendado real de producción

Si quieres montarlo sin liarte, este sería el orden más sensato:

1. escena bootstrap
2. prefab player
3. UI global
4. escena nivel 1 completa
5. prefab enemigo base
6. escondites + ruido + distracción
7. nivel 2 completo
8. stealth kill + boss secuencia nivel 3
9. nivel 4 con tensión, boss dinámico y energía
10. final boss fight
11. debug y balance
12. pulido audiovisual

Ese orden evita que estés intentando balancear al boss cuando todavía una puerta no abre y una linterna atraviesa una pared con convicción.

---

# 13) Siguiente paso útil

Lo más útil ahora es hacerte un **checklist técnico final por escena**, en formato muy práctico, tipo:

* qué objetos crear
* qué referencias asignar
* qué probar antes de dar la escena por válida

Eso ya sería literalmente tu hoja de montaje.
