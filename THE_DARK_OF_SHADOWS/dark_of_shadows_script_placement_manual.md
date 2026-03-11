# Dark of Shadows – Script Placement Manual

## 1. Estructura exacta de carpetas

Crea esta estructura dentro de **Assets** antes de pegar scripts:

- `Assets/Scripts/Core`
- `Assets/Scripts/Player`
- `Assets/Scripts/Interaction`
- `Assets/Scripts/Items`
- `Assets/Scripts/Enemy`
- `Assets/Scripts/Boss`
- `Assets/Scripts/Puzzle`
- `Assets/Scripts/Story`
- `Assets/Scripts/Audio`
- `Assets/Scripts/UI`
- `Assets/Scripts/Debug`
- `Assets/ScriptableObjects/TensionProfiles`

## 2. Managers persistentes (Bootstrap)

| Script | Ruta exacta | Dónde se coloca | Referencias clave |
|---|---|---|---|
| GameManager | `Assets/Scripts/Core/GameManager.cs` | `Bootstrap_Managers/CoreManagers` | Conecta `onGameOver`, `onPause`, `onResume` a UI si quieres usar UnityEvents |
| SaveSystem | `Assets/Scripts/Core/SaveSystem.cs` | `Bootstrap_Managers/CoreManagers` | Player, PlayerStamina, InventorySystem se pueden resolver automáticamente |
| SceneLoader | `Assets/Scripts/Core/SceneLoader.cs` | `Bootstrap_Managers/CoreManagers` | `CanvasGroup` de LoadingPanel |
| StoryFlags | `Assets/Scripts/Core/StoryFlags.cs` | `Bootstrap_Managers/CoreManagers` | Singleton, sin refs |
| ObjectiveTracker | `Assets/Scripts/Core/ObjectiveTracker.cs` | `Bootstrap_Managers/CoreManagers` | Singleton, sin refs |
| CheckpointSystem | `Assets/Scripts/Core/CheckpointSystem.cs` | `Bootstrap_Managers/CoreManagers` | Player, PlayerStamina, InventorySystem |
| MusicManager | `Assets/Scripts/Audio/MusicManager.cs` | `Bootstrap_Managers/AudioManagers` | 3 AudioSource: Calm, Suspicious, Chase |
| AmbientAudioManager | `Assets/Scripts/Audio/AmbientAudioManager.cs` | `Bootstrap_Managers/AudioManagers` | 2 AudioSource: calmAmbient, powerOnAmbient |
| TensionDirector | `Assets/Scripts/Enemy/TensionDirector.cs` | `Bootstrap_Managers/AIDirectors` | LevelTensionProfile activo por escena |
| BossSpawnRules | `Assets/Scripts/Boss/BossSpawnRules.cs` | `Bootstrap_Managers/AIDirectors` | Sin refs |
| EnemySearchCoordinator | `Assets/Scripts/Enemy/EnemySearchCoordinator.cs` | `Bootstrap_Managers/AIDirectors` | Sin refs |
| EnemySearchReservationSystem | `Assets/Scripts/Enemy/EnemySearchReservationSystem.cs` | `Bootstrap_Managers/AIDirectors` | Sin refs |
| BossSpawnManager | `Assets/Scripts/Boss/BossSpawnManager.cs` | `Bootstrap_Managers/AIDirectors` | Array de BossSpawnPoint y Player |
| BossEncounterDirector | `Assets/Scripts/Boss/BossEncounterDirector.cs` | `Bootstrap_Managers/AIDirectors` | BossController, optionalRunTargetPoint, guidedJumpscarePoints |
| BossTensionSpawner | `Assets/Scripts/Boss/BossTensionSpawner.cs` | `Bootstrap_Managers/AIDirectors` | Usa BossEncounterDirector |

## 3. Player: prefab `PF_Player`

Raíz del prefab: **PF_Player**

Hijos recomendados:
- `CameraRoot/MainCamera`
- `GroundCheck`
- `HoldPoint`
- `FlashlightLight`

| Script | Ruta exacta | Dónde se coloca | Referencias clave |
|---|---|---|---|
| PlayerMovement | `Assets/Scripts/Player/PlayerMovement.cs` | Raíz | CharacterController, PlayerStamina, GroundCheck |
| PlayerStamina | `Assets/Scripts/Player/PlayerStamina.cs` | Raíz | Opcional UI de stamina |
| InteractionSystem | `Assets/Scripts/Interaction/InteractionSystem.cs` | Raíz | Camera, TMP_Text prompt, interactMask |
| PlayerHide | `Assets/Scripts/Player/PlayerHide.cs` | Raíz | Camera playerCamera, cameraHiddenAnchor opcional |
| FlashlightController | `Assets/Scripts/Player/FlashlightController.cs` | Raíz | Light flashlightLight |
| PlayerItemHolder | `Assets/Scripts/Player/PlayerItemHolder.cs` | Raíz | Transform HoldPoint |
| FootstepNoiseEmitter | `Assets/Scripts/Audio/FootstepNoiseEmitter.cs` | Raíz | Depende de PlayerMovement |
| FootstepSurfaceAudio | `Assets/Scripts/Audio/FootstepSurfaceAudio.cs` | Raíz | AudioSource, groundMask, tags de superficie |
| PlayerAnimationBridge | `Assets/Scripts/Player/PlayerAnimationBridge.cs` | Raíz | Animator, PlayerMovement |
| ThirdPersonBossFightCamera | `Assets/Scripts/Boss/ThirdPersonBossFightCamera.cs` | Cámara aparte en escena final | Transform target |

## 4. UI global

Canvas recomendado:

- `InteractionPrompt`
- `ObjectivePanel`
- `SubtitlePanel`
- `TutorialPanel`
- `PausePanel`
- `GameOverPanel`
- `LoadingPanel`
- `DebugPanel`

| Script | Ruta exacta | Dónde se coloca | Referencias clave |
|---|---|---|---|
| ObjectiveUI | `Assets/Scripts/UI/ObjectiveUI.cs` | `Canvas_Gameplay/ObjectivePanel` | TMP_Text objectiveText |
| SubtitleUI | `Assets/Scripts/UI/SubtitleUI.cs` | `Canvas_Gameplay/SubtitlePanel` | CanvasGroup + TMP_Text |
| TutorialUI | `Assets/Scripts/Story/TutorialUI.cs` | `Canvas_Gameplay/TutorialPanel` | CanvasGroup + TMP_Text |
| PauseMenuUI | `Assets/Scripts/UI/PauseMenuUI.cs` | `Canvas_Gameplay/PausePanel` | pausePanel |
| GameOverUI | `Assets/Scripts/UI/GameOverUI.cs` | `Canvas_Gameplay/GameOverPanel` | gameOverPanel |
| TensionDebugUI | `Assets/Scripts/Debug/TensionDebugUI.cs` | `Canvas_Debug/TensionPanel` | TMP_Text + Slider |
| EnemyStateDebugUI | `Assets/Scripts/Debug/EnemyStateDebugUI.cs` | `Canvas_Debug/EnemyInfoPanel` | Camera, enemyMask, TMP_Text |

## 5. Interacción, pickups y puertas

| Script | Ruta exacta | Dónde se coloca | Referencias clave |
|---|---|---|---|
| IInteractable | `Assets/Scripts/Interaction/IInteractable.cs` | No se añade a objeto | Interfaz base |
| KeyPickup | `Assets/Scripts/Items/KeyPickup.cs` | Objeto llave | Collider + InventorySystem |
| KeycardPickup | `Assets/Scripts/Items/KeycardPickup.cs` | Objeto keycard | Collider + AccessCardLevel |
| FlashlightPickup | `Assets/Scripts/Items/FlashlightPickup.cs` | Objeto linterna | Player con FlashlightController |
| FusePickup | `Assets/Scripts/Items/FusePickup.cs` | Objeto fusible | InventorySystem |
| ThrowableItem | `Assets/Scripts/Items/ThrowableItem.cs` | Botellas/cajas/herramientas | Rigidbody + Collider |
| DoorInteract | `Assets/Scripts/Interaction/DoorInteract.cs` | `PF_Door_Standard` | Animator con OpenForward/OpenBackward |
| KeycardDoor | `Assets/Scripts/Interaction/KeycardDoor.cs` | `PF_Door_Keycard` | Animator con trigger Open |
| HideSpot | `Assets/Scripts/Interaction/HideSpot.cs` | `PF_HideSpot_Locker` | HidePoint, ExitPoint, InspectPoint |
| PhoneInteract | `Assets/Scripts/Interaction/PhoneInteract.cs` | Teléfono Nivel 1 | UnityEvent onPhoneUsed |
| MirrorInteract | `Assets/Scripts/Interaction/MirrorInteract.cs` | Espejo Nivel 2 | UnityEvent onMirrorUsed |
| WalkieTalkieInteract | `Assets/Scripts/Interaction/WalkieTalkieInteract.cs` | Walkie Nivel 3 | UnityEvent onWalkieUsed |
| ElevatorAccess | `Assets/Scripts/Interaction/ElevatorAccess.cs` | Panel/puerta del ascensor | Requiere power y/o keycard |
| SavePointInteract | `Assets/Scripts/Interaction/SavePointInteract.cs` | `PF_SavePoint` | SaveSystem |

## 6. Enemigos normales: prefab `PF_Enemy_Base`

Hijos recomendados:
- `Model`
- `EyePoint`
- `PatrolRoot`

| Script | Ruta exacta | Dónde se coloca | Referencias clave |
|---|---|---|---|
| EnemyAI | `Assets/Scripts/Enemy/EnemyAI.cs` | Raíz de PF_Enemy_Base | NavMeshAgent, Player, EyePoint, patrolPoints |
| EnemyInvestigationMemory | `Assets/Scripts/Enemy/EnemyInvestigationMemory.cs` | Raíz | Sin refs |
| EnemyCommunicationSystem | `Assets/Scripts/Enemy/EnemyCommunicationSystem.cs` | Raíz | enemyMask = layer Enemy |
| EnemyHideSpotSearch | `Assets/Scripts/Enemy/EnemyHideSpotSearch.cs` | Raíz | hideSpotMask = layer HideSpot |
| EnemyFlankHelper | `Assets/Scripts/Enemy/EnemyFlankHelper.cs` | Raíz | Sin refs |
| TimedNoiseMemory | `Assets/Scripts/Enemy/TimedNoiseMemory.cs` | Raíz | noiseLifetime |
| EnemyAnimatorBridge | `Assets/Scripts/Enemy/EnemyAnimatorBridge.cs` | Raíz | Animator, NavMeshAgent, EnemyAI |
| AIDebugGizmos | `Assets/Scripts/Debug/AIDebugGizmos.cs` | Raíz | EnemyAI, NavMeshAgent, EyePoint |
| StealthKillTarget | `Assets/Scripts/Enemy/StealthKillTarget.cs` | Solo en enemigos rematables | Flag de unlock |
| StealthKillTargetKeyDrop | `Assets/Scripts/Enemy/StealthKillTargetKeyDrop.cs` | Solo en enemigo especial | Conectar a onStealthKill |

## 7. Boss y encuentros

| Script | Ruta exacta | Dónde se coloca | Referencias clave |
|---|---|---|---|
| BossController | `Assets/Scripts/Boss/BossController.cs` | Raíz de PF_Boss | NavMeshAgent, Player, warning UI, stinger |
| BossAnimatorBridge | `Assets/Scripts/Boss/BossAnimatorBridge.cs` | Raíz de PF_Boss | Animator, NavMeshAgent |
| BossTimedEncounter | `Assets/Scripts/Boss/BossTimedEncounter.cs` | Trigger de escena | BossController |
| BossRunToPointSequence | `Assets/Scripts/Boss/BossRunToPointSequence.cs` | Trigger de escena | BossController + endPoint |
| BossSpawnPoint | `Assets/Scripts/Boss/BossSpawnPoint.cs` | `PF_BossSpawnPoint` | Sin refs |
| BossSpawnPointGizmo | `Assets/Scripts/Debug/BossSpawnPointGizmo.cs` | `PF_BossSpawnPoint` | Solo debug |
| BossSpawnManager | `Assets/Scripts/Boss/BossSpawnManager.cs` | Manager global | Array de BossSpawnPoint |
| BossSpawnRules | `Assets/Scripts/Boss/BossSpawnRules.cs` | Manager global | Sin refs |
| BossSpawnRuleZone | `Assets/Scripts/Boss/BossSpawnRuleZone.cs` | Safe rooms/checkpoints/puzzles | RuleType |
| BossEncounterDirector | `Assets/Scripts/Boss/BossEncounterDirector.cs` | Manager global | BossController + puntos |
| BossTensionSpawner | `Assets/Scripts/Boss/BossTensionSpawner.cs` | Manager global | Usa BossEncounterDirector |
| BossFightManager | `Assets/Scripts/Boss/BossFightManager.cs` | Arena final | UnityEvents de combate |
| BossWeakPointInteract | `Assets/Scripts/Boss/BossWeakPointInteract.cs` | Punto débil del boss | BossFightManager |
| BossFightStarter | `Assets/Scripts/Boss/BossFightStarter.cs` | Trigger/objeto inicio combate | BossFightManager, cámara TP, BossController |
| BossVulnerabilityTrigger | `Assets/Scripts/Boss/BossVulnerabilityTrigger.cs` | Trigger o Animation Event | BossFightManager |
| BossEncounterBlockerManual | `Assets/Scripts/Boss/BossEncounterBlockerManual.cs` | Timeline / puzzle crítico | TensionDirector |
| BossSpawnRulesRelay | `Assets/Scripts/Boss/BossSpawnRulesRelay.cs` | Timeline o eventos | BossSpawnRules |
| EndingLoader | `Assets/Scripts/Boss/EndingLoader.cs` | Finales | Nombres de escena |

## 8. Puzzles, historia y flujo de nivel

| Script | Ruta exacta | Dónde se coloca | Referencias clave |
|---|---|---|---|
| TutorialTrigger | `Assets/Scripts/Story/TutorialTrigger.cs` | Triggers de tutorial | Mensaje y duración |
| SubtitleTrigger | `Assets/Scripts/Story/SubtitleTrigger.cs` | Triggers o eventos | Texto y duración |
| CinematicSignalRelay | `Assets/Scripts/Story/CinematicSignalRelay.cs` | Timeline / señales | Flags, objetivos, subtítulos, escenas |
| AutoObjectiveSetter | `Assets/Scripts/Story/AutoObjectiveSetter.cs` | Trigger o evento | Texto de objetivo |
| StealthKillUnlock | `Assets/Scripts/Story/StealthKillUnlock.cs` | Evento/cinemática Nivel 3 | Flag de unlock |
| FinalChoiceManager | `Assets/Scripts/Story/FinalChoiceManager.cs` | Escena final | onEscapeChosen / onFightChosen |
| PuzzleSolvedRelay | `Assets/Scripts/Puzzle/PuzzleSolvedRelay.cs` | Objeto Rewards | Lista de reward receivers |
| PuzzleReward_GiveKeycard | `Assets/Scripts/Puzzle/PuzzleReward_GiveKeycard.cs` | Rewards | Nivel de tarjeta |
| PuzzleReward_SetStoryFlag | `Assets/Scripts/Puzzle/PuzzleReward_SetStoryFlag.cs` | Rewards | Flag a activar |
| PuzzleReward_OpenAnimator | `Assets/Scripts/Puzzle/PuzzleReward_OpenAnimator.cs` | Rewards | Animator y trigger |
| KeycardLevel1Puzzle | `Assets/Scripts/Puzzle/KeycardLevel1Puzzle.cs` | `PF_Puzzle_Keycard_Lv1` | correctSequence + UnityEvents |
| KeycardLevel2Puzzle | `Assets/Scripts/Puzzle/KeycardLevel2Puzzle.cs` | `PF_Puzzle_Keycard_Lv2` | bool[] correctStates |
| KeycardLevel3Puzzle | `Assets/Scripts/Puzzle/KeycardLevel3Puzzle.cs` | `PF_Puzzle_Keycard_Lv3` | requiredNodeCount |
| HiddenCodePuzzle | `Assets/Scripts/Puzzle/HiddenCodePuzzle.cs` | `PF_Puzzle_HiddenCode` | correctCode + UnityEvents |
| GeneratorSwitch | `Assets/Scripts/Puzzle/GeneratorSwitch.cs` | `PF_GeneratorSwitch` | FuseBox + onGeneratorStarted |
| FuseBox | `Assets/Scripts/Puzzle/FuseBox.cs` | `PF_FuseBox` | requiredFuseItemId, requiredFuseCount |
| StoryFlagGate | `Assets/Scripts/Story/StoryFlagGate.cs` | Objetos dependientes de flag | requiredFlag, targetObject |
| TensionProfileApplier | `Assets/Scripts/Enemy/TensionProfileApplier.cs` | Inicio de escena | LevelTensionProfile |

## 9. Debug y testeo

| Script | Ruta exacta | Dónde se coloca | Referencias clave |
|---|---|---|---|
| DebugGameplaySettings | `Assets/Scripts/Debug/DebugGameplaySettings.cs` | GameplayDebug | Flags globales de debug |
| NoiseDebugMonitor | `Assets/Scripts/Debug/NoiseDebugMonitor.cs` | GameplayDebug | Escucha NoiseSystem |
| SearchZoneDebugGizmos | `Assets/Scripts/Debug/SearchZoneDebugGizmos.cs` | Cada SearchZone | Solo debug |
| BossDebugPanel | `Assets/Scripts/Debug/BossDebugPanel.cs` | Canvas debug o GameplayDebug | BossController, runToPointTarget |
| GameplayDebugHotkeys | `Assets/Scripts/Debug/GameplayDebugHotkeys.cs` | GameplayDebug | BossDebugPanel, fakeNoiseOrigin |
| TensionDebugUI | `Assets/Scripts/Debug/TensionDebugUI.cs` | Canvas_Debug | TMP_Text + Slider |
| EnemyStateDebugUI | `Assets/Scripts/Debug/EnemyStateDebugUI.cs` | Canvas_Debug | Camera, enemyMask, TMP_Text |

## 10. Qué poner exactamente por escena

| Escena | Bloques base | Elementos específicos |
|---|---|---|
| Bootstrap | PF_BootstrapManagers + Canvas global si es persistente | Managers, audio, directores de IA y debug |
| Level_01 | PF_Player + UI jugable | FlashlightPickup, PhoneInteract, TutorialTriggers, AutoObjectiveSetter, 1 Checkpoint |
| Level_02 | PF_Player + PF_Enemy_Base | HideSpots, SearchZone, Puzzle Keycard Lv1, MirrorInteract, checkpoints, objetos lanzables |
| Level_03 | PF_Player + PF_Enemy_Base + PF_Boss | StealthKillUnlock, enemigo especial con KeyDrop, WalkieTalkieInteract, Door Keycard, BossRunToPointSequence |
| Level_04 | PF_Player + PF_Enemy_Base + PF_Boss | HiddenCodePuzzle, FuseBox, FusePickup, GeneratorSwitch, ElevatorAccess, BossSpawnPoints, RuleZones, SearchZones, TensionProfileApplier |
| Final_Level | PF_Player + PF_Boss + cámara 3ª persona | FinalChoiceManager, BossFightManager, BossWeakPointInteract, BossFightStarter, BossVulnerabilityTrigger, EndingLoader |

## 11. Orden de montaje exacto

1. Crear carpetas de `Assets/Scripts` y `Prefabs`.
2. Montar `PF_BootstrapManagers` y comprobar que Bootstrap no da errores.
3. Montar `PF_Player` completo y dejar movimiento + interacción funcionando.
4. Montar `Canvas_Gameplay` y conectar UI.
5. Montar `PF_Enemy_Base` con NavMeshAgent, EyePoint y patrulla mínima.
6. Montar HideSpot, puertas, pickups y objetos lanzables.
7. Montar `PF_Boss` y probar BossController manualmente.
8. Crear `SearchZone`, `BossSpawnPoints`, `RuleZones` y perfiles de tensión.
9. Construir cada escena siguiendo la tabla del apartado 10.
10. Hacer una pasada final con debug hotkeys, TensionDebugUI y EnemyStateDebugUI.

## 12. Errores típicos y dónde mirar

| Problema | Revisión rápida |
|---|---|
| El enemigo no se mueve | NavMesh sin hornear, patrolPoints vacíos, agent fuera del NavMesh |
| No se puede interactuar | InteractionSystem sin cámara o promptText, collider fuera de interactMask |
| El boss no aparece | TensionDirector.CanSpawnBoss false, BossSpawnRules bloqueando, spawn points inválidos |
| El escondite falla | HidePoint/ExitPoint/InspectPoint mal colocados o collider inaccesible |
| Save/Load restaura mal | Tag Player ausente, CharacterController no se desactiva al reposicionar |

## Nota final

Mantén los prefabs base genéricos y asigna referencias de escena (patrolPoints, targets concretos, endPoints, safe rooms, etc.) dentro de cada escena. Eso evita prefabs rotos, referencias perdidas y peleas absurdas con Unity.

