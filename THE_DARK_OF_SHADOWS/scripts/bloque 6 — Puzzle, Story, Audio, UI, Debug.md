Perfecto. Vamos con el **Bloque 6 — Puzzle / Story / Audio / UI / Debug**.
Este bloque cierra prácticamente todo lo que falta para que el proyecto quede montable y testeable de punta a punta.

Voy a incluir:

* puzzles
* historia / flujo
* audio
* UI
* tensión
* búsqueda avanzada
* debug

Te lo dejo en formato **copiar y pegar**.

---

# 1) LevelTensionProfile.cs

**Ruta:** `Assets/Scripts/Enemy/LevelTensionProfile.cs`
**Se coloca en:** archivo scriptable object, no en GameObject

```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "LevelTensionProfile", menuName = "DarkOfShadows/Level Tension Profile")]
public class LevelTensionProfile : ScriptableObject
{
    [Header("Tension")]
    public float passiveDecayPerSecond = 4f;
    public float bossSpawnThreshold = 75f;
    public float cooldownAfterBoss = 20f;
    public float reliefWindowDuration = 12f;

    [Header("Boss Chances")]
    [Range(0f, 1f)] public float timedChaseWeight = 0.6f;
    [Range(0f, 1f)] public float runToPointWeight = 0.3f;
    [Range(0f, 1f)] public float guidedJumpscareWeight = 0.1f;

    [Header("Restrictions")]
    public bool bossAllowed = true;
}
```

---

# 2) TensionDirector.cs

**Ruta:** `Assets/Scripts/Enemy/TensionDirector.cs`
**Se coloca en:** `Bootstrap_Managers/AIDirectors/TensionDirector`

```csharp
using UnityEngine;

public class TensionDirector : MonoBehaviour
{
    public static TensionDirector Instance { get; private set; }

    [Header("Runtime")]
    [SerializeField] private LevelTensionProfile activeProfile;
    [SerializeField] private float currentTension = 0f;
    [SerializeField] private float maxTension = 100f;

    public float CurrentTension => currentTension;
    public LevelTensionProfile ActiveProfile => activeProfile;

    public bool CanSpawnBoss =>
        activeProfile != null &&
        activeProfile.bossAllowed &&
        currentTension >= activeProfile.bossSpawnThreshold &&
        bossCooldownTimer <= 0f &&
        reliefTimer <= 0f &&
        !blockBossSpawns;

    private float bossCooldownTimer;
    private float reliefTimer;
    private bool blockBossSpawns;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
        DontDestroyOnLoad(gameObject);
    }

    private void Update()
    {
        float decay = activeProfile != null ? activeProfile.passiveDecayPerSecond : 4f;
        currentTension = Mathf.Max(0f, currentTension - decay * Time.deltaTime);

        if (bossCooldownTimer > 0f)
            bossCooldownTimer -= Time.deltaTime;

        if (reliefTimer > 0f)
            reliefTimer -= Time.deltaTime;
    }

    public void SetProfile(LevelTensionProfile profile)
    {
        activeProfile = profile;
    }

    public void AddTension(float amount)
    {
        currentTension = Mathf.Clamp(currentTension + amount, 0f, maxTension);
    }

    public void ConsumeBossSpawn()
    {
        currentTension = Mathf.Max(0f, currentTension - 40f);

        if (activeProfile != null)
        {
            bossCooldownTimer = activeProfile.cooldownAfterBoss;
            reliefTimer = activeProfile.reliefWindowDuration;
        }
        else
        {
            bossCooldownTimer = 20f;
            reliefTimer = 12f;
        }
    }

    public void StartReliefWindow(float duration = -1f)
    {
        if (duration > 0f)
            reliefTimer = duration;
        else if (activeProfile != null)
            reliefTimer = activeProfile.reliefWindowDuration;
        else
            reliefTimer = 12f;
    }

    public void SetBossSpawnsBlocked(bool blocked)
    {
        blockBossSpawns = blocked;
    }

    public void ResetTension()
    {
        currentTension = 0f;
        bossCooldownTimer = 0f;
        reliefTimer = 0f;
        blockBossSpawns = false;
    }
}
```

---

# 3) TensionProfileApplier.cs

**Ruta:** `Assets/Scripts/Enemy/TensionProfileApplier.cs`
**Se coloca en:** objeto vacío al inicio de cada escena

```csharp
using UnityEngine;

public class TensionProfileApplier : MonoBehaviour
{
    [SerializeField] private LevelTensionProfile profile;
    [SerializeField] private bool applyOnStart = true;

    private void Start()
    {
        if (applyOnStart)
            ApplyProfile();
    }

    public void ApplyProfile()
    {
        TensionDirector.Instance?.SetProfile(profile);
    }
}
```

---

# 4) SearchZone.cs

**Ruta:** `Assets/Scripts/Enemy/SearchZone.cs`
**Se coloca en:** zonas de búsqueda del nivel

```csharp
using UnityEngine;

public class SearchZone : MonoBehaviour
{
    [SerializeField] private Transform[] searchPoints;

    public Transform[] SearchPoints => searchPoints;

    public Vector3 GetClosestPoint(Vector3 origin)
    {
        if (searchPoints == null || searchPoints.Length == 0)
            return transform.position;

        float bestDist = float.MaxValue;
        Vector3 best = transform.position;

        foreach (Transform point in searchPoints)
        {
            if (point == null)
                continue;

            float dist = Vector3.Distance(origin, point.position);
            if (dist < bestDist)
            {
                bestDist = dist;
                best = point.position;
            }
        }

        return best;
    }

    public Vector3 GetRandomPoint()
    {
        if (searchPoints == null || searchPoints.Length == 0)
            return transform.position;

        Transform point = searchPoints[Random.Range(0, searchPoints.Length)];
        return point != null ? point.position : transform.position;
    }
}
```

---

# 5) EnemySearchCoordinator.cs

**Ruta:** `Assets/Scripts/Enemy/EnemySearchCoordinator.cs`
**Se coloca en:** `Bootstrap_Managers/AIDirectors/EnemySearchCoordinator`

```csharp
using System.Collections.Generic;
using UnityEngine;

public class EnemySearchCoordinator : MonoBehaviour
{
    public static EnemySearchCoordinator Instance { get; private set; }

    private readonly List<EnemyAI> registeredEnemies = new();

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
        DontDestroyOnLoad(gameObject);
    }

    public void RegisterEnemy(EnemyAI enemy)
    {
        if (enemy == null || registeredEnemies.Contains(enemy))
            return;

        registeredEnemies.Add(enemy);
    }

    public void UnregisterEnemy(EnemyAI enemy)
    {
        if (enemy == null)
            return;

        registeredEnemies.Remove(enemy);
    }

    public void AssignTacticalRoles(Vector3 targetPosition)
    {
        List<EnemyAI> nearby = GetNearbyEnemies(targetPosition, 20f);

        if (nearby.Count == 0)
            return;

        nearby.Sort((a, b) =>
            Vector3.Distance(a.transform.position, targetPosition)
            .CompareTo(Vector3.Distance(b.transform.position, targetPosition)));

        for (int i = 0; i < nearby.Count; i++)
        {
            EnemyAI enemy = nearby[i];

            if (i == 0)
                enemy.SetTacticalRole(EnemyTacticalRole.Pursuer);
            else if (i == 1)
                enemy.SetTacticalRole(EnemyTacticalRole.Flanker);
            else
                enemy.SetTacticalRole(EnemyTacticalRole.Searcher);
        }
    }

    public SearchZone FindBestSearchZone(Vector3 targetPosition, float maxDistance = 15f)
    {
        SearchZone[] zones = FindObjectsOfType<SearchZone>();
        SearchZone best = null;
        float bestDist = float.MaxValue;

        foreach (SearchZone zone in zones)
        {
            float dist = Vector3.Distance(zone.transform.position, targetPosition);
            if (dist < maxDistance && dist < bestDist)
            {
                best = zone;
                bestDist = dist;
            }
        }

        return best;
    }

    private List<EnemyAI> GetNearbyEnemies(Vector3 center, float radius)
    {
        List<EnemyAI> result = new();

        foreach (EnemyAI enemy in registeredEnemies)
        {
            if (enemy == null || !enemy.gameObject.activeInHierarchy)
                continue;

            float dist = Vector3.Distance(enemy.transform.position, center);
            if (dist <= radius)
                result.Add(enemy);
        }

        return result;
    }
}
```

---

# 6) EnemySearchReservationSystem.cs

**Ruta:** `Assets/Scripts/Enemy/EnemySearchReservationSystem.cs`
**Se coloca en:** `Bootstrap_Managers/AIDirectors/EnemySearchReservationSystem`

```csharp
using System.Collections.Generic;
using UnityEngine;

public class EnemySearchReservationSystem : MonoBehaviour
{
    public static EnemySearchReservationSystem Instance { get; private set; }

    private readonly Dictionary<EnemyAI, Vector3> reservedPoints = new();
    private readonly Dictionary<EnemyAI, SearchZone> reservedZones = new();

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
        DontDestroyOnLoad(gameObject);
    }

    public bool IsPointCovered(Vector3 point, float radius)
    {
        foreach (var pair in reservedPoints)
        {
            if (Vector3.Distance(pair.Value, point) <= radius)
                return true;
        }

        return false;
    }

    public bool IsZoneCovered(SearchZone zone)
    {
        if (zone == null)
            return false;

        foreach (var pair in reservedZones)
        {
            if (pair.Value == zone)
                return true;
        }

        return false;
    }

    public void ReservePoint(EnemyAI enemy, Vector3 point)
    {
        if (enemy == null)
            return;

        reservedPoints[enemy] = point;
    }

    public void ReserveZone(EnemyAI enemy, SearchZone zone)
    {
        if (enemy == null || zone == null)
            return;

        reservedZones[enemy] = zone;
    }

    public void ReleaseReservations(EnemyAI enemy)
    {
        if (enemy == null)
            return;

        reservedPoints.Remove(enemy);
        reservedZones.Remove(enemy);
    }
}
```

---

# 7) TutorialUI.cs

**Ruta:** `Assets/Scripts/Story/TutorialUI.cs`
**Se coloca en:** `Canvas_Gameplay/TutorialPanel`

```csharp
using System.Collections;
using TMPro;
using UnityEngine;

public class TutorialUI : MonoBehaviour
{
    public static TutorialUI Instance { get; private set; }

    [SerializeField] private CanvasGroup canvasGroup;
    [SerializeField] private TMP_Text messageText;
    [SerializeField] private float fadeDuration = 0.25f;

    private Coroutine currentRoutine;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;

        if (canvasGroup != null)
        {
            canvasGroup.alpha = 0f;
            canvasGroup.gameObject.SetActive(false);
        }
    }

    public void ShowMessage(string message, float duration = 3f)
    {
        if (currentRoutine != null)
            StopCoroutine(currentRoutine);

        currentRoutine = StartCoroutine(ShowRoutine(message, duration));
    }

    private IEnumerator ShowRoutine(string message, float duration)
    {
        if (canvasGroup == null || messageText == null)
            yield break;

        messageText.text = message;
        canvasGroup.gameObject.SetActive(true);

        yield return Fade(0f, 1f);
        yield return new WaitForSeconds(duration);
        yield return Fade(1f, 0f);

        canvasGroup.gameObject.SetActive(false);
    }

    private IEnumerator Fade(float from, float to)
    {
        float t = 0f;
        canvasGroup.alpha = from;

        while (t < fadeDuration)
        {
            t += Time.deltaTime;
            canvasGroup.alpha = Mathf.Lerp(from, to, t / fadeDuration);
            yield return null;
        }

        canvasGroup.alpha = to;
    }
}
```

---

# 8) TutorialTrigger.cs

**Ruta:** `Assets/Scripts/Story/TutorialTrigger.cs`
**Se coloca en:** triggers de tutorial

```csharp
using UnityEngine;

public class TutorialTrigger : MonoBehaviour
{
    [TextArea]
    [SerializeField] private string tutorialMessage = "Tutorial";
    [SerializeField] private float duration = 3f;
    [SerializeField] private bool oneUseOnly = true;

    private bool used;

    private void OnTriggerEnter(Collider other)
    {
        if (used && oneUseOnly)
            return;

        if (!other.CompareTag("Player"))
            return;

        TutorialUI.Instance?.ShowMessage(tutorialMessage, duration);

        if (oneUseOnly)
            used = true;
    }
}
```

---

# 9) SubtitleUI.cs

**Ruta:** `Assets/Scripts/UI/SubtitleUI.cs`
**Se coloca en:** `Canvas_Gameplay/SubtitlePanel`

```csharp
using System.Collections;
using TMPro;
using UnityEngine;

public class SubtitleUI : MonoBehaviour
{
    public static SubtitleUI Instance { get; private set; }

    [SerializeField] private CanvasGroup canvasGroup;
    [SerializeField] private TMP_Text subtitleText;
    [SerializeField] private float fadeDuration = 0.2f;

    private Coroutine currentRoutine;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;

        if (canvasGroup != null)
        {
            canvasGroup.alpha = 0f;
            canvasGroup.gameObject.SetActive(false);
        }
    }

    public void ShowSubtitle(string text, float duration)
    {
        if (currentRoutine != null)
            StopCoroutine(currentRoutine);

        currentRoutine = StartCoroutine(ShowRoutine(text, duration));
    }

    private IEnumerator ShowRoutine(string text, float duration)
    {
        if (canvasGroup == null || subtitleText == null)
            yield break;

        subtitleText.text = text;
        canvasGroup.gameObject.SetActive(true);

        yield return Fade(0f, 1f);
        yield return new WaitForSeconds(duration);
        yield return Fade(1f, 0f);

        canvasGroup.gameObject.SetActive(false);
    }

    private IEnumerator Fade(float from, float to)
    {
        float t = 0f;
        canvasGroup.alpha = from;

        while (t < fadeDuration)
        {
            t += Time.deltaTime;
            canvasGroup.alpha = Mathf.Lerp(from, to, t / fadeDuration);
            yield return null;
        }

        canvasGroup.alpha = to;
    }
}
```

---

# 10) SubtitleTrigger.cs

**Ruta:** `Assets/Scripts/Story/SubtitleTrigger.cs`
**Se coloca en:** triggers o eventos de escena

```csharp
using UnityEngine;

public class SubtitleTrigger : MonoBehaviour
{
    [TextArea]
    [SerializeField] private string subtitleText = "Subtitle";
    [SerializeField] private float duration = 3f;
    [SerializeField] private bool oneUseOnly = true;

    private bool used;

    private void OnTriggerEnter(Collider other)
    {
        if (used && oneUseOnly)
            return;

        if (!other.CompareTag("Player"))
            return;

        SubtitleUI.Instance?.ShowSubtitle(subtitleText, duration);

        if (oneUseOnly)
            used = true;
    }

    public void TriggerSubtitle()
    {
        SubtitleUI.Instance?.ShowSubtitle(subtitleText, duration);

        if (oneUseOnly)
            used = true;
    }
}
```

---

# 11) ObjectiveUI.cs

**Ruta:** `Assets/Scripts/UI/ObjectiveUI.cs`
**Se coloca en:** `Canvas_Gameplay/ObjectivePanel`

```csharp
using TMPro;
using UnityEngine;

public class ObjectiveUI : MonoBehaviour
{
    [SerializeField] private TMP_Text objectiveText;
    [SerializeField] private GameObject container;

    private void OnEnable()
    {
        if (ObjectiveTracker.Instance != null)
            ObjectiveTracker.Instance.OnObjectiveChanged += UpdateObjective;
    }

    private void OnDisable()
    {
        if (ObjectiveTracker.Instance != null)
            ObjectiveTracker.Instance.OnObjectiveChanged -= UpdateObjective;
    }

    private void Start()
    {
        if (ObjectiveTracker.Instance != null)
            UpdateObjective(ObjectiveTracker.Instance.CurrentObjective);
    }

    private void UpdateObjective(string objective)
    {
        if (objectiveText == null)
            return;

        objectiveText.text = objective;
        if (container != null)
            container.SetActive(!string.IsNullOrWhiteSpace(objective));
    }
}
```

---

# 12) PauseMenuUI.cs

**Ruta:** `Assets/Scripts/UI/PauseMenuUI.cs`
**Se coloca en:** `Canvas_Gameplay/PausePanel`

```csharp
using UnityEngine;

public class PauseMenuUI : MonoBehaviour
{
    [SerializeField] private GameObject pausePanel;

    private void OnEnable()
    {
        if (GameManager.Instance != null)
        {
            GameManager.Instance.PauseEvent += ShowPause;
            GameManager.Instance.ResumeEvent += HidePause;
        }
    }

    private void OnDisable()
    {
        if (GameManager.Instance != null)
        {
            GameManager.Instance.PauseEvent -= ShowPause;
            GameManager.Instance.ResumeEvent -= HidePause;
        }
    }

    private void Start()
    {
        HidePause();
    }

    public void ShowPause()
    {
        if (pausePanel != null)
            pausePanel.SetActive(true);
    }

    public void HidePause()
    {
        if (pausePanel != null)
            pausePanel.SetActive(false);
    }

    public void OnResumeButton()
    {
        GameManager.Instance?.ResumeGame();
    }

    public void OnRestartButton()
    {
        GameManager.Instance?.RestartScene();
    }

    public void OnSaveButton()
    {
        SaveSystem.Instance?.SaveGame();
    }

    public void OnLoadButton()
    {
        SaveSystem.Instance?.LoadGame();
    }

    public void OnQuitButton()
    {
        GameManager.Instance?.QuitToMenu();
    }
}
```

---

# 13) GameOverUI.cs

**Ruta:** `Assets/Scripts/UI/GameOverUI.cs`
**Se coloca en:** `Canvas_Gameplay/GameOverPanel`

```csharp
using UnityEngine;

public class GameOverUI : MonoBehaviour
{
    [SerializeField] private GameObject gameOverPanel;

    private void OnEnable()
    {
        if (GameManager.Instance != null)
            GameManager.Instance.GameOverEvent += ShowGameOver;
    }

    private void OnDisable()
    {
        if (GameManager.Instance != null)
            GameManager.Instance.GameOverEvent -= ShowGameOver;
    }

    private void Start()
    {
        HideGameOver();
    }

    public void ShowGameOver()
    {
        if (gameOverPanel != null)
            gameOverPanel.SetActive(true);
    }

    public void HideGameOver()
    {
        if (gameOverPanel != null)
            gameOverPanel.SetActive(false);
    }

    public void OnRespawnButton()
    {
        HideGameOver();
        GameManager.Instance?.RespawnFromCheckpoint();
    }

    public void OnRestartButton()
    {
        HideGameOver();
        GameManager.Instance?.RestartScene();
    }

    public void OnQuitButton()
    {
        HideGameOver();
        GameManager.Instance?.QuitToMenu();
    }
}
```

---

# 14) CinematicSignalRelay.cs

**Ruta:** `Assets/Scripts/Story/CinematicSignalRelay.cs`
**Se coloca en:** Timeline / cinemáticas / objetos de señal

```csharp
using UnityEngine;
using UnityEngine.Events;

public class CinematicSignalRelay : MonoBehaviour
{
    [Header("Optional Events")]
    [SerializeField] private UnityEvent onSignal;

    public void InvokeSignal()
    {
        onSignal?.Invoke();
    }

    public void SetStoryFlag(string flag)
    {
        StoryFlags.Instance?.SetFlag(flag);
    }

    public void SetObjective(string objective)
    {
        ObjectiveTracker.Instance?.SetObjective(objective);
    }

    public void ClearObjective()
    {
        ObjectiveTracker.Instance?.ClearObjective();
    }

    public void ShowSubtitle(string subtitle)
    {
        SubtitleUI.Instance?.ShowSubtitle(subtitle, 3f);
    }

    public void LoadScene(string sceneName)
    {
        SceneLoader.Instance?.LoadSceneByName(sceneName);
    }

    public void SaveGame()
    {
        SaveSystem.Instance?.SaveGame();
    }
}
```

---

# 15) AutoObjectiveSetter.cs

**Ruta:** `Assets/Scripts/Story/AutoObjectiveSetter.cs`
**Se coloca en:** triggers o eventos de objetivo

```csharp
using UnityEngine;

public class AutoObjectiveSetter : MonoBehaviour
{
    [TextArea]
    [SerializeField] private string objectiveText;
    [SerializeField] private bool oneUseOnly = true;

    private bool used;

    private void OnTriggerEnter(Collider other)
    {
        if (used && oneUseOnly)
            return;

        if (!other.CompareTag("Player"))
            return;

        ObjectiveTracker.Instance?.SetObjective(objectiveText);

        if (oneUseOnly)
            used = true;
    }

    public void SetObjectiveNow()
    {
        ObjectiveTracker.Instance?.SetObjective(objectiveText);
        used = true;
    }
}
```

---

# 16) IPuzzleRewardReceiver.cs

**Ruta:** `Assets/Scripts/Puzzle/IPuzzleRewardReceiver.cs`
**Se coloca en:** archivo suelto

```csharp
public interface IPuzzleRewardReceiver
{
    void OnPuzzleSolved();
}
```

---

# 17) PuzzleSolvedRelay.cs

**Ruta:** `Assets/Scripts/Puzzle/PuzzleSolvedRelay.cs`
**Se coloca en:** objeto Rewards de cada puzzle

```csharp
using UnityEngine;

public class PuzzleSolvedRelay : MonoBehaviour
{
    [SerializeField] private MonoBehaviour[] rewardReceivers;

    public void TriggerRewards()
    {
        foreach (MonoBehaviour receiver in rewardReceivers)
        {
            if (receiver is IPuzzleRewardReceiver reward)
                reward.OnPuzzleSolved();
        }
    }
}
```

---

# 18) PuzzleReward_GiveKeycard.cs

**Ruta:** `Assets/Scripts/Puzzle/PuzzleReward_GiveKeycard.cs`
**Se coloca en:** Rewards del puzzle

```csharp
using UnityEngine;

public class PuzzleReward_GiveKeycard : MonoBehaviour, IPuzzleRewardReceiver
{
    [SerializeField] private AccessCardLevel cardLevel = AccessCardLevel.Blue;

    public void OnPuzzleSolved()
    {
        InventorySystem.Instance?.AddAccessCard(cardLevel);
    }
}
```

---

# 19) PuzzleReward_SetStoryFlag.cs

**Ruta:** `Assets/Scripts/Puzzle/PuzzleReward_SetStoryFlag.cs`
**Se coloca en:** Rewards del puzzle

```csharp
using UnityEngine;

public class PuzzleReward_SetStoryFlag : MonoBehaviour, IPuzzleRewardReceiver
{
    [SerializeField] private string flagToSet = "PuzzleSolved";

    public void OnPuzzleSolved()
    {
        StoryFlags.Instance?.SetFlag(flagToSet);
    }
}
```

---

# 20) PuzzleReward_OpenAnimator.cs

**Ruta:** `Assets/Scripts/Puzzle/PuzzleReward_OpenAnimator.cs`
**Se coloca en:** Rewards del puzzle

```csharp
using UnityEngine;

public class PuzzleReward_OpenAnimator : MonoBehaviour, IPuzzleRewardReceiver
{
    [SerializeField] private Animator animator;
    [SerializeField] private string triggerName = "Open";

    public void OnPuzzleSolved()
    {
        if (animator != null)
            animator.SetTrigger(triggerName);
    }
}
```

---

# 21) KeycardLevel1Puzzle.cs

**Ruta:** `Assets/Scripts/Puzzle/KeycardLevel1Puzzle.cs`
**Se coloca en:** `PF_Puzzle_Keycard_Lv1`

```csharp
using UnityEngine;
using UnityEngine.Events;

public class KeycardLevel1Puzzle : MonoBehaviour
{
    [SerializeField] private string correctSequence = "3142";
    [SerializeField] private UnityEvent onSolved;
    [SerializeField] private UnityEvent onFailed;

    private string currentSequence = "";
    private bool solved;

    public void PressButton(string value)
    {
        if (solved)
            return;

        currentSequence += value;

        if (currentSequence.Length >= correctSequence.Length)
            Validate();
    }

    public void ResetPuzzle()
    {
        if (solved)
            return;

        currentSequence = "";
    }

    private void Validate()
    {
        if (currentSequence == correctSequence)
        {
            solved = true;
            onSolved?.Invoke();
        }
        else
        {
            onFailed?.Invoke();
            currentSequence = "";
        }
    }
}
```

---

# 22) KeycardLevel2Puzzle.cs

**Ruta:** `Assets/Scripts/Puzzle/KeycardLevel2Puzzle.cs`
**Se coloca en:** `PF_Puzzle_Keycard_Lv2`

```csharp
using UnityEngine;
using UnityEngine.Events;

public class KeycardLevel2Puzzle : MonoBehaviour
{
    [SerializeField] private bool[] correctStates;
    [SerializeField] private UnityEvent onSolved;
    [SerializeField] private UnityEvent onFailed;

    private bool[] currentStates;
    private bool solved;

    private void Awake()
    {
        currentStates = new bool[correctStates.Length];
    }

    public void SetLeverState(int index, bool state)
    {
        if (solved)
            return;

        if (index < 0 || index >= currentStates.Length)
            return;

        currentStates[index] = state;
        CheckSolution();
    }

    private void CheckSolution()
    {
        for (int i = 0; i < correctStates.Length; i++)
        {
            if (currentStates[i] != correctStates[i])
            {
                onFailed?.Invoke();
                return;
            }
        }

        solved = true;
        onSolved?.Invoke();
    }
}
```

---

# 23) KeycardLevel3Puzzle.cs

**Ruta:** `Assets/Scripts/Puzzle/KeycardLevel3Puzzle.cs`
**Se coloca en:** `PF_Puzzle_Keycard_Lv3`

```csharp
using UnityEngine;
using UnityEngine.Events;

public class KeycardLevel3Puzzle : MonoBehaviour
{
    [SerializeField] private int requiredNodeCount = 3;
    [SerializeField] private UnityEvent onSolved;

    private int activatedNodes;
    private bool solved;

    public void ActivateNode()
    {
        if (solved)
            return;

        activatedNodes++;

        if (activatedNodes >= requiredNodeCount)
        {
            solved = true;
            onSolved?.Invoke();
        }
    }

    public void ResetNodes()
    {
        if (solved)
            return;

        activatedNodes = 0;
    }
}
```

---

# 24) HiddenCodePuzzle.cs

**Ruta:** `Assets/Scripts/Puzzle/HiddenCodePuzzle.cs`
**Se coloca en:** `PF_Puzzle_HiddenCode`

```csharp
using UnityEngine;
using UnityEngine.Events;

public class HiddenCodePuzzle : MonoBehaviour
{
    [SerializeField] private string correctCode = "5827";
    [SerializeField] private UnityEvent onSolved;
    [SerializeField] private UnityEvent onFailed;

    private string currentCode = "";
    private bool solved;

    public void InputDigit(string digit)
    {
        if (solved)
            return;

        currentCode += digit;

        if (currentCode.Length >= correctCode.Length)
            Validate();
    }

    public void ClearCode()
    {
        if (solved)
            return;

        currentCode = "";
    }

    private void Validate()
    {
        if (currentCode == correctCode)
        {
            solved = true;
            onSolved?.Invoke();
        }
        else
        {
            currentCode = "";
            onFailed?.Invoke();
        }
    }
}
```

---

# 25) FuseBox.cs

**Ruta:** `Assets/Scripts/Puzzle/FuseBox.cs`
**Se coloca en:** `PF_FuseBox`

```csharp
using UnityEngine;
using UnityEngine.Events;

public class FuseBox : MonoBehaviour, IInteractable
{
    [SerializeField] private string requiredFuseItemId = "Fuse";
    [SerializeField] private int requiredFuseCount = 3;
    [SerializeField] private string prompt = "Press E to insert fuse";
    [SerializeField] private string completedPrompt = "Fuse box complete";
    [SerializeField] private string missingPrompt = "You need more fuses";
    [SerializeField] private UnityEvent onAllFusesInserted;

    public int CurrentInserted { get; private set; }
    public bool HasAllFusesInserted => CurrentInserted >= requiredFuseCount;

    public string GetPrompt()
    {
        if (HasAllFusesInserted)
            return completedPrompt;

        if (InventorySystem.Instance != null && InventorySystem.Instance.HasItem(requiredFuseItemId))
            return $"{prompt} ({CurrentInserted}/{requiredFuseCount})";

        return missingPrompt;
    }

    public bool CanInteract(InteractionSystem interactor)
    {
        if (HasAllFusesInserted)
            return false;

        return InventorySystem.Instance != null &&
               InventorySystem.Instance.HasItem(requiredFuseItemId, 1);
    }

    public void Interact(InteractionSystem interactor)
    {
        if (HasAllFusesInserted)
            return;

        if (InventorySystem.Instance == null)
            return;

        if (InventorySystem.Instance.RemoveItem(requiredFuseItemId, 1))
        {
            CurrentInserted++;

            if (HasAllFusesInserted)
            {
                StoryFlags.Instance?.SetFlag("FuseBoxReady");
                onAllFusesInserted?.Invoke();
            }
        }
    }
}
```

---

# 26) GeneratorSwitch.cs

**Ruta:** `Assets/Scripts/Puzzle/GeneratorSwitch.cs`
**Se coloca en:** `PF_GeneratorSwitch`

```csharp
using UnityEngine;
using UnityEngine.Events;

public class GeneratorSwitch : MonoBehaviour, IInteractable
{
    [SerializeField] private FuseBox fuseBox;
    [SerializeField] private string prompt = "Press E to start generator";
    [SerializeField] private string missingFusesPrompt = "The generator needs fuses";
    [SerializeField] private UnityEvent onGeneratorStarted;

    private bool activated;

    public string GetPrompt()
    {
        if (activated)
            return string.Empty;

        if (fuseBox != null && !fuseBox.HasAllFusesInserted)
            return missingFusesPrompt;

        return prompt;
    }

    public bool CanInteract(InteractionSystem interactor)
    {
        if (activated)
            return false;

        if (fuseBox != null && !fuseBox.HasAllFusesInserted)
            return false;

        return true;
    }

    public void Interact(InteractionSystem interactor)
    {
        if (activated)
            return;

        activated = true;
        StoryFlags.Instance?.SetFlag("GeneratorPowered");
        onGeneratorStarted?.Invoke();
    }
}
```

---

# 27) FinalChoiceManager.cs

**Ruta:** `Assets/Scripts/Story/FinalChoiceManager.cs`
**Se coloca en:** escena final

```csharp
using UnityEngine;
using UnityEngine.Events;

public class FinalChoiceManager : MonoBehaviour
{
    [SerializeField] private UnityEvent onEscapeChosen;
    [SerializeField] private UnityEvent onFightChosen;

    private bool choiceMade;

    public void ChooseEscape()
    {
        if (choiceMade)
            return;

        choiceMade = true;
        StoryFlags.Instance?.SetFlag("Ending_EscapeChosen");
        onEscapeChosen?.Invoke();
    }

    public void ChooseFight()
    {
        if (choiceMade)
            return;

        choiceMade = true;
        StoryFlags.Instance?.SetFlag("Ending_FightChosen");
        onFightChosen?.Invoke();
    }
}
```

---

# 28) StealthKillUnlock.cs

**Ruta:** `Assets/Scripts/Story/StealthKillUnlock.cs`
**Se coloca en:** evento/cinemática del nivel 3

```csharp
using UnityEngine;
using UnityEngine.Events;

public class StealthKillUnlock : MonoBehaviour
{
    [SerializeField] private string unlockFlag = "StealthKillUnlocked";
    [SerializeField] private UnityEvent onUnlocked;
    [SerializeField] private bool oneUseOnly = true;

    private bool used;

    public void Unlock()
    {
        if (used && oneUseOnly)
            return;

        StoryFlags.Instance?.SetFlag(unlockFlag);
        onUnlocked?.Invoke();
        used = true;
    }
}
```

---

# 29) MusicManager.cs

**Ruta:** `Assets/Scripts/Audio/MusicManager.cs`
**Se coloca en:** `Bootstrap_Managers/AudioManagers/MusicManager`

```csharp
using UnityEngine;

public class MusicManager : MonoBehaviour
{
    public enum MusicState
    {
        Calm,
        Suspicious,
        Chase
    }

    public static MusicManager Instance { get; private set; }

    [Header("Sources")]
    [SerializeField] private AudioSource calmSource;
    [SerializeField] private AudioSource suspiciousSource;
    [SerializeField] private AudioSource chaseSource;

    [Header("Mix")]
    [SerializeField] private float fadeSpeed = 2f;
    [SerializeField] private float maxVolume = 1f;

    public MusicState CurrentState { get; private set; } = MusicState.Calm;

    private int alertedEnemies;
    private bool bossActive;
    private bool forcedChase;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
        DontDestroyOnLoad(gameObject);

        PrepareSource(calmSource);
        PrepareSource(suspiciousSource);
        PrepareSource(chaseSource);
    }

    private void Update()
    {
        ResolveState();
        UpdateVolumes();
    }

    private void PrepareSource(AudioSource source)
    {
        if (source == null) return;

        source.loop = true;
        if (!source.isPlaying)
            source.Play();
    }

    public void RegisterEnemyAlert(bool isAlerted)
    {
        alertedEnemies += isAlerted ? 1 : -1;
        alertedEnemies = Mathf.Max(0, alertedEnemies);
    }

    public void SetBossActive(bool active)
    {
        bossActive = active;
    }

    public void ForceChase(bool active)
    {
        forcedChase = active;
    }

    public void SetState(MusicState state)
    {
        CurrentState = state;
    }

    private void ResolveState()
    {
        if (forcedChase || bossActive)
        {
            CurrentState = MusicState.Chase;
        }
        else if (alertedEnemies > 0)
        {
            CurrentState = MusicState.Suspicious;
        }
        else
        {
            CurrentState = MusicState.Calm;
        }
    }

    private void UpdateVolumes()
    {
        SetTargetVolume(calmSource, CurrentState == MusicState.Calm ? maxVolume : 0f);
        SetTargetVolume(suspiciousSource, CurrentState == MusicState.Suspicious ? maxVolume : 0f);
        SetTargetVolume(chaseSource, CurrentState == MusicState.Chase ? maxVolume : 0f);
    }

    private void SetTargetVolume(AudioSource source, float target)
    {
        if (source == null) return;
        source.volume = Mathf.MoveTowards(source.volume, target, fadeSpeed * Time.deltaTime);
    }
}
```

---

# 30) AmbientAudioManager.cs

**Ruta:** `Assets/Scripts/Audio/AmbientAudioManager.cs`
**Se coloca en:** `Bootstrap_Managers/AudioManagers/AmbientAudioManager`

```csharp
using UnityEngine;

public class AmbientAudioManager : MonoBehaviour
{
    public static AmbientAudioManager Instance { get; private set; }

    [SerializeField] private AudioSource calmAmbient;
    [SerializeField] private AudioSource powerOnAmbient;
    [SerializeField] private float fadeSpeed = 1.5f;

    private bool generatorPowered;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
        DontDestroyOnLoad(gameObject);

        Prepare(calmAmbient);
        Prepare(powerOnAmbient);
    }

    private void Update()
    {
        float calmTarget = generatorPowered ? 0.2f : 1f;
        float powerTarget = generatorPowered ? 1f : 0f;

        if (calmAmbient != null)
            calmAmbient.volume = Mathf.MoveTowards(calmAmbient.volume, calmTarget, fadeSpeed * Time.deltaTime);

        if (powerOnAmbient != null)
            powerOnAmbient.volume = Mathf.MoveTowards(powerOnAmbient.volume, powerTarget, fadeSpeed * Time.deltaTime);
    }

    private void Prepare(AudioSource source)
    {
        if (source == null)
            return;

        source.loop = true;
        if (!source.isPlaying)
            source.Play();
    }

    public void SetGeneratorPowered(bool powered)
    {
        generatorPowered = powered;
    }
}
```

---

# 31) FootstepNoiseEmitter.cs

**Ruta:** `Assets/Scripts/Audio/FootstepNoiseEmitter.cs`
**Se coloca en:** `PF_Player`

```csharp
using UnityEngine;

[RequireComponent(typeof(PlayerMovement))]
public class FootstepNoiseEmitter : MonoBehaviour
{
    [SerializeField] private float walkInterval = 0.55f;
    [SerializeField] private float sprintInterval = 0.35f;
    [SerializeField] private float crouchInterval = 0.8f;

    [SerializeField] private float walkNoise = 0.45f;
    [SerializeField] private float sprintNoise = 1f;
    [SerializeField] private float crouchNoise = 0.2f;

    private PlayerMovement movement;
    private float timer;

    private void Awake()
    {
        movement = GetComponent<PlayerMovement>();
    }

    private void Update()
    {
        if (!movement.IsGrounded)
            return;

        if (movement.HorizontalVelocity.magnitude < 0.1f)
        {
            timer = 0f;
            return;
        }

        float interval;
        float intensity;
        NoisePriority priority = NoisePriority.Low;

        switch (movement.CurrentState)
        {
            case PlayerMovement.MoveState.Sprinting:
                interval = sprintInterval;
                intensity = sprintNoise;
                priority = NoisePriority.Medium;
                break;

            case PlayerMovement.MoveState.Crouching:
                interval = crouchInterval;
                intensity = crouchNoise;
                priority = NoisePriority.Low;
                break;

            default:
                interval = walkInterval;
                intensity = walkNoise;
                priority = NoisePriority.Low;
                break;
        }

        timer += Time.deltaTime;

        if (timer >= interval)
        {
            timer = 0f;
            NoiseSystem.Emit(
                transform.position,
                intensity,
                NoiseSourceType.Footstep,
                priority,
                transform
            );
        }
    }
}
```

---

# 32) FootstepSurfaceAudio.cs

**Ruta:** `Assets/Scripts/Audio/FootstepSurfaceAudio.cs`
**Se coloca en:** `PF_Player`

```csharp
using UnityEngine;

[RequireComponent(typeof(PlayerMovement))]
public class FootstepSurfaceAudio : MonoBehaviour
{
    [System.Serializable]
    public class SurfaceAudioSet
    {
        public string surfaceTag;
        public AudioClip[] clips;
    }

    [SerializeField] private AudioSource audioSource;
    [SerializeField] private LayerMask groundMask;
    [SerializeField] private float rayDistance = 1.5f;
    [SerializeField] private SurfaceAudioSet[] surfaces;

    private PlayerMovement movement;
    private float timer;

    [SerializeField] private float walkInterval = 0.55f;
    [SerializeField] private float sprintInterval = 0.35f;
    [SerializeField] private float crouchInterval = 0.8f;

    private void Awake()
    {
        movement = GetComponent<PlayerMovement>();
    }

    private void Update()
    {
        if (movement == null || !movement.IsGrounded)
            return;

        if (movement.HorizontalVelocity.magnitude < 0.1f)
        {
            timer = 0f;
            return;
        }

        float interval = walkInterval;

        switch (movement.CurrentState)
        {
            case PlayerMovement.MoveState.Sprinting:
                interval = sprintInterval;
                break;
            case PlayerMovement.MoveState.Crouching:
                interval = crouchInterval;
                break;
        }

        timer += Time.deltaTime;

        if (timer >= interval)
        {
            timer = 0f;
            PlayFootstep();
        }
    }

    private void PlayFootstep()
    {
        if (audioSource == null)
            return;

        if (Physics.Raycast(transform.position + Vector3.up * 0.2f, Vector3.down, out RaycastHit hit, rayDistance, groundMask))
        {
            string tag = hit.collider.tag;

            foreach (var surface in surfaces)
            {
                if (surface.surfaceTag == tag && surface.clips != null && surface.clips.Length > 0)
                {
                    AudioClip clip = surface.clips[Random.Range(0, surface.clips.Length)];
                    audioSource.PlayOneShot(clip);
                    return;
                }
            }
        }
    }
}
```

---

# 33) DebugGameplaySettings.cs

**Ruta:** `Assets/Scripts/Debug/DebugGameplaySettings.cs`
**Se coloca en:** `GameplayDebug`

```csharp
using UnityEngine;

public class DebugGameplaySettings : MonoBehaviour
{
    public static DebugGameplaySettings Instance { get; private set; }

    [Header("General")]
    [SerializeField] private bool debugEnabled = true;

    [Header("AI")]
    [SerializeField] private bool showEnemyVision = true;
    [SerializeField] private bool showEnemyPaths = true;
    [SerializeField] private bool showSearchZones = true;
    [SerializeField] private bool showHideSpotTargets = true;

    [Header("Noise")]
    [SerializeField] private bool showNoiseEvents = true;
    [SerializeField] private float noiseMarkerLifetime = 2.5f;

    [Header("Boss")]
    [SerializeField] private bool showBossSpawnPoints = true;
    [SerializeField] private bool allowBossHotkeys = true;

    [Header("UI")]
    [SerializeField] private bool showTensionUI = true;
    [SerializeField] private bool showEnemyStateUI = true;

    public bool DebugEnabled => debugEnabled;
    public bool ShowEnemyVision => debugEnabled && showEnemyVision;
    public bool ShowEnemyPaths => debugEnabled && showEnemyPaths;
    public bool ShowSearchZones => debugEnabled && showSearchZones;
    public bool ShowHideSpotTargets => debugEnabled && showHideSpotTargets;
    public bool ShowNoiseEvents => debugEnabled && showNoiseEvents;
    public float NoiseMarkerLifetime => noiseMarkerLifetime;
    public bool ShowBossSpawnPoints => debugEnabled && showBossSpawnPoints;
    public bool AllowBossHotkeys => debugEnabled && allowBossHotkeys;
    public bool ShowTensionUI => debugEnabled && showTensionUI;
    public bool ShowEnemyStateUI => debugEnabled && showEnemyStateUI;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
        DontDestroyOnLoad(gameObject);
    }
}
```

---

# 34) NoiseDebugMonitor.cs

**Ruta:** `Assets/Scripts/Debug/NoiseDebugMonitor.cs`
**Se coloca en:** `GameplayDebug`

```csharp
using System.Collections.Generic;
using UnityEngine;

public class NoiseDebugMonitor : MonoBehaviour
{
    private struct NoiseMarker
    {
        public Vector3 Position;
        public float TimeCreated;
        public float Intensity;
        public NoisePriority Priority;
        public NoiseSourceType SourceType;
    }

    private readonly List<NoiseMarker> markers = new();

    private void OnEnable()
    {
        NoiseSystem.OnNoise += OnNoise;
    }

    private void OnDisable()
    {
        NoiseSystem.OnNoise -= OnNoise;
    }

    private void Update()
    {
        float lifetime = DebugGameplaySettings.Instance != null
            ? DebugGameplaySettings.Instance.NoiseMarkerLifetime
            : 2.5f;

        float now = Time.time;
        markers.RemoveAll(m => now - m.TimeCreated > lifetime);
    }

    private void OnNoise(NoiseEventData data)
    {
        if (DebugGameplaySettings.Instance != null && !DebugGameplaySettings.Instance.ShowNoiseEvents)
            return;

        markers.Add(new NoiseMarker
        {
            Position = data.Position,
            TimeCreated = Time.time,
            Intensity = data.Intensity,
            Priority = data.Priority,
            SourceType = data.SourceType
        });
    }

    private void OnDrawGizmos()
    {
        if (Application.isPlaying &&
            DebugGameplaySettings.Instance != null &&
            !DebugGameplaySettings.Instance.ShowNoiseEvents)
            return;

        foreach (var marker in markers)
        {
            Gizmos.color = GetPriorityColor(marker.Priority);
            float radius = Mathf.Lerp(0.25f, 1.25f, Mathf.Clamp01(marker.Intensity));
            Gizmos.DrawWireSphere(marker.Position, radius);
            Gizmos.DrawSphere(marker.Position, 0.08f);
        }
    }

    private Color GetPriorityColor(NoisePriority priority)
    {
        return priority switch
        {
            NoisePriority.Low => Color.green,
            NoisePriority.Medium => Color.yellow,
            NoisePriority.High => new Color(1f, 0.5f, 0f),
            NoisePriority.Critical => Color.red,
            _ => Color.white
        };
    }
}
```

---

# 35) AIDebugGizmos.cs

**Ruta:** `Assets/Scripts/Debug/AIDebugGizmos.cs`
**Se coloca en:** `PF_Enemy_Base`

```csharp
using UnityEngine;
using UnityEngine.AI;

[RequireComponent(typeof(EnemyAI))]
public class AIDebugGizmos : MonoBehaviour
{
    [SerializeField] private EnemyAI enemyAI;
    [SerializeField] private NavMeshAgent agent;
    [SerializeField] private Transform eyePoint;

    [Header("Vision")]
    [SerializeField] private float fallbackViewDistance = 12f;
    [SerializeField] private float fallbackViewAngle = 90f;

    private void Reset()
    {
        enemyAI = GetComponent<EnemyAI>();
        agent = GetComponent<NavMeshAgent>();
    }

    private void OnDrawGizmos()
    {
        if (!Application.isPlaying)
            return;

        if (DebugGameplaySettings.Instance != null && !DebugGameplaySettings.Instance.ShowEnemyVision)
            return;

        if (enemyAI == null)
            enemyAI = GetComponent<EnemyAI>();

        if (agent == null)
            agent = GetComponent<NavMeshAgent>();

        Vector3 origin = eyePoint != null ? eyePoint.position : transform.position + Vector3.up * 1.6f;

        DrawVisionCone(origin);
        DrawPath();
        DrawCurrentTargets();
    }

    private void DrawVisionCone(Vector3 origin)
    {
        float distance = fallbackViewDistance;
        float halfAngle = fallbackViewAngle * 0.5f;

        Vector3 left = Quaternion.Euler(0f, -halfAngle, 0f) * transform.forward;
        Vector3 right = Quaternion.Euler(0f, halfAngle, 0f) * transform.forward;

        Gizmos.color = Color.cyan;
        Gizmos.DrawLine(origin, origin + left * distance);
        Gizmos.DrawLine(origin, origin + right * distance);
        Gizmos.DrawWireSphere(origin + transform.forward * distance * 0.5f, 0.05f);
    }

    private void DrawPath()
    {
        if (agent == null)
            return;

        if (DebugGameplaySettings.Instance != null && !DebugGameplaySettings.Instance.ShowEnemyPaths)
            return;

        if (!agent.hasPath)
            return;

        Gizmos.color = Color.blue;
        Vector3 prev = transform.position;

        foreach (Vector3 corner in agent.path.corners)
        {
            Gizmos.DrawLine(prev, corner);
            Gizmos.DrawSphere(corner, 0.08f);
            prev = corner;
        }
    }

    private void DrawCurrentTargets()
    {
        if (enemyAI == null)
            return;

        if (enemyAI.TryGetDebugTargets(out Vector3 investigatePos, out Vector3 lastKnownPos, out Vector3 hideSpotPos, out Vector3 searchZonePos))
        {
            Gizmos.color = Color.yellow;
            Gizmos.DrawSphere(investigatePos, 0.12f);

            Gizmos.color = Color.red;
            Gizmos.DrawWireSphere(lastKnownPos, 0.2f);

            if (hideSpotPos != Vector3.zero)
            {
                Gizmos.color = Color.magenta;
                Gizmos.DrawCube(hideSpotPos, Vector3.one * 0.2f);
            }

            if (searchZonePos != Vector3.zero)
            {
                Gizmos.color = Color.green;
                Gizmos.DrawWireCube(searchZonePos, Vector3.one * 0.5f);
            }
        }
    }
}
```

---

# 36) SearchZoneDebugGizmos.cs

**Ruta:** `Assets/Scripts/Debug/SearchZoneDebugGizmos.cs`
**Se coloca en:** cada `SearchZone`

```csharp
using UnityEngine;

[RequireComponent(typeof(SearchZone))]
public class SearchZoneDebugGizmos : MonoBehaviour
{
    [SerializeField] private SearchZone searchZone;

    private void Reset()
    {
        searchZone = GetComponent<SearchZone>();
    }

    private void OnDrawGizmos()
    {
        if (!Application.isPlaying && searchZone == null)
            searchZone = GetComponent<SearchZone>();

        if (DebugGameplaySettings.Instance != null && !DebugGameplaySettings.Instance.ShowSearchZones)
            return;

        if (searchZone == null)
            return;

        Gizmos.color = Color.green;
        Gizmos.DrawWireCube(transform.position, new Vector3(1.5f, 0.2f, 1.5f));

        Transform[] points = searchZone.SearchPoints;
        if (points == null)
            return;

        foreach (Transform point in points)
        {
            if (point == null)
                continue;

            Gizmos.DrawSphere(point.position, 0.12f);
            Gizmos.DrawLine(transform.position, point.position);
        }
    }
}
```

---

# 37) BossSpawnPointGizmo.cs

**Ruta:** `Assets/Scripts/Debug/BossSpawnPointGizmo.cs`
**Se coloca en:** `PF_BossSpawnPoint`

```csharp
using UnityEngine;

[RequireComponent(typeof(BossSpawnPoint))]
public class BossSpawnPointGizmo : MonoBehaviour
{
    private void OnDrawGizmos()
    {
        if (DebugGameplaySettings.Instance != null && !DebugGameplaySettings.Instance.ShowBossSpawnPoints)
            return;

        Gizmos.color = Color.red;
        Gizmos.DrawWireSphere(transform.position, 0.4f);
        Gizmos.DrawLine(transform.position, transform.position + transform.forward * 1.5f);
    }
}
```

---

# 38) BossDebugPanel.cs

**Ruta:** `Assets/Scripts/Debug/BossDebugPanel.cs`
**Se coloca en:** Canvas debug o GameplayDebug

```csharp
using UnityEngine;

public class BossDebugPanel : MonoBehaviour
{
    [SerializeField] private BossController bossController;
    [SerializeField] private Transform runToPointTarget;

    public void SpawnTimedChase()
    {
        if (bossController == null) return;

        if (BossSpawnManager.Instance != null &&
            BossSpawnManager.Instance.TryGetSpawnPoint(out BossSpawnPoint point))
        {
            bossController.transform.position = point.Position;
            bossController.transform.rotation = point.Rotation;
        }

        bossController.BeginTimedChase();
    }

    public void SpawnRunToPoint()
    {
        if (bossController == null || runToPointTarget == null) return;

        if (BossSpawnManager.Instance != null &&
            BossSpawnManager.Instance.TryGetSpawnPoint(out BossSpawnPoint point))
        {
            bossController.transform.position = point.Position;
            bossController.transform.rotation = point.Rotation;
        }

        bossController.BeginChaseToPoint(runToPointTarget);
    }

    public void StopBoss()
    {
        bossController?.StopBoss();
    }

    public void AddTension(float amount)
    {
        TensionDirector.Instance?.AddTension(amount);
    }

    public void ResetTension()
    {
        TensionDirector.Instance?.ResetTension();
    }
}
```

---

# 39) TensionDebugUI.cs

**Ruta:** `Assets/Scripts/Debug/TensionDebugUI.cs`
**Se coloca en:** `Canvas_Debug/TensionPanel`

```csharp
using TMPro;
using UnityEngine;
using UnityEngine.UI;

public class TensionDebugUI : MonoBehaviour
{
    [SerializeField] private GameObject container;
    [SerializeField] private TMP_Text valueText;
    [SerializeField] private TMP_Text stateText;
    [SerializeField] private Slider tensionSlider;

    private void Update()
    {
        bool visible = DebugGameplaySettings.Instance == null || DebugGameplaySettings.Instance.ShowTensionUI;
        if (container != null)
            container.SetActive(visible);

        if (!visible || TensionDirector.Instance == null)
            return;

        float value = TensionDirector.Instance.CurrentTension;

        if (valueText != null)
            valueText.text = $"Tension: {value:0.0}";

        if (tensionSlider != null)
            tensionSlider.value = value / 100f;

        if (stateText != null)
            stateText.text = TensionDirector.Instance.CanSpawnBoss ? "Boss Eligible" : "Cooling / Blocked";
    }
}
```

---

# 40) EnemyStateDebugUI.cs

**Ruta:** `Assets/Scripts/Debug/EnemyStateDebugUI.cs`
**Se coloca en:** `Canvas_Debug/EnemyInfoPanel`

```csharp
using TMPro;
using UnityEngine;

public class EnemyStateDebugUI : MonoBehaviour
{
    [SerializeField] private Camera playerCamera;
    [SerializeField] private float inspectDistance = 20f;
    [SerializeField] private LayerMask enemyMask;
    [SerializeField] private GameObject container;
    [SerializeField] private TMP_Text infoText;

    private void Update()
    {
        bool visible = DebugGameplaySettings.Instance == null || DebugGameplaySettings.Instance.ShowEnemyStateUI;
        if (container != null)
            container.SetActive(visible);

        if (!visible || playerCamera == null || infoText == null)
            return;

        Ray ray = new Ray(playerCamera.transform.position, playerCamera.transform.forward);

        if (Physics.Raycast(ray, out RaycastHit hit, inspectDistance, enemyMask, QueryTriggerInteraction.Collide))
        {
            EnemyAI enemy = hit.collider.GetComponentInParent<EnemyAI>();
            if (enemy != null)
            {
                infoText.text =
                    $"Enemy: {enemy.name}\n" +
                    $"State: {enemy.DebugCurrentState}\n" +
                    $"Role: {enemy.GetTacticalRole()}";
                return;
            }
        }

        infoText.text = "Enemy: none";
    }
}
```

---

# 41) GameplayDebugHotkeys.cs

**Ruta:** `Assets/Scripts/Debug/GameplayDebugHotkeys.cs`
**Se coloca en:** `GameplayDebug`

```csharp
using UnityEngine;

public class GameplayDebugHotkeys : MonoBehaviour
{
    [SerializeField] private BossDebugPanel bossDebugPanel;
    [SerializeField] private Transform fakeNoiseOrigin;

    private void Update()
    {
        if (DebugGameplaySettings.Instance != null && !DebugGameplaySettings.Instance.DebugEnabled)
            return;

        if (Input.GetKeyDown(KeyCode.F5))
            SaveSystem.Instance?.SaveGame();

        if (Input.GetKeyDown(KeyCode.F9))
            SaveSystem.Instance?.LoadGame();

        if (Input.GetKeyDown(KeyCode.F6))
            TensionDirector.Instance?.AddTension(20f);

        if (Input.GetKeyDown(KeyCode.F7))
            TensionDirector.Instance?.ResetTension();

        if (Input.GetKeyDown(KeyCode.F8) && bossDebugPanel != null)
            bossDebugPanel.SpawnTimedChase();

        if (Input.GetKeyDown(KeyCode.F10))
        {
            Vector3 pos = fakeNoiseOrigin != null ? fakeNoiseOrigin.position : transform.position + transform.forward * 3f;

            NoiseSystem.Emit(
                pos,
                1f,
                NoiseSourceType.ScriptedEvent,
                NoisePriority.High,
                transform
            );
        }
    }
}
```

---

# 42) Dónde poner este bloque

## En Bootstrap / managers

Crea o completa:

```text
Bootstrap_Managers
├── AudioManagers
│   ├── MusicManager
│   └── AmbientAudioManager
├── AIDirectors
│   ├── TensionDirector
│   ├── EnemySearchCoordinator
│   └── EnemySearchReservationSystem
└── DebugRoot
    ├── DebugGameplaySettings
    ├── NoiseDebugMonitor
    └── GameplayDebugHotkeys
```

## En Canvas_Gameplay

Añade:

* `TutorialUI`
* `SubtitleUI`
* `ObjectiveUI`
* `PauseMenuUI`
* `GameOverUI`

## En Canvas_Debug

Añade:

* `TensionDebugUI`
* `EnemyStateDebugUI`
* opcional `BossDebugPanel`

## En puzzles

Añade:

* `PuzzleSolvedRelay`
* rewards según necesites

## En escenas

Añade:

* `TensionProfileApplier`
* `SearchZone`
* `SearchZoneDebugGizmos`

---

# 43) Qué queda ya cubierto

Con estos 6 bloques ya tienes:

* core completo
* player completo
* interacción y pickups
* enemigos y ruido
* boss y encuentros
* puzzles e historia
* audio
* UI
* debug

O sea, ya tienes la base estructural del juego casi entera montada en scripts.

---

# 44) Lo que falta realmente

A partir de aquí ya no faltan “bloques grandes” esenciales. Lo que queda suele ser de este tipo:

* scripts pequeños de botones o palancas concretas
* setup de Timeline / animaciones
* prefabs específicos del arte final
* balance de valores
* conexiones de escena
* polish visual y sonoro

---

# 45) Recomendación ahora mismo

Lo más útil ahora es hacer una de estas dos cosas:

**Opción 1:** te preparo el **checklist técnico final por escena**, muy práctico, para montar cada nivel sin olvidarte de nada.

**Opción 2:** te hago un **índice maestro de todos los scripts** con:

* nombre
* ruta
* prefab/escena donde va
* dependencias
* ya ordenado como documento final

Como ya tienes bastante código, yo haría **primero el checklist por escena**, porque te va a servir inmediatamente para montar el proyecto en Unity sin volverte loco.
