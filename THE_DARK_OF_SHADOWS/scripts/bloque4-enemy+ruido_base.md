Perfecto. Vamos con el **Bloque 4 — Enemy + ruido base**.
Aquí está el corazón de la IA básica y del sistema de ruido sobre el que se apoya casi todo.

Te lo dejo en orden para que compile con el menor drama posible.

---

# 1) NoisePriority.cs

**Ruta:** `Assets/Scripts/Enemy/NoisePriority.cs`
**Se coloca en:** archivo suelto, no se añade a GameObject

```csharp
public enum NoisePriority
{
    Low = 0,
    Medium = 1,
    High = 2,
    Critical = 3
}
```

---

# 2) NoiseSourceType.cs

**Ruta:** `Assets/Scripts/Enemy/NoiseSourceType.cs`
**Se coloca en:** archivo suelto, no se añade a GameObject

```csharp
public enum NoiseSourceType
{
    Footstep,
    ThrowableImpact,
    Door,
    Generator,
    ScriptedEvent,
    PlayerAction
}
```

---

# 3) NoiseEventData.cs

**Ruta:** `Assets/Scripts/Enemy/NoiseEventData.cs`
**Se coloca en:** archivo suelto, no se añade a GameObject

```csharp
using UnityEngine;

public struct NoiseEventData
{
    public Vector3 Position;
    public float Intensity;
    public NoisePriority Priority;
    public NoiseSourceType SourceType;
    public Transform SourceTransform;

    public NoiseEventData(
        Vector3 position,
        float intensity,
        NoisePriority priority,
        NoiseSourceType sourceType,
        Transform sourceTransform = null)
    {
        Position = position;
        Intensity = intensity;
        Priority = priority;
        SourceType = sourceType;
        SourceTransform = sourceTransform;
    }
}
```

---

# 4) NoiseSystem.cs

**Ruta:** `Assets/Scripts/Enemy/NoiseSystem.cs`
**Se coloca en:** archivo estático, no se añade a GameObject

```csharp
using System;
using UnityEngine;

public static class NoiseSystem
{
    public static Action<NoiseEventData> OnNoise;

    public static void Emit(
        Vector3 position,
        float intensity,
        NoiseSourceType sourceType = NoiseSourceType.PlayerAction,
        NoisePriority priority = NoisePriority.Medium,
        Transform sourceTransform = null)
    {
        NoiseEventData data = new NoiseEventData(
            position,
            intensity,
            priority,
            sourceType,
            sourceTransform
        );

        OnNoise?.Invoke(data);
    }
}
```

---

# 5) EnemyTacticalRole.cs

**Ruta:** `Assets/Scripts/Enemy/EnemyTacticalRole.cs`
**Se coloca en:** archivo suelto, no se añade a GameObject

```csharp
public enum EnemyTacticalRole
{
    None,
    Pursuer,
    Flanker,
    Searcher
}
```

---

# 6) EnemyInvestigationMemory.cs

**Ruta:** `Assets/Scripts/Enemy/EnemyInvestigationMemory.cs`
**Se coloca en:** `PF_Enemy_Base`

```csharp
using UnityEngine;

public class EnemyInvestigationMemory : MonoBehaviour
{
    public bool HasTarget => hasTarget;
    public Vector3 LastKnownPosition => lastKnownPosition;
    public NoisePriority CurrentPriority => currentPriority;
    public float LastUpdateTime => lastUpdateTime;

    private bool hasTarget;
    private Vector3 lastKnownPosition;
    private NoisePriority currentPriority;
    private float lastUpdateTime;

    public bool TrySetInvestigationTarget(Vector3 position, NoisePriority priority)
    {
        if (!hasTarget)
        {
            SetTarget(position, priority);
            return true;
        }

        if (priority > currentPriority)
        {
            SetTarget(position, priority);
            return true;
        }

        if (priority == currentPriority)
        {
            float currentDist = Vector3.Distance(transform.position, lastKnownPosition);
            float newDist = Vector3.Distance(transform.position, position);

            if (newDist < currentDist)
            {
                SetTarget(position, priority);
                return true;
            }
        }

        return false;
    }

    public void ForceSetTarget(Vector3 position, NoisePriority priority)
    {
        SetTarget(position, priority);
    }

    public void ClearTarget()
    {
        hasTarget = false;
        currentPriority = NoisePriority.Low;
        lastKnownPosition = Vector3.zero;
        lastUpdateTime = 0f;
    }

    private void SetTarget(Vector3 position, NoisePriority priority)
    {
        hasTarget = true;
        lastKnownPosition = position;
        currentPriority = priority;
        lastUpdateTime = Time.time;
    }
}
```

---

# 7) EnemyCommunicationSystem.cs

**Ruta:** `Assets/Scripts/Enemy/EnemyCommunicationSystem.cs`
**Se coloca en:** `PF_Enemy_Base`

```csharp
using UnityEngine;

public class EnemyCommunicationSystem : MonoBehaviour
{
    [SerializeField] private float communicationRadius = 12f;
    [SerializeField] private LayerMask enemyMask;

    private EnemyAI enemyAI;

    private void Awake()
    {
        enemyAI = GetComponent<EnemyAI>();
    }

    public void BroadcastPlayerSighted(Vector3 playerPosition)
    {
        Collider[] hits = Physics.OverlapSphere(
            transform.position,
            communicationRadius,
            enemyMask,
            QueryTriggerInteraction.Collide
        );

        foreach (Collider hit in hits)
        {
            EnemyAI otherEnemy = hit.GetComponentInParent<EnemyAI>();

            if (otherEnemy == null || otherEnemy == enemyAI)
                continue;

            otherEnemy.ReceiveSharedAlert(playerPosition, NoisePriority.Critical);
        }
    }

    public void BroadcastInvestigationPoint(Vector3 point, NoisePriority priority)
    {
        Collider[] hits = Physics.OverlapSphere(
            transform.position,
            communicationRadius,
            enemyMask,
            QueryTriggerInteraction.Collide
        );

        foreach (Collider hit in hits)
        {
            EnemyAI otherEnemy = hit.GetComponentInParent<EnemyAI>();

            if (otherEnemy == null || otherEnemy == enemyAI)
                continue;

            otherEnemy.ReceiveSharedInvestigation(point, priority);
        }
    }
}
```

---

# 8) EnemyHideSpotSearch.cs

**Ruta:** `Assets/Scripts/Enemy/EnemyHideSpotSearch.cs`
**Se coloca en:** `PF_Enemy_Base`

```csharp
using System.Collections.Generic;
using UnityEngine;

public class EnemyHideSpotSearch : MonoBehaviour
{
    [SerializeField] private float searchRadius = 8f;
    [SerializeField] private int maxHideSpotsToCheck = 2;
    [SerializeField] private LayerMask hideSpotMask;

    public List<HideSpot> FindNearbyHideSpots(Vector3 center)
    {
        List<HideSpot> result = new List<HideSpot>();

        Collider[] hits = Physics.OverlapSphere(center, searchRadius, hideSpotMask, QueryTriggerInteraction.Collide);
        List<HideSpotDistance> candidates = new List<HideSpotDistance>();

        foreach (Collider hit in hits)
        {
            HideSpot spot = hit.GetComponentInParent<HideSpot>();
            if (spot == null)
                continue;

            float dist = Vector3.Distance(center, spot.transform.position);
            candidates.Add(new HideSpotDistance(spot, dist));
        }

        candidates.Sort((a, b) => a.Distance.CompareTo(b.Distance));

        for (int i = 0; i < candidates.Count && i < maxHideSpotsToCheck; i++)
        {
            result.Add(candidates[i].Spot);
        }

        return result;
    }

    private struct HideSpotDistance
    {
        public HideSpot Spot;
        public float Distance;

        public HideSpotDistance(HideSpot spot, float distance)
        {
            Spot = spot;
            Distance = distance;
        }
    }
}
```

---

# 9) EnemyFlankHelper.cs

**Ruta:** `Assets/Scripts/Enemy/EnemyFlankHelper.cs`
**Se coloca en:** `PF_Enemy_Base`

```csharp
using UnityEngine;
using UnityEngine.AI;

public class EnemyFlankHelper : MonoBehaviour
{
    [SerializeField] private float flankDistance = 4f;
    [SerializeField] private float sampleRadius = 2f;

    public bool TryGetFlankPoint(Vector3 targetPosition, Vector3 playerForwardEstimate, out Vector3 flankPoint)
    {
        flankPoint = targetPosition;

        Vector3 right = Vector3.Cross(Vector3.up, playerForwardEstimate.normalized);
        if (right.sqrMagnitude < 0.01f)
            right = transform.right;

        Vector3 candidateA = targetPosition + right * flankDistance;
        Vector3 candidateB = targetPosition - right * flankDistance;

        bool validA = NavMesh.SamplePosition(candidateA, out NavMeshHit hitA, sampleRadius, NavMesh.AllAreas);
        bool validB = NavMesh.SamplePosition(candidateB, out NavMeshHit hitB, sampleRadius, NavMesh.AllAreas);

        if (validA && validB)
        {
            flankPoint = Random.value > 0.5f ? hitA.position : hitB.position;
            return true;
        }

        if (validA)
        {
            flankPoint = hitA.position;
            return true;
        }

        if (validB)
        {
            flankPoint = hitB.position;
            return true;
        }

        return false;
    }
}
```

---

# 10) TimedNoiseMemory.cs

**Ruta:** `Assets/Scripts/Enemy/TimedNoiseMemory.cs`
**Se coloca en:** `PF_Enemy_Base`

```csharp
using UnityEngine;

public class TimedNoiseMemory : MonoBehaviour
{
    [SerializeField] private float noiseLifetime = 8f;

    public bool HasValidNoise => hasNoise && Time.time - lastNoiseTime <= noiseLifetime;
    public Vector3 LastNoisePosition => lastNoisePosition;
    public NoisePriority LastNoisePriority => lastNoisePriority;
    public float LastNoiseTime => lastNoiseTime;

    private bool hasNoise;
    private Vector3 lastNoisePosition;
    private NoisePriority lastNoisePriority;
    private float lastNoiseTime;

    public void StoreNoise(Vector3 position, NoisePriority priority)
    {
        hasNoise = true;
        lastNoisePosition = position;
        lastNoisePriority = priority;
        lastNoiseTime = Time.time;
    }

    public bool IsNoiseExpired()
    {
        return !HasValidNoise;
    }

    public void Clear()
    {
        hasNoise = false;
        lastNoisePosition = Vector3.zero;
        lastNoisePriority = NoisePriority.Low;
        lastNoiseTime = 0f;
    }
}
```

---

# 11) EnemyAnimatorBridge.cs

**Ruta:** `Assets/Scripts/Enemy/EnemyAnimatorBridge.cs`
**Se coloca en:** `PF_Enemy_Base`

```csharp
using UnityEngine;
using UnityEngine.AI;

[RequireComponent(typeof(Animator))]
public class EnemyAnimatorBridge : MonoBehaviour
{
    [SerializeField] private EnemyAI enemyAI;
    [SerializeField] private NavMeshAgent agent;
    [SerializeField] private Animator animator;

    [Header("Parameters")]
    [SerializeField] private string speedParam = "Speed";
    [SerializeField] private string alertParam = "Alert";
    [SerializeField] private string chaseParam = "Chasing";

    private void Reset()
    {
        animator = GetComponent<Animator>();
        agent = GetComponent<NavMeshAgent>();
        enemyAI = GetComponent<EnemyAI>();
    }

    private void Update()
    {
        if (animator == null || agent == null || enemyAI == null)
            return;

        float normalizedSpeed = agent.velocity.magnitude / Mathf.Max(agent.speed, 0.01f);
        animator.SetFloat(speedParam, normalizedSpeed);

        bool alert = enemyAI.CurrentState == EnemyAI.State.Investigate ||
                     enemyAI.CurrentState == EnemyAI.State.Search ||
                     enemyAI.CurrentState == EnemyAI.State.Chase;

        bool chasing = enemyAI.CurrentState == EnemyAI.State.Chase;

        animator.SetBool(alertParam, alert);
        animator.SetBool(chaseParam, chasing);
    }

    public void TriggerKill()
    {
        if (animator != null)
            animator.SetTrigger("Kill");
    }

    public void TriggerDeath()
    {
        if (animator != null)
            animator.SetTrigger("Die");
    }
}
```

---

# 12) EnemyAI.cs

**Ruta:** `Assets/Scripts/Enemy/EnemyAI.cs`
**Se coloca en:** `PF_Enemy_Base`

```csharp
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

[RequireComponent(typeof(NavMeshAgent))]
[RequireComponent(typeof(EnemyInvestigationMemory))]
[RequireComponent(typeof(EnemyCommunicationSystem))]
[RequireComponent(typeof(EnemyHideSpotSearch))]
[RequireComponent(typeof(EnemyFlankHelper))]
[RequireComponent(typeof(TimedNoiseMemory))]
public class EnemyAI : MonoBehaviour
{
    public enum State
    {
        Patrol,
        Investigate,
        Search,
        Chase
    }

    [Header("References")]
    [SerializeField] private NavMeshAgent agent;
    [SerializeField] private Transform player;
    [SerializeField] private Transform eyePoint;

    [Header("Patrol")]
    [SerializeField] private Transform[] patrolPoints;
    [SerializeField] private float patrolWaitTime = 1.5f;

    [Header("Vision")]
    [SerializeField] private float viewDistance = 12f;
    [SerializeField, Range(1f, 180f)] private float viewAngle = 90f;

    [Header("Hearing")]
    [SerializeField] private float hearingRadius = 14f;
    [SerializeField] private float minNoiseToInvestigate = 0.2f;

    [Header("Movement")]
    [SerializeField] private float patrolSpeed = 2f;
    [SerializeField] private float investigateSpeed = 3f;
    [SerializeField] private float searchSpeed = 2.5f;
    [SerializeField] private float chaseSpeed = 4.5f;

    [Header("Search")]
    [SerializeField] private float searchDuration = 5f;
    [SerializeField] private float loseSightGraceTime = 1.5f;
    [SerializeField] private float hideSpotKillDistance = 1.5f;

    [Header("Catch")]
    [SerializeField] private float catchDistance = 1.4f;

    [Header("Tactics")]
    [SerializeField] private EnemyTacticalRole tacticalRole = EnemyTacticalRole.None;
    [SerializeField] private float flankPlayerForwardMemorySeconds = 2f;

    public State CurrentState { get; private set; } = State.Patrol;
    public State DebugCurrentState => CurrentState;

    private EnemyInvestigationMemory memory;
    private EnemyCommunicationSystem comms;
    private EnemyHideSpotSearch hideSpotSearch;
    private EnemyFlankHelper flankHelper;
    private TimedNoiseMemory timedNoiseMemory;

    private int patrolIndex;
    private float patrolWaitTimer;
    private float searchTimer;
    private float lostSightTimer;
    private Vector3 lastKnownPlayerPosition;
    private bool registeredAsAlert;

    private HideSpot knownPlayerHideSpot;
    private bool goingToKnownHideSpot;

    private readonly Queue<HideSpot> hideSpotQueue = new Queue<HideSpot>();
    private HideSpot currentHideSpotTarget;

    private Vector3 cachedPlayerForwardEstimate;
    private float playerForwardEstimateTimer;

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

        memory = GetComponent<EnemyInvestigationMemory>();
        comms = GetComponent<EnemyCommunicationSystem>();
        hideSpotSearch = GetComponent<EnemyHideSpotSearch>();
        flankHelper = GetComponent<EnemyFlankHelper>();
        timedNoiseMemory = GetComponent<TimedNoiseMemory>();
    }

    private void OnEnable()
    {
        NoiseSystem.OnNoise += OnNoiseHeard;
        PlayerHide.OnPlayerEnteredHideSpot += OnPlayerEnteredHideSpot;
    }

    private void OnDisable()
    {
        NoiseSystem.OnNoise -= OnNoiseHeard;
        PlayerHide.OnPlayerEnteredHideSpot -= OnPlayerEnteredHideSpot;

        if (registeredAsAlert)
            MusicManager.Instance?.RegisterEnemyAlert(false);
    }

    private void Update()
    {
        if (player == null)
            return;

        UpdatePlayerForwardEstimate();
        bool canSeePlayer = CanSeePlayer();

        switch (CurrentState)
        {
            case State.Patrol:
                UpdatePatrol(canSeePlayer);
                break;
            case State.Investigate:
                UpdateInvestigate(canSeePlayer);
                break;
            case State.Search:
                UpdateSearch(canSeePlayer);
                break;
            case State.Chase:
                UpdateChase(canSeePlayer);
                break;
        }

        TryCatchPlayer();
    }

    private void UpdatePatrol(bool canSeePlayer)
    {
        SetSpeed(patrolSpeed);

        if (canSeePlayer)
        {
            EnterChase();
            return;
        }

        if (memory.HasTarget)
        {
            CurrentState = State.Investigate;
            return;
        }

        if (patrolPoints == null || patrolPoints.Length == 0)
            return;

        Transform target = patrolPoints[patrolIndex];
        agent.SetDestination(target.position);

        if (!agent.pathPending && agent.remainingDistance <= agent.stoppingDistance + 0.15f)
        {
            patrolWaitTimer += Time.deltaTime;

            if (patrolWaitTimer >= patrolWaitTime)
            {
                patrolIndex = (patrolIndex + 1) % patrolPoints.Length;
                patrolWaitTimer = 0f;
            }
        }
    }

    private void UpdateInvestigate(bool canSeePlayer)
    {
        SetSpeed(investigateSpeed);

        if (canSeePlayer)
        {
            EnterChase();
            return;
        }

        if (!memory.HasTarget || (timedNoiseMemory != null && timedNoiseMemory.IsNoiseExpired()))
        {
            EnterSearch(transform.position);
            return;
        }

        agent.SetDestination(memory.LastKnownPosition);

        if (!agent.pathPending && agent.remainingDistance <= agent.stoppingDistance + 0.25f)
        {
            EnterSearch(memory.LastKnownPosition);
        }
    }

    private void UpdateSearch(bool canSeePlayer)
    {
        SetSpeed(searchSpeed);

        if (canSeePlayer)
        {
            EnterChase();
            return;
        }

        if (goingToKnownHideSpot && knownPlayerHideSpot != null)
        {
            agent.SetDestination(knownPlayerHideSpot.InspectPoint.position);

            float dist = Vector3.Distance(transform.position, knownPlayerHideSpot.InspectPoint.position);
            if (dist <= hideSpotKillDistance)
            {
                if (knownPlayerHideSpot.HasHiddenPlayer())
                {
                    GameManager.Instance?.GameOverCaught();
                    return;
                }

                goingToKnownHideSpot = false;
                knownPlayerHideSpot = null;
            }

            return;
        }

        if (currentHideSpotTarget != null)
        {
            agent.SetDestination(currentHideSpotTarget.InspectPoint.position);

            if (!agent.pathPending && agent.remainingDistance <= agent.stoppingDistance + 0.25f)
            {
                if (currentHideSpotTarget.HasHiddenPlayer())
                {
                    GameManager.Instance?.GameOverCaught();
                    return;
                }

                currentHideSpotTarget = null;
            }

            return;
        }

        if (hideSpotQueue.Count > 0)
        {
            currentHideSpotTarget = hideSpotQueue.Dequeue();
            return;
        }

        agent.SetDestination(lastKnownPlayerPosition);

        searchTimer -= Time.deltaTime;
        if (searchTimer <= 0f)
        {
            ExitAlertIfNeeded();
            memory.ClearTarget();
            timedNoiseMemory?.Clear();
            tacticalRole = EnemyTacticalRole.None;
            CurrentState = State.Patrol;
        }
    }

    private void UpdateChase(bool canSeePlayer)
    {
        SetSpeed(chaseSpeed);

        switch (tacticalRole)
        {
            case EnemyTacticalRole.Flanker:
                if (flankHelper != null &&
                    flankHelper.TryGetFlankPoint(lastKnownPlayerPosition, cachedPlayerForwardEstimate, out Vector3 flankPoint))
                {
                    agent.SetDestination(flankPoint);
                }
                else
                {
                    agent.SetDestination(player.position);
                }
                break;

            default:
                agent.SetDestination(player.position);
                break;
        }

        if (canSeePlayer)
        {
            lastKnownPlayerPosition = player.position;
            lostSightTimer = 0f;
        }
        else
        {
            lostSightTimer += Time.deltaTime;

            if (lostSightTimer >= loseSightGraceTime)
            {
                EnterSearch(lastKnownPlayerPosition);
            }
        }
    }

    private void OnNoiseHeard(NoiseEventData noise)
    {
        float distance = Vector3.Distance(transform.position, noise.Position);
        if (distance > hearingRadius)
            return;

        if (noise.Intensity < minNoiseToInvestigate)
            return;

        if (CurrentState == State.Chase)
            return;

        bool accepted = memory.TrySetInvestigationTarget(noise.Position, noise.Priority);

        if (accepted)
        {
            timedNoiseMemory.StoreNoise(noise.Position, noise.Priority);
            CurrentState = State.Investigate;
            lastKnownPlayerPosition = noise.Position;

            if (noise.SourceType == NoiseSourceType.ThrowableImpact)
                QueueNearbyHideSpots(noise.Position);

            if (noise.Priority >= NoisePriority.High)
                comms.BroadcastInvestigationPoint(noise.Position, noise.Priority);

            if (TensionDirector.Instance != null)
                TensionDirector.Instance.AddTension(8f);
        }
    }

    private void OnPlayerEnteredHideSpot(PlayerHide playerHide, HideSpot spot)
    {
        if (player == null || playerHide == null || spot == null)
            return;

        if (playerHide.transform != player)
            return;

        if (CanSeePlayerDirect(player.position))
        {
            knownPlayerHideSpot = spot;
            goingToKnownHideSpot = true;
            lastKnownPlayerPosition = spot.InspectPoint.position;
            CurrentState = State.Search;
        }
    }

    public void ReceiveSharedAlert(Vector3 playerPosition, NoisePriority priority)
    {
        if (CurrentState == State.Chase)
            return;

        memory.ForceSetTarget(playerPosition, priority);
        lastKnownPlayerPosition = playerPosition;
        CurrentState = State.Investigate;
    }

    public void ReceiveSharedInvestigation(Vector3 point, NoisePriority priority)
    {
        if (CurrentState == State.Chase)
            return;

        if (memory.TrySetInvestigationTarget(point, priority))
        {
            lastKnownPlayerPosition = point;
            CurrentState = State.Investigate;
        }
    }

    public void SetTacticalRole(EnemyTacticalRole role)
    {
        tacticalRole = role;
    }

    public EnemyTacticalRole GetTacticalRole()
    {
        return tacticalRole;
    }

    public bool TryGetDebugTargets(out Vector3 investigatePos, out Vector3 lastKnownPos, out Vector3 hideSpotPos, out Vector3 searchZonePos)
    {
        investigatePos = memory != null && memory.HasTarget ? memory.LastKnownPosition : Vector3.zero;
        lastKnownPos = lastKnownPlayerPosition;
        hideSpotPos = knownPlayerHideSpot != null ? knownPlayerHideSpot.InspectPoint.position : Vector3.zero;
        searchZonePos = Vector3.zero;
        return true;
    }

    private void EnterChase()
    {
        CurrentState = State.Chase;
        lastKnownPlayerPosition = player.position;
        lostSightTimer = 0f;
        goingToKnownHideSpot = false;
        knownPlayerHideSpot = null;
        currentHideSpotTarget = null;
        hideSpotQueue.Clear();
        timedNoiseMemory?.Clear();

        if (!registeredAsAlert)
        {
            registeredAsAlert = true;
            MusicManager.Instance?.RegisterEnemyAlert(true);
        }

        comms.BroadcastPlayerSighted(player.position);

        if (TensionDirector.Instance != null)
            TensionDirector.Instance.AddTension(15f);
    }

    private void EnterSearch(Vector3 where)
    {
        CurrentState = State.Search;
        lastKnownPlayerPosition = where;
        searchTimer = searchDuration;
    }

    private void ExitAlertIfNeeded()
    {
        if (!registeredAsAlert)
            return;

        registeredAsAlert = false;
        MusicManager.Instance?.RegisterEnemyAlert(false);
    }

    private void QueueNearbyHideSpots(Vector3 center)
    {
        hideSpotQueue.Clear();
        List<HideSpot> spots = hideSpotSearch.FindNearbyHideSpots(center);

        foreach (HideSpot spot in spots)
            hideSpotQueue.Enqueue(spot);
    }

    private bool CanSeePlayer()
    {
        if (player == null)
            return false;

        PlayerHide playerHide = player.GetComponent<PlayerHide>();
        if (playerHide != null && playerHide.IsHidden)
            return false;

        return CanSeePlayerDirect(player.position);
    }

    private bool CanSeePlayerDirect(Vector3 playerWorldPosition)
    {
        Vector3 origin = eyePoint != null ? eyePoint.position : transform.position + Vector3.up * 1.6f;
        Vector3 target = playerWorldPosition + Vector3.up * 1.2f;
        Vector3 direction = target - origin;

        float distance = direction.magnitude;
        if (distance > viewDistance)
            return false;

        float angle = Vector3.Angle(transform.forward, direction.normalized);
        if (angle > viewAngle * 0.5f)
            return false;

        if (Physics.Raycast(origin, direction.normalized, out RaycastHit hit, distance, ~0, QueryTriggerInteraction.Ignore))
            return hit.transform.CompareTag("Player");

        return false;
    }

    private void UpdatePlayerForwardEstimate()
    {
        if (player == null)
            return;

        Vector3 forward = player.forward;
        forward.y = 0f;

        if (forward.sqrMagnitude > 0.01f)
        {
            cachedPlayerForwardEstimate = forward.normalized;
            playerForwardEstimateTimer = flankPlayerForwardMemorySeconds;
        }
        else if (playerForwardEstimateTimer > 0f)
        {
            playerForwardEstimateTimer -= Time.deltaTime;
        }
    }

    private void SetSpeed(float speed)
    {
        if (agent.speed != speed)
            agent.speed = speed;
    }

    private void TryCatchPlayer()
    {
        if (player == null)
            return;

        if (Vector3.Distance(transform.position, player.position) <= catchDistance)
            GameManager.Instance?.GameOverCaught();
    }

    public bool IsInChase()
    {
        return CurrentState == State.Chase;
    }
}
```

---

# 13) StealthKillTarget.cs

**Ruta:** `Assets/Scripts/Enemy/StealthKillTarget.cs`
**Se coloca en:** enemigos rematables

```csharp
using UnityEngine;
using UnityEngine.Events;

public class StealthKillTarget : MonoBehaviour, IInteractable
{
    [Header("Requirements")]
    [SerializeField] private string requiredUnlockFlag = "StealthKillUnlocked";
    [SerializeField] private float requiredDistance = 2f;
    [SerializeField] private float backAngleThreshold = 0.5f;

    [Header("Prompt")]
    [SerializeField] private string prompt = "Press E to stealth kill";
    [SerializeField] private string unavailablePrompt = "";

    [Header("Events")]
    [SerializeField] private UnityEvent onStealthKill;
    [SerializeField] private bool destroyAfterKill = true;

    private EnemyAI enemyAI;
    private bool killed;

    private void Awake()
    {
        enemyAI = GetComponent<EnemyAI>();
    }

    public string GetPrompt()
    {
        return CanBeKilledByPlayer() ? prompt : unavailablePrompt;
    }

    public bool CanInteract(InteractionSystem interactor)
    {
        return CanBeKilledByPlayer(interactor.transform);
    }

    public void Interact(InteractionSystem interactor)
    {
        if (!CanBeKilledByPlayer(interactor.transform))
            return;

        killed = true;
        onStealthKill?.Invoke();

        if (destroyAfterKill)
            Destroy(gameObject);
        else
            gameObject.SetActive(false);
    }

    private bool CanBeKilledByPlayer()
    {
        GameObject player = GameObject.FindGameObjectWithTag("Player");
        if (player == null)
            return false;

        return CanBeKilledByPlayer(player.transform);
    }

    private bool CanBeKilledByPlayer(Transform player)
    {
        if (killed || player == null)
            return false;

        if (StoryFlags.Instance == null || !StoryFlags.Instance.HasFlag(requiredUnlockFlag))
            return false;

        if (enemyAI != null && enemyAI.IsInChase())
            return false;

        Vector3 toPlayer = (player.position - transform.position).normalized;
        float dot = Vector3.Dot(transform.forward, toPlayer);

        bool playerIsBehind = dot > backAngleThreshold;
        bool inRange = Vector3.Distance(player.position, transform.position) <= requiredDistance;

        return playerIsBehind && inRange;
    }
}
```

---

# 14) StealthKillTargetKeyDrop.cs

**Ruta:** `Assets/Scripts/Enemy/StealthKillTargetKeyDrop.cs`
**Se coloca en:** enemigo especial del nivel 3

```csharp
using UnityEngine;
using UnityEngine.Events;

public class StealthKillTargetKeyDrop : MonoBehaviour
{
    [SerializeField] private string keyId = "Level3EnemyKey";
    [SerializeField] private UnityEvent onKeyDropped;

    public void DropKey()
    {
        InventorySystem.Instance?.AddKey(keyId);
        onKeyDropped?.Invoke();
    }
}
```

---

# 15) Dónde poner cada script de este bloque

## En PF_Enemy_Base

Añade:

* `NavMeshAgent`
* `Animator`
* `EnemyAI`
* `EnemyInvestigationMemory`
* `EnemyCommunicationSystem`
* `EnemyHideSpotSearch`
* `EnemyFlankHelper`
* `TimedNoiseMemory`
* `EnemyAnimatorBridge`

Y en modo debug:

* `AIDebugGizmos` cuando lo añadamos en el bloque de debug

## En enemigos especiales

Añade además:

* `StealthKillTarget`
* `StealthKillTargetKeyDrop` si debe soltar llave

---

# 16) Referencias que debes asignar

## EnemyAI

* `player` → Player
* `eyePoint` → hijo `EyePoint`
* `patrolPoints` → puntos de patrulla de la escena
* `viewDistance` / `viewAngle` → ajusta a tu gusto

## EnemyCommunicationSystem

* `enemyMask` → layer `Enemy`

## EnemyHideSpotSearch

* `hideSpotMask` → layer `HideSpot`

## EnemyAnimatorBridge

* `animator` → animator del enemigo
* `agent` → NavMeshAgent
* `enemyAI` → EnemyAI

## StealthKillTarget

* `requiredUnlockFlag` → `"StealthKillUnlocked"`

## StealthKillTargetKeyDrop

Conecta `DropKey()` al `onStealthKill` del `StealthKillTarget`

---

# 17) Prueba rápida de este bloque

## Patrulla

* el enemigo debe ir entre puntos

## Visión

* si te ve, entra en `Chase`

## Ruido

* si lanzas objeto, investiga

## Escondite visto

* si te ve entrar, va al `InspectPoint` del escondite
* si sigues dentro cuando llega, game over

## Comunicación

* un enemigo detecta
* otro cercano recibe la alerta

## Stealth kill

* solo por detrás
* solo si está desbloqueado
* no si el enemigo está en chase

---

# 18) Nota importante

Este bloque usa `MusicManager` y `TensionDirector`.
Si todavía no los has pegado, te puede salir error de compilación. Es normal. En los siguientes bloques quedan resueltos.

---

# Siguiente bloque

En el siguiente mensaje te paso el **Bloque 5 — Boss**, con código completo de:

* `BossController.cs`
* `BossAnimatorBridge.cs`
* `BossTimedEncounter.cs`
* `BossRunToPointSequence.cs`
* `BossSpawnPoint.cs`
* `BossSpawnManager.cs`
* `BossSpawnRules.cs`
* `BossSpawnRuleZone.cs`
* `BossEncounterDirector.cs`
* `BossTensionSpawner.cs`
* `BossFightManager.cs`
* `BossWeakPointInteract.cs`
* `BossFightStarter.cs`
* `BossVulnerabilityTrigger.cs`
* `EndingLoader.cs`

Seguimos con ese.
