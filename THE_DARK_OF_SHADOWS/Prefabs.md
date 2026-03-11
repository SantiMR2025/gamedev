Perfecto. Empecemos por las **plantillas de prefabs**.
Es lo más útil ahora, porque te va a evitar montar cada enemigo “a mano y con fe”.

Voy a dejarte una **biblioteca base de prefabs** para tu proyecto:

* Enemigo base
* Boss
* Escondite
* Puerta normal
* Puerta con keycard
* Puzzle
* Checkpoint
* Save point
* Spawn point del boss

Y además te diré **qué componentes lleva cada uno**, **qué hijos necesita** y **qué debes configurar**.

---

# 1) Estructura recomendada de prefabs

Dentro de `Assets/Prefabs`:

```text
Prefabs
├── Characters
│   ├── PF_Player
│   ├── PF_Enemy_Base
│   └── PF_Boss
│
├── Interaction
│   ├── PF_HideSpot_Locker
│   ├── PF_Door_Standard
│   ├── PF_Door_Keycard
│   ├── PF_Checkpoint
│   └── PF_SavePoint
│
├── Puzzles
│   ├── PF_Puzzle_Keycard_Lv1
│   ├── PF_Puzzle_Keycard_Lv2
│   ├── PF_Puzzle_Keycard_Lv3
│   ├── PF_Puzzle_HiddenCode
│   ├── PF_FuseBox
│   └── PF_GeneratorSwitch
│
└── Boss
    ├── PF_BossSpawnPoint
    └── PF_BossSequenceTrigger
```

---

# 2) PF_Enemy_Base

## Objeto raíz

```text
PF_Enemy_Base
```

## Componentes

* `CapsuleCollider` o collider del modelo
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
* opcional: `StealthKillTarget`

## Hijos recomendados

```text
PF_Enemy_Base
├── Model
├── EyePoint
└── DetectionRoot
```

## Configuración

* `EyePoint`: a la altura de los ojos/cabeza
* `NavMeshAgent`

  * Speed base: 3.5–4.5
  * Angular speed: 600 aprox
  * Stopping distance: 0.8–1.2
* Layer: `Enemy`
* Tag: opcional `Enemy`

## Inspector importante

En `EnemyAI`:

* `player` → Player
* `eyePoint` → hijo `EyePoint`
* `patrolPoints` → vacíos por defecto, se rellenan por escena
* `hearingRadius` → 14
* `viewDistance` → 12
* `viewAngle` → 90

---

# 3) PF_Boss

## Objeto raíz

```text
PF_Boss
```

## Componentes

* `CapsuleCollider`
* `NavMeshAgent`
* `Animator`
* `BossController`
* `BossAnimatorBridge`

## Hijos

```text
PF_Boss
├── Model
├── SpawnFXPoint
└── AttackPoint
```

## Configuración

* **Desactivado por defecto** en escena si lo controlas por spawner
* `NavMeshAgent`

  * speed: 6–7
  * angular speed: 800
* Tag opcional: `Boss`
* Layer opcional: `Enemy`

## Inspector importante

En `BossController`:

* `player` → Player
* `chaseEndPoint` → vacío por defecto
* `runWarningUI` → referencia UI
* `stingerAudio` → audio source del boss o externo

---

# 4) PF_HideSpot_Locker

Sirve para armario, taquilla o caja.

## Objeto raíz

```text
PF_HideSpot_Locker
```

## Componentes

* collider principal
* `HideSpot`

## Hijos

```text
PF_HideSpot_Locker
├── Mesh
├── HidePoint
├── ExitPoint
└── InspectPoint
```

## Configuración

* `HidePoint`: donde se coloca el jugador escondido
* `ExitPoint`: donde sale
* `InspectPoint`: donde llega el enemigo si sospecha o te vio entrar

## Importante

* `InspectPoint` debe quedar **fuera** del armario
* `ExitPoint` no debe chocar con pared
* collider interactuable accesible desde fuera

---

# 5) PF_Door_Standard

## Objeto raíz

```text
PF_Door_Standard
```

## Componentes

* `BoxCollider`
* `Animator`
* `DoorInteract`

## Hijos

```text
PF_Door_Standard
├── DoorMesh
└── HingePivot
```

## Animator

Crea dos triggers:

* `OpenForward`
* `OpenBackward`

## Configuración

En `DoorInteract`:

* `requiresKey` → según caso
* `requiredKeyId` → si aplica
* `animator` → Animator de la puerta

---

# 6) PF_Door_Keycard

## Objeto raíz

```text
PF_Door_Keycard
```

## Componentes

* `BoxCollider`
* `Animator`
* `KeycardDoor`

## Hijos

```text
PF_Door_Keycard
├── DoorMesh
└── ReaderPanel
```

## Animator

Trigger:

* `Open`

## Configuración

En `KeycardDoor`:

* `requiredLevel` → Blue / Red / Black
* `animator` → Animator

---

# 7) PF_Checkpoint

## Objeto raíz

```text
PF_Checkpoint
```

## Componentes

* `BoxCollider` con `Is Trigger`
* `CheckpointTrigger`

## Hijos opcionales

```text
PF_Checkpoint
├── VFX
└── MarkerMesh
```

## Uso

Cuando el jugador entra, guarda checkpoint.

## Consejo

Pon uno pequeño visual:

* luz
* poste
* lámpara
* altar raro si quieres ponerte dramático

---

# 8) PF_SavePoint

## Objeto raíz

```text
PF_SavePoint
```

## Componentes

* collider
* `SavePointInteract`

## Hijos opcionales

```text
PF_SavePoint
├── Mesh
└── ScreenGlow
```

## Uso

Punto manual de guardado.

---

# 9) PF_Puzzle_Keycard_Lv1

## Objeto raíz

```text
PF_Puzzle_Keycard_Lv1
```

## Componentes

* `KeycardLevel1Puzzle`

## Hijos

```text
PF_Puzzle_Keycard_Lv1
├── Button_1
├── Button_2
├── Button_3
├── Button_4
└── Rewards
```

## Rewards

En `Rewards`:

* `PuzzleSolvedRelay`
* `PuzzleReward_GiveKeycard`
* opcional `PuzzleReward_SetStoryFlag`

Cada botón puede tener un script de botón simple que llame:

```csharp
KeycardLevel1Puzzle.PressButton("1");
```

---

# 10) PF_Puzzle_HiddenCode

## Objeto raíz

```text
PF_Puzzle_HiddenCode
```

## Componentes

* `HiddenCodePuzzle`

## Hijos

```text
PF_Puzzle_HiddenCode
├── Keypad
├── Screen
└── Rewards
```

## Rewards

* `PuzzleSolvedRelay`
* `PuzzleReward_OpenAnimator`
* o `PuzzleReward_SetStoryFlag`

---

# 11) PF_FuseBox

## Objeto raíz

```text
PF_FuseBox
```

## Componentes

* `FuseBox`

## Hijos

```text
PF_FuseBox
├── Slot_1
├── Slot_2
├── Slot_3
└── IndicatorLight
```

## Configuración

* `requiredFuseCount` → 3
* `requiredFuseItemId` → `"Fuse"`

---

# 12) PF_GeneratorSwitch

## Objeto raíz

```text
PF_GeneratorSwitch
```

## Componentes

* `GeneratorSwitch`

## Hijos

```text
PF_GeneratorSwitch
├── Lever
├── Panel
└── PowerLight
```

## Configuración

* `fuseBox` → referencia al `FuseBox`
* `onGeneratorStarted` → luz, audio, flag, ambient

---

# 13) PF_BossSpawnPoint

## Objeto raíz

```text
PF_BossSpawnPoint
```

## Componentes

* `BossSpawnPoint`
* `BossSpawnPointGizmo`

## Uso

Lo colocas por el mapa.

## Consejo

Pon varios:

* esquinas
* puertas laterales
* pasillos largos
* zonas fuera de visión

---

# 14) PF_BossSequenceTrigger

Para secuencias tipo nivel 3 o encuentros forzados.

## Objeto raíz

```text
PF_BossSequenceTrigger
```

## Componentes posibles

* `BoxCollider` con `Is Trigger`
* `BossTimedEncounter`
  o
* `BossRunToPointSequence`

## Uso

Según el nivel:

* persecución temporal
* persecución hasta punto objetivo

---

# 15) PF_Player

Aunque ya lo tienes medio montado, conviene dejarlo como prefab limpio.

## Objeto raíz

```text
PF_Player
```

## Componentes

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

## Hijos

```text
PF_Player
├── CameraRoot
│   └── MainCamera
├── HoldPoint
├── GroundCheck
└── FlashlightLight
```

## Configuración importante

* Tag: `Player`
* `InteractionSystem.playerCamera` → `MainCamera`
* `PlayerMovement.groundCheck` → `GroundCheck`
* `PlayerItemHolder.holdPoint` → `HoldPoint`
* `FlashlightController.flashlightLight` → `FlashlightLight`

---

# 16) Plantilla por escena

Ahora lo útil: qué prefabs deberías arrastrar por escena.

## Escena bootstrap o managers

* `GameManager`
* `SaveSystem`
* `SceneLoader`
* `StoryFlags`
* `ObjectiveTracker`
* `TensionDirector`
* `BossSpawnRules`
* `EnemySearchCoordinator`
* `EnemySearchReservationSystem`
* `BossSpawnManager`
* `BossEncounterDirector`
* `BossTensionSpawner`
* `GameplayDebug`

## Nivel 1

* `PF_Player`
* `FlashlightPickup`
* `PhoneInteract`
* `PF_Checkpoint`
* tutoriales

## Nivel 2

* `PF_Player`
* varios `PF_Enemy_Base`
* `PF_HideSpot_Locker`
* `PF_Puzzle_Keycard_Lv1`
* `MirrorInteract`
* `PF_Checkpoint`

## Nivel 3

* `PF_Player`
* `PF_Enemy_Base`
* enemigo especial con `StealthKillTarget`
* `WalkieTalkieInteract`
* `PF_Door_Keycard`
* `PF_Boss`
* `PF_BossSequenceTrigger`

## Nivel 4

* `PF_Player`
* `PF_Enemy_Base`
* `PF_Puzzle_HiddenCode`
* `PF_FuseBox`
* `PF_GeneratorSwitch`
* `PF_Door_Keycard`
* `ElevatorAccess`
* `PF_Boss`
* `PF_BossSpawnPoint`

---

# 17) Reglas de oro para prefabs

Para que Unity no te haga la cobra:

* Los **prefabs base** deben quedar lo más genéricos posible.
* Lo específico de cada escena se rellena **en la escena**, no en el prefab.
* No metas referencias rotas a objetos de escena dentro del prefab base.
* Usa hijos estándar con nombres fijos:

  * `EyePoint`
  * `HidePoint`
  * `ExitPoint`
  * `InspectPoint`
  * `HoldPoint`
  * `GroundCheck`

Eso te ahorra media vida y tres disgustos existenciales.

---

# 18) Qué toca justo después

El siguiente paso ideal es la **guía final de montaje en Unity**, ya muy práctica:

* orden exacto de creación de escena
* tags y layers
* NavMesh
* colliders
* Canvas/UI
* conexión de managers
* checklist por nivel
* errores típicos que revisar primero

Ese documento ya sería tu “manual de ensamblaje”, versión menos sueca y más útil.
