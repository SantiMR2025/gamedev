Perfecto. Vamos a hacerlo como debe hacerse: **orden exacto de pegado y compilación** para que Unity no te escupa 200 errores de golpe y tú no tengas que adivinar cuál fue el primer dominó.

La regla es esta: primero pegas **tipos base y enums**, luego **core**, después **player**, luego **interacción/items**, luego **enemigos**, luego **boss**, y al final **UI/debug**.

---

# Fase 0 — Antes de pegar nada

Dentro de `Assets`, crea estas carpetas:

```text
Assets/Scripts/Core
Assets/Scripts/Player
Assets/Scripts/Interaction
Assets/Scripts/Items
Assets/Scripts/Enemy
Assets/Scripts/Boss
Assets/Scripts/Puzzle
Assets/Scripts/Story
Assets/Scripts/Audio
Assets/Scripts/UI
Assets/Scripts/Debug
```

Y estas escenas/carpeta si quieres dejarlo ordenado:

```text
Assets/Scenes
Assets/Prefabs
Assets/ScriptableObjects/TensionProfiles
```

---

# Fase 1 — Tipos base y archivos “sin dependencias”

Pega **estos primero**, porque casi todo lo demás los usa.

## 1. Core / tipos

1. `AccessCardLevel.cs`
2. `InventoryEntry` y `InventorySnapshot` ya van dentro de `SaveSystem` en tu versión actual, así que no hace falta separarlos
3. `SerializableVector3` ya va dentro de `SaveSystem`

## 2. Interaction / interfaces

4. `IInteractable.cs`

## 3. Enemy / enums y structs

5. `NoisePriority.cs`
6. `NoiseSourceType.cs`
7. `NoiseEventData.cs`
8. `EnemyTacticalRole.cs`

## 4. Boss / enums

9. `BossEncounterType.cs`

## 5. Puzzle / interfaces

10. `IPuzzleRewardReceiver.cs`

### Resultado esperado

Después de esta fase, Unity debería compilar **sin errores** porque son archivos base.

---

# Fase 2 — Core real

Ahora pega los sistemas centrales.

1. `StoryFlags.cs`
2. `ObjectiveTracker.cs`
3. `SceneLoader.cs`
4. `InventorySystem.cs`
5. `GameManager.cs`
6. `SaveSystem.cs`
7. `CheckpointSystem.cs`
8. `CheckpointTrigger.cs`

### Si aparece error aquí

Lo más probable es:

* `PlayerStamina` todavía no existe y `SaveSystem` / `CheckpointSystem` lo referencian

Eso es normal si pegas `SaveSystem` antes del bloque player.
La solución correcta es: si Unity marca error por `PlayerStamina`, sigue con la fase 3 enseguida. No te pongas a reescribir nada.

---

# Fase 3 — Player

Ahora pegas todo lo del jugador.

1. `PlayerStamina.cs`
2. `PlayerMovement.cs`
3. `PlayerItemHolder.cs`
4. `FlashlightController.cs`
5. `PlayerHide.cs`
6. `PlayerAnimationBridge.cs`

### Importante

`PlayerHide.cs` usa `HideSpot`, que todavía no existe si no has pegado interacción.
Así que aquí puede saltar error momentáneo por `HideSpot`.

No pasa nada. Sigue a la fase 4.

---

# Fase 4 — Interaction + Items

Esto suele limpiar muchos errores del bloque player.

## Interaction

1. `InteractionSystem.cs`
2. `DoorInteract.cs`
3. `KeycardDoor.cs`
4. `HideSpot.cs`
5. `PhoneInteract.cs`
6. `MirrorInteract.cs`
7. `WalkieTalkieInteract.cs`
8. `ElevatorAccess.cs`
9. `SavePointInteract.cs`

## Items

10. `KeyPickup.cs`
11. `KeycardPickup.cs`
12. `FlashlightPickup.cs`
13. `FusePickup.cs`
14. `ThrowableItem.cs`

### Resultado esperado

Tras esta fase, deberían desaparecer:

* errores de `HideSpot`
* errores de `InteractionSystem`
* errores de `IInteractable`

Si sale error en `ThrowableItem`, revisa que ya exista `NoiseSystem`. Si no, sigue a la fase 5.

---

# Fase 5 — Enemy base

Ahora pegas el bloque de ruido e IA.

1. `NoiseSystem.cs`
2. `EnemyInvestigationMemory.cs`
3. `EnemyCommunicationSystem.cs`
4. `EnemyHideSpotSearch.cs`
5. `EnemyFlankHelper.cs`
6. `TimedNoiseMemory.cs`
7. `EnemyAnimatorBridge.cs`
8. `EnemyAI.cs`
9. `StealthKillTarget.cs`
10. `StealthKillTargetKeyDrop.cs`

### Posibles errores aquí

Si `EnemyAI.cs` marca errores por:

* `MusicManager`
* `TensionDirector`

No lo toques todavía. Eso se resuelve en fases 6 y 7.

---

# Fase 6 — Boss

Ahora pegas todo el sistema boss.

1. `BossController.cs`
2. `BossAnimatorBridge.cs`
3. `BossTimedEncounter.cs`
4. `BossRunToPointSequence.cs`
5. `BossSpawnPoint.cs`
6. `BossSpawnManager.cs`
7. `BossSpawnRules.cs`
8. `BossSpawnRuleZone.cs`
9. `BossEncounterDirector.cs`
10. `BossTensionSpawner.cs`
11. `BossFightManager.cs`
12. `BossWeakPointInteract.cs`
13. `BossFightStarter.cs`
14. `BossVulnerabilityTrigger.cs`
15. `ThirdPersonBossFightCamera.cs`
16. `EndingLoader.cs`
17. `BossEncounterBlockerManual.cs`
18. `BossSpawnRulesRelay.cs`

### Posibles errores aquí

Si salen errores por:

* `LevelTensionProfile`
* `TensionDirector`
* `MusicManager`

Todavía faltan piezas. Sigue a la fase 7.

---

# Fase 7 — Enemy avanzado / tensión / búsqueda

Ahora pegas los sistemas que conectan IA avanzada y boss dinámico.

1. `LevelTensionProfile.cs`
2. `TensionDirector.cs`
3. `TensionProfileApplier.cs`
4. `SearchZone.cs`
5. `EnemySearchCoordinator.cs`
6. `EnemySearchReservationSystem.cs`

### Resultado esperado

Aquí deberían desaparecer muchos errores de:

* `EnemyAI`
* `BossEncounterDirector`
* `BossTensionSpawner`
* `BossEncounterBlockerManual`

---

# Fase 8 — Story + Puzzles

Ahora pegas narrativa, objetivos y puzzles.

## Story

1. `TutorialUI.cs`
2. `TutorialTrigger.cs`
3. `SubtitleTrigger.cs`
4. `CinematicSignalRelay.cs`
5. `AutoObjectiveSetter.cs`
6. `FinalChoiceManager.cs`
7. `StealthKillUnlock.cs`

## Puzzle

8. `PuzzleSolvedRelay.cs`
9. `PuzzleReward_GiveKeycard.cs`
10. `PuzzleReward_SetStoryFlag.cs`
11. `PuzzleReward_OpenAnimator.cs`
12. `KeycardLevel1Puzzle.cs`
13. `KeycardLevel2Puzzle.cs`
14. `KeycardLevel3Puzzle.cs`
15. `HiddenCodePuzzle.cs`
16. `FuseBox.cs`
17. `GeneratorSwitch.cs`

### Resultado esperado

Aquí ya deberías tener compilando toda la lógica de niveles.

---

# Fase 9 — Audio

1. `MusicManager.cs`
2. `AmbientAudioManager.cs`
3. `FootstepNoiseEmitter.cs`
4. `FootstepSurfaceAudio.cs`

### Resultado esperado

Aquí deberían desaparecer los errores de `EnemyAI` y `BossController` relacionados con música y ruido.

---

# Fase 10 — UI

1. `SubtitleUI.cs`
2. `ObjectiveUI.cs`
3. `PauseMenuUI.cs`
4. `GameOverUI.cs`

### Posibles errores

Si `PauseMenuUI` o `GameOverUI` dan error, revisa que `GameManager` ya esté compilado. Si has seguido el orden, debería estar bien.

---

# Fase 11 — Debug

1. `DebugGameplaySettings.cs`
2. `NoiseDebugMonitor.cs`
3. `AIDebugGizmos.cs`
4. `SearchZoneDebugGizmos.cs`
5. `BossSpawnPointGizmo.cs`
6. `BossDebugPanel.cs`
7. `TensionDebugUI.cs`
8. `EnemyStateDebugUI.cs`
9. `GameplayDebugHotkeys.cs`

### Esta fase debe ir al final

Porque debug toca casi todos los sistemas.

---

# Orden resumido ultra práctico

Si quieres la versión corta:

```text
1. Enums + interfaces
2. Core
3. Player
4. Interaction + Items
5. Enemy base
6. Boss
7. Tensión + búsqueda avanzada
8. Story + Puzzle
9. Audio
10. UI
11. Debug
```

---

# Cómo pegar sin volverte loco

Hazlo así:

## Método recomendado

* pega **máximo 5–8 scripts**
* espera a que Unity compile
* mira la consola
* sigue al siguiente grupo

No pegues 70 scripts de golpe salvo que te guste el deporte extremo.

---

# Qué errores ignorar temporalmente

Durante el proceso, estos errores son “normales” si aún no has llegado a la fase correspondiente:

* `The type or namespace name 'HideSpot' could not be found`
* `The type or namespace name 'MusicManager' could not be found`
* `The type or namespace name 'TensionDirector' could not be found`
* `The type or namespace name 'LevelTensionProfile' could not be found`

Eso solo significa que todavía no has pegado ese bloque.

---

# Qué errores NO deberías ignorar

Si aparece algo como esto, sí hay que corregirlo al momento:

* dos clases con el mismo nombre
* dos enums con el mismo nombre
* un script cuyo nombre no coincide con el nombre de la clase
* llaves `{}` mal cerradas
* usar `MonoBehaviour` sin `using UnityEngine;`
* copiar dos veces el mismo script en carpetas diferentes

---

# Regla crítica de Unity

**El nombre del archivo debe coincidir exactamente con la clase pública.**

Ejemplo correcto:

* archivo: `GameManager.cs`
* clase pública: `public class GameManager : MonoBehaviour`

Si no coincide, Unity se enfada con razón.

---

# Después de compilar todo

Cuando ya no haya errores de compilación, monta en este orden:

1. `Bootstrap_Managers`
2. `Canvas_Gameplay`
3. `PF_Player`
4. `PF_Enemy_Base`
5. `PF_Boss`
6. interactivos
7. puzzles
8. triggers
9. debug

---

# Mi consejo más práctico

Empieza ahora mismo con este suborden exacto:

```text
A. AccessCardLevel
B. IInteractable
C. NoisePriority / NoiseSourceType / NoiseEventData / EnemyTacticalRole / BossEncounterType / IPuzzleRewardReceiver
D. StoryFlags / ObjectiveTracker / SceneLoader / InventorySystem / GameManager
E. PlayerStamina / PlayerMovement / PlayerItemHolder / FlashlightController
F. InteractionSystem / HideSpot
G. PlayerHide / PlayerAnimationBridge
H. SaveSystem / CheckpointSystem / CheckpointTrigger
```

Ese miniorden te deja la base funcional con el mínimo número de errores cruzados.

Si quieres, el siguiente paso te lo puedo dar todavía más mascado: **una lista de “pega estos 10 scripts primero” con casillas de verificación**, para que vayas tachando mientras montas.
