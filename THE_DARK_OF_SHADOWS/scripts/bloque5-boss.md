Perfecto. Vamos con el **Bloque 5 — Boss**.
Aquí está todo lo relacionado con el boss: persecuciones, spawns, reglas, encuentros y combate final.

Te lo dejo en formato **copiar y pegar**.

---

# 1) BossEncounterType.cs

**Ruta:** `Assets/Scripts/Boss/BossEncounterType.cs`
**Se coloca en:** archivo suelto, no se añade a GameObject

```csharp
public enum BossEncounterType
{
    TimedChase,
    RunToPoint,
    GuidedJumpscare
}
```

---

# 2) BossController.cs

**Ruta:** `Assets/Scripts/Boss/BossController.cs`
**Se coloca en:** `PF_Boss`

```csharp
using System.Collections;
using UnityEngine;
using UnityEngine.AI;

[RequireComponent(typeof(NavMeshAgent))]
public class BossController : MonoBehaviour
{
    public enum BossMode
    {
        Inactive,
        SpawnSequence,
        TimedChase,
        ChaseToTargetPoint,
        FinalFight
    }

    [Header("References")]
    [SerializeField] private NavMeshAgent agent;
    [SerializeField] private Transform player;
    [SerializeField] private Transform chaseEndPoint;

    [Header("Movement")]
    [SerializeField] private float chaseSpeed = 6.5f;
    [SerializeField] private float catchDistance = 1.6f;

    [Header("Timed Chase")]
    [SerializeField] private float minChaseTime = 15f;
    [SerializeField] private float maxChaseTime = 25f;

    [Header("Spawn Sequence")]
    [SerializeField] private float spawnFreezeTime = 1f;
    [SerializeField] private GameObject runWarningUI;
    [SerializeField] private AudioSource stingerAudio;

    public BossMode CurrentMode { get; private set; } = BossMode.Inactive;
    public bool IsActiveChase =>
        CurrentMode == BossMode.TimedChase ||
        CurrentMode == BossMode.ChaseToTargetPoint;

    private float timedChaseTimer;

    private void Reset()
    {
        agent = GetComponent<NavMeshAgent>();
    }

    private void Awake()
    {
        if (agent == null)
            agent = GetComponent<NavMeshAgent>();

        if (player == null)
            player = GameObject.FindGameObjectWithTag("Player")?.transform;

        agent.enabled = false;
        gameObject.SetActive(false);
    }

    private void Update()
    {
        if (player == null || !agent.enabled)
            return;

        switch (CurrentMode)
        {
            case BossMode.TimedChase:
                UpdateTimedChase();
                break;

            case BossMode.ChaseToTargetPoint:
                UpdateTargetPointChase();
                break;

            case BossMode.FinalFight:
                break;
        }

        TryCatchPlayer();
    }

    public void BeginTimedChase()
    {
        gameObject.SetActive(true);
        StartCoroutine(SpawnSequenceRoutine(BossMode.TimedChase));
    }

    public void BeginChaseToPoint(Transform endPoint)
    {
        chaseEndPoint = endPoint;
        gameObject.SetActive(true);
        StartCoroutine(SpawnSequenceRoutine(BossMode.ChaseToTargetPoint));
    }

    public void BeginFinalFightMode()
    {
        gameObject.SetActive(true);
        agent.enabled = false;
        CurrentMode = BossMode.FinalFight;
        MusicManager.Instance?.ForceChase(true);
    }

    public void StopBoss()
    {
        CurrentMode = BossMode.Inactive;

        if (agent.enabled)
            agent.ResetPath();

        agent.enabled = false;
        MusicManager.Instance?.SetBossActive(false);
        MusicManager.Instance?.ForceChase(false);
        gameObject.SetActive(false);
    }

    private IEnumerator SpawnSequenceRoutine(BossMode modeAfterSpawn)
    {
        CurrentMode = BossMode.SpawnSequence;

        if (runWarningUI != null)
            runWarningUI.SetActive(false);

        if (stingerAudio != null)
            stingerAudio.Play();

        Time.timeScale = 0f;

        yield return new WaitForSecondsRealtime(spawnFreezeTime);

        if (runWarningUI != null)
            runWarningUI.SetActive(true);

        yield return new WaitForSecondsRealtime(1f);

        if (runWarningUI != null)
            runWarningUI.SetActive(false);

        Time.timeScale = 1f;

        agent.enabled = true;
        agent.speed = chaseSpeed;
        MusicManager.Instance?.SetBossActive(true);

        CurrentMode = modeAfterSpawn;

        if (modeAfterSpawn == BossMode.TimedChase)
            timedChaseTimer = Random.Range(minChaseTime, maxChaseTime);
    }

    private void UpdateTimedChase()
    {
        agent.SetDestination(player.position);

        timedChaseTimer -= Time.deltaTime;
        if (timedChaseTimer <= 0f)
            StopBoss();
    }

    private void UpdateTargetPointChase()
    {
        agent.SetDestination(player.position);

        if (chaseEndPoint != null && Vector3.Distance(player.position, chaseEndPoint.position) <= 1.75f)
        {
            StopBoss();
        }
    }

    private void TryCatchPlayer()
    {
        if (player == null)
            return;

        if (Vector3.Distance(transform.position, player.position) <= catchDistance)
        {
            GameManager.Instance?.GameOverCaught();
        }
    }

    public string GetDebugInfo()
    {
        return $"Mode: {CurrentMode} | ActiveChase: {IsActiveChase}";
    }
}
```

---

# 3) BossAnimatorBridge.cs

**Ruta:** `Assets/Scripts/Boss/BossAnimatorBridge.cs`
**Se coloca en:** `PF_Boss`

```csharp
using UnityEngine;
using UnityEngine.AI;

[RequireComponent(typeof(Animator))]
public class BossAnimatorBridge : MonoBehaviour
{
    [SerializeField] private BossController bossController;
    [SerializeField] private NavMeshAgent agent;
    [SerializeField] private Animator animator;

    [Header("Parameters")]
    [SerializeField] private string speedParam = "Speed";
    [SerializeField] private string activeParam = "Active";
    [SerializeField] private string finalFightParam = "FinalFight";

    private void Reset()
    {
        animator = GetComponent<Animator>();
        agent = GetComponent<NavMeshAgent>();
        bossController = GetComponent<BossController>();
    }

    private void Update()
    {
        if (animator == null || bossController == null)
            return;

        float speed = 0f;

        if (agent != null && agent.enabled)
            speed = agent.velocity.magnitude / Mathf.Max(agent.speed, 0.01f);

        animator.SetFloat(speedParam, speed);
        animator.SetBool(activeParam, bossController.IsActiveChase);
        animator.SetBool(finalFightParam, bossController.CurrentMode == BossController.BossMode.FinalFight);
    }

    public void TriggerSpawn()
    {
        animator?.SetTrigger("Spawn");
    }

    public void TriggerAttack()
    {
        animator?.SetTrigger("Attack");
    }

    public void TriggerDisappear()
    {
        animator?.SetTrigger("Disappear");
    }
}
```

---

# 4) BossTimedEncounter.cs

**Ruta:** `Assets/Scripts/Boss/BossTimedEncounter.cs`
**Se coloca en:** trigger de escena

```csharp
using UnityEngine;

public class BossTimedEncounter : MonoBehaviour
{
    [SerializeField] private BossController bossController;
    [SerializeField] private bool oneUseOnly = true;

    private bool used;

    private void OnTriggerEnter(Collider other)
    {
        if (used && oneUseOnly)
            return;

        if (!other.CompareTag("Player"))
            return;

        if (bossController != null)
        {
            bossController.BeginTimedChase();
            used = true;
        }
    }
}
```

---

# 5) BossRunToPointSequence.cs

**Ruta:** `Assets/Scripts/Boss/BossRunToPointSequence.cs`
**Se coloca en:** trigger del nivel 3

```csharp
using UnityEngine;
using UnityEngine.Events;

public class BossRunToPointSequence : MonoBehaviour
{
    [SerializeField] private BossController bossController;
    [SerializeField] private Transform endPoint;
    [SerializeField] private UnityEvent onSequenceStarted;
    [SerializeField] private bool oneUseOnly = true;

    private bool used;

    private void OnTriggerEnter(Collider other)
    {
        if (used && oneUseOnly)
            return;

        if (!other.CompareTag("Player"))
            return;

        if (bossController != null && endPoint != null)
        {
            bossController.BeginChaseToPoint(endPoint);
            onSequenceStarted?.Invoke();
            used = true;
        }
    }
}
```

---

# 6) BossSpawnPoint.cs

**Ruta:** `Assets/Scripts/Boss/BossSpawnPoint.cs`
**Se coloca en:** `PF_BossSpawnPoint`

```csharp
using UnityEngine;

public class BossSpawnPoint : MonoBehaviour
{
    [SerializeField] private bool enabledForSpawn = true;
    public bool EnabledForSpawn => enabledForSpawn;

    public Vector3 Position => transform.position;
    public Quaternion Rotation => transform.rotation;
}
```

---

# 7) BossSpawnManager.cs

**Ruta:** `Assets/Scripts/Boss/BossSpawnManager.cs`
**Se coloca en:** `Bootstrap_Managers/AIDirectors/BossSpawnManager`

```csharp
using System.Collections.Generic;
using UnityEngine;

public class BossSpawnManager : MonoBehaviour
{
    public static BossSpawnManager Instance { get; private set; }

    [SerializeField] private BossSpawnPoint[] spawnPoints;
    [SerializeField] private Transform player;
    [SerializeField] private float minSpawnDistance = 10f;
    [SerializeField] private float maxSpawnDistance = 30f;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
    }

    public bool TryGetSpawnPoint(out BossSpawnPoint point)
    {
        point = null;

        if (player == null)
            player = GameObject.FindGameObjectWithTag("Player")?.transform;

        if (spawnPoints == null || spawnPoints.Length == 0 || player == null)
            return false;

        List<BossSpawnPoint> valid = new List<BossSpawnPoint>();

        foreach (BossSpawnPoint sp in spawnPoints)
        {
            if (sp == null || !sp.EnabledForSpawn)
                continue;

            float dist = Vector3.Distance(player.position, sp.Position);

            if (dist < minSpawnDistance || dist > maxSpawnDistance)
                continue;

            valid.Add(sp);
        }

        if (valid.Count == 0)
            return false;

        point = valid[Random.Range(0, valid.Count)];
        return true;
    }
}
```

---

# 8) BossSpawnRules.cs

**Ruta:** `Assets/Scripts/Boss/BossSpawnRules.cs`
**Se coloca en:** `Bootstrap_Managers/AIDirectors/BossSpawnRules`

```csharp
using UnityEngine;

public class BossSpawnRules : MonoBehaviour
{
    public static BossSpawnRules Instance { get; private set; }

    public bool InCinematic { get; private set; }
    public bool InCriticalPuzzle { get; private set; }
    public bool NearCheckpoint { get; private set; }
    public bool InSafeRoom { get; private set; }

    public bool CanSpawnBoss =>
        !InCinematic &&
        !InCriticalPuzzle &&
        !NearCheckpoint &&
        !InSafeRoom;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
    }

    public void SetCinematicState(bool value) => InCinematic = value;
    public void SetCriticalPuzzleState(bool value) => InCriticalPuzzle = value;
    public void SetNearCheckpointState(bool value) => NearCheckpoint = value;
    public void SetSafeRoomState(bool value) => InSafeRoom = value;
}
```

---

# 9) BossSpawnRuleZone.cs

**Ruta:** `Assets/Scripts/Boss/BossSpawnRuleZone.cs`
**Se coloca en:** safe rooms, checkpoints, puzzles críticos

```csharp
using UnityEngine;

public class BossSpawnRuleZone : MonoBehaviour
{
    public enum RuleType
    {
        SafeRoom,
        CheckpointBlock,
        CriticalPuzzle
    }

    [SerializeField] private RuleType ruleType;

    private void OnTriggerEnter(Collider other)
    {
        if (!other.CompareTag("Player") || BossSpawnRules.Instance == null)
            return;

        SetRule(true);
    }

    private void OnTriggerExit(Collider other)
    {
        if (!other.CompareTag("Player") || BossSpawnRules.Instance == null)
            return;

        SetRule(false);
    }

    private void SetRule(bool value)
    {
        switch (ruleType)
        {
            case RuleType.SafeRoom:
                BossSpawnRules.Instance.SetSafeRoomState(value);
                break;
            case RuleType.CheckpointBlock:
                BossSpawnRules.Instance.SetNearCheckpointState(value);
                break;
            case RuleType.CriticalPuzzle:
                BossSpawnRules.Instance.SetCriticalPuzzleState(value);
                break;
        }
    }
}
```

---

# 10) BossEncounterDirector.cs

**Ruta:** `Assets/Scripts/Boss/BossEncounterDirector.cs`
**Se coloca en:** `Bootstrap_Managers/AIDirectors/BossEncounterDirector`

```csharp
using UnityEngine;

public class BossEncounterDirector : MonoBehaviour
{
    public static BossEncounterDirector Instance { get; private set; }

    [SerializeField] private BossController bossController;
    [SerializeField] private Transform optionalRunTargetPoint;
    [SerializeField] private Transform[] guidedJumpscarePoints;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
    }

    public void TryTriggerEncounter()
    {
        if (bossController == null ||
            TensionDirector.Instance == null ||
            BossSpawnManager.Instance == null)
            return;

        if (!TensionDirector.Instance.CanSpawnBoss)
            return;

        if (BossSpawnRules.Instance != null && !BossSpawnRules.Instance.CanSpawnBoss)
            return;

        if (!BossSpawnManager.Instance.TryGetSpawnPoint(out BossSpawnPoint spawnPoint))
            return;

        bossController.transform.position = spawnPoint.Position;
        bossController.transform.rotation = spawnPoint.Rotation;

        BossEncounterType type = ChooseEncounterType();

        switch (type)
        {
            case BossEncounterType.RunToPoint:
                if (optionalRunTargetPoint != null)
                    bossController.BeginChaseToPoint(optionalRunTargetPoint);
                else
                    bossController.BeginTimedChase();
                break;

            case BossEncounterType.GuidedJumpscare:
                TriggerGuidedJumpscare(spawnPoint);
                break;

            default:
                bossController.BeginTimedChase();
                break;
        }

        TensionDirector.Instance.ConsumeBossSpawn();
    }

    private BossEncounterType ChooseEncounterType()
    {
        LevelTensionProfile profile = TensionDirector.Instance.ActiveProfile;
        if (profile == null)
            return BossEncounterType.TimedChase;

        float timed = profile.timedChaseWeight;
        float run = profile.runToPointWeight;
        float jump = profile.guidedJumpscareWeight;

        float total = timed + run + jump;
        if (total <= 0.001f)
            return BossEncounterType.TimedChase;

        float value = Random.value * total;

        if (value < timed)
            return BossEncounterType.TimedChase;

        value -= timed;
        if (value < run)
            return BossEncounterType.RunToPoint;

        return BossEncounterType.GuidedJumpscare;
    }

    private void TriggerGuidedJumpscare(BossSpawnPoint fallbackPoint)
    {
        if (guidedJumpscarePoints != null && guidedJumpscarePoints.Length > 0)
        {
            Transform point = guidedJumpscarePoints[Random.Range(0, guidedJumpscarePoints.Length)];
            if (point != null)
            {
                bossController.transform.position = point.position;
                bossController.transform.rotation = point.rotation;
            }
        }
        else
        {
            bossController.transform.position = fallbackPoint.Position;
            bossController.transform.rotation = fallbackPoint.Rotation;
        }

        bossController.BeginTimedChase();
    }
}
```

---

# 11) BossTensionSpawner.cs

**Ruta:** `Assets/Scripts/Boss/BossTensionSpawner.cs`
**Se coloca en:** `Bootstrap_Managers/AIDirectors/BossTensionSpawner`

```csharp
using UnityEngine;

public class BossTensionSpawner : MonoBehaviour
{
    [SerializeField] private float checkInterval = 2f;

    private float timer;

    private void Update()
    {
        if (BossEncounterDirector.Instance == null)
            return;

        timer -= Time.deltaTime;
        if (timer > 0f)
            return;

        timer = checkInterval;
        BossEncounterDirector.Instance.TryTriggerEncounter();
    }
}
```

---

# 12) BossFightManager.cs

**Ruta:** `Assets/Scripts/Boss/BossFightManager.cs`
**Se coloca en:** arena final

```csharp
using UnityEngine;
using UnityEngine.Events;

[System.Serializable]
public class BossPhaseEvent : UnityEvent<int> { }

public class BossFightManager : MonoBehaviour
{
    [Header("Fight Settings")]
    [SerializeField] private int totalPhases = 3;
    [SerializeField] private float vulnerableDuration = 6f;

    [Header("Events")]
    [SerializeField] private BossPhaseEvent onPhaseStarted;
    [SerializeField] private BossPhaseEvent onBossVulnerable;
    [SerializeField] private BossPhaseEvent onBossHit;
    [SerializeField] private UnityEvent onFightWon;
    [SerializeField] private UnityEvent onFightLost;

    public int CurrentPhase { get; private set; } = 0;
    public bool IsVulnerable { get; private set; }
    public bool FightStarted { get; private set; }
    public bool FightFinished { get; private set; }

    private float vulnerableTimer;

    private void Update()
    {
        if (!FightStarted || FightFinished)
            return;

        if (IsVulnerable)
        {
            vulnerableTimer -= Time.deltaTime;

            if (vulnerableTimer <= 0f)
            {
                IsVulnerable = false;
            }
        }
    }

    public void StartFight()
    {
        if (FightStarted)
            return;

        FightStarted = true;
        CurrentPhase = 1;
        StartPhase(CurrentPhase);
    }

    private void StartPhase(int phase)
    {
        IsVulnerable = false;
        onPhaseStarted?.Invoke(phase);
    }

    public void SetBossVulnerable()
    {
        if (!FightStarted || FightFinished)
            return;

        IsVulnerable = true;
        vulnerableTimer = vulnerableDuration;
        onBossVulnerable?.Invoke(CurrentPhase);
    }

    public void HitBossWeakPoint()
    {
        if (!FightStarted || FightFinished || !IsVulnerable)
            return;

        IsVulnerable = false;
        onBossHit?.Invoke(CurrentPhase);

        if (CurrentPhase >= totalPhases)
        {
            WinFight();
        }
        else
        {
            CurrentPhase++;
            StartPhase(CurrentPhase);
        }
    }

    public void LoseFight()
    {
        if (FightFinished)
            return;

        FightFinished = true;
        onFightLost?.Invoke();
    }

    private void WinFight()
    {
        FightFinished = true;
        StoryFlags.Instance?.SetFlag("Ending_BossDefeated");
        onFightWon?.Invoke();
    }
}
```

---

# 13) BossWeakPointInteract.cs

**Ruta:** `Assets/Scripts/Boss/BossWeakPointInteract.cs`
**Se coloca en:** punto débil del boss

```csharp
using UnityEngine;

public class BossWeakPointInteract : MonoBehaviour, IInteractable
{
    [SerializeField] private BossFightManager fightManager;
    [SerializeField] private string prompt = "Press E to strike";
    [SerializeField] private string unavailablePrompt = "";

    public string GetPrompt()
    {
        if (fightManager != null && fightManager.IsVulnerable)
            return prompt;

        return unavailablePrompt;
    }

    public bool CanInteract(InteractionSystem interactor)
    {
        return fightManager != null &&
               fightManager.FightStarted &&
               !fightManager.FightFinished &&
               fightManager.IsVulnerable;
    }

    public void Interact(InteractionSystem interactor)
    {
        if (fightManager == null)
            return;

        fightManager.HitBossWeakPoint();
    }
}
```

---

# 14) BossFightStarter.cs

**Ruta:** `Assets/Scripts/Boss/BossFightStarter.cs`
**Se coloca en:** trigger o evento del inicio del combate final

```csharp
using UnityEngine;

public class BossFightStarter : MonoBehaviour
{
    [SerializeField] private BossFightManager fightManager;
    [SerializeField] private ThirdPersonBossFightCamera thirdPersonCamera;
    [SerializeField] private BossController bossController;
    [SerializeField] private Transform cameraTarget;
    [SerializeField] private bool oneUseOnly = true;

    private bool used;

    public void StartFight()
    {
        if (used && oneUseOnly)
            return;

        if (bossController != null)
            bossController.BeginFinalFightMode();

        if (thirdPersonCamera != null)
        {
            if (cameraTarget != null)
                thirdPersonCamera.SetTarget(cameraTarget);

            thirdPersonCamera.Activate();
        }

        fightManager?.StartFight();
        used = true;
    }
}
```

---

# 15) BossVulnerabilityTrigger.cs

**Ruta:** `Assets/Scripts/Boss/BossVulnerabilityTrigger.cs`
**Se coloca en:** trigger o animation event del boss

```csharp
using UnityEngine;

public class BossVulnerabilityTrigger : MonoBehaviour
{
    [SerializeField] private BossFightManager fightManager;

    public void MakeBossVulnerable()
    {
        fightManager?.SetBossVulnerable();
    }
}
```

---

# 16) EndingLoader.cs

**Ruta:** `Assets/Scripts/Boss/EndingLoader.cs`
**Se coloca en:** escena final o manager del final

```csharp
using UnityEngine;

public class EndingLoader : MonoBehaviour
{
    [SerializeField] private string goodEndingScene = "Ending_Good";
    [SerializeField] private string badEndingScene = "Ending_Bad";

    public void LoadGoodEnding()
    {
        SceneLoader.Instance?.LoadSceneByName(goodEndingScene);
    }

    public void LoadBadEnding()
    {
        SceneLoader.Instance?.LoadSceneByName(badEndingScene);
    }
}
```

---

# 17) BossEncounterBlockerManual.cs

**Ruta:** `Assets/Scripts/Boss/BossEncounterBlockerManual.cs`
**Se coloca en:** Timeline, puzzles o eventos especiales

```csharp
using UnityEngine;

public class BossEncounterBlockerManual : MonoBehaviour
{
    public void BlockBossSpawns()
    {
        TensionDirector.Instance?.SetBossSpawnsBlocked(true);
    }

    public void UnblockBossSpawns()
    {
        TensionDirector.Instance?.SetBossSpawnsBlocked(false);
    }

    public void StartReliefWindow(float duration)
    {
        TensionDirector.Instance?.StartReliefWindow(duration);
    }
}
```

---

# 18) BossSpawnRulesRelay.cs

**Ruta:** `Assets/Scripts/Boss/BossSpawnRulesRelay.cs`
**Se coloca en:** Timeline o eventos de escena

```csharp
using UnityEngine;

public class BossSpawnRulesRelay : MonoBehaviour
{
    public void SetCinematicState(bool value)
    {
        BossSpawnRules.Instance?.SetCinematicState(value);
    }

    public void SetCriticalPuzzleState(bool value)
    {
        BossSpawnRules.Instance?.SetCriticalPuzzleState(value);
    }

    public void SetNearCheckpointState(bool value)
    {
        BossSpawnRules.Instance?.SetNearCheckpointState(value);
    }

    public void SetSafeRoomState(bool value)
    {
        BossSpawnRules.Instance?.SetSafeRoomState(value);
    }
}
```

---

# 19) Dónde poner cada script de este bloque

## En PF_Boss

Añade:

* `NavMeshAgent`
* `Animator`
* `BossController`
* `BossAnimatorBridge`

## En triggers de escena

Añade según el caso:

* `BossTimedEncounter`
* `BossRunToPointSequence`

## En Bootstrap / managers

Crea objetos vacíos para:

* `BossSpawnManager`
* `BossSpawnRules`
* `BossEncounterDirector`
* `BossTensionSpawner`

## En el final

Añade:

* `BossFightManager`
* `BossWeakPointInteract`
* `BossFightStarter`
* `BossVulnerabilityTrigger`
* `EndingLoader`

---

# 20) Referencias que debes asignar

## BossController

* `player` → Player
* `runWarningUI` → panel RUN / warning UI
* `stingerAudio` → audio del boss
* `chaseEndPoint` → solo si lo usas manualmente

## BossEncounterDirector

* `bossController` → PF_Boss de la escena
* `optionalRunTargetPoint` → punto final de persecución
* `guidedJumpscarePoints` → array opcional

## BossSpawnManager

* `spawnPoints` → todos los `BossSpawnPoint`
* `player` → Player

## BossFightStarter

* `fightManager`
* `thirdPersonCamera`
* `bossController`
* `cameraTarget`

## BossWeakPointInteract

* `fightManager`

## BossVulnerabilityTrigger

* `fightManager`

---

# 21) Prueba rápida de este bloque

## Boss timed chase

* trigger entra
* aparece boss
* sale warning
* persigue
* desaparece

## Run to point

* boss aparece
* persigue hasta `endPoint`
* desaparece al llegar

## Spawns dinámicos

* si hay puntos válidos, puede aparecer
* no debería aparecer en safe room o puzzle crítico

## Final fight

* arranca combate
* boss entra en vulnerable
* puedes golpear weak point
* cambia de fase
* al final carga ending

---

# 22) Dependencias de este bloque

Este bloque necesita:

* `TensionDirector`
* `LevelTensionProfile`
* `MusicManager`

Eso lo resolveremos en el siguiente bloque junto con:

* puzzles
* story
* audio
* UI
* debug

---

# Siguiente bloque

En el siguiente mensaje te paso el **Bloque 6 — Puzzle / Story / Audio / UI / Debug**, con lo que falta para cerrar el sistema completo.
