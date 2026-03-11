Perfecto. Empezamos con el **Bloque 1 — Core**.
Este bloque es la base del proyecto: guardado, escenas, flags, objetivos y checkpoints.

Te lo dejo ya en formato de **copiar y pegar**.

---

# 1) GameManager.cs

**Ruta:** `Assets/Scripts/Core/GameManager.cs`
**Se coloca en:** `Bootstrap_Managers/CoreManagers/GameManager`

```csharp
using System;
using UnityEngine;
using UnityEngine.Events;

public class GameManager : MonoBehaviour
{
    public static GameManager Instance { get; private set; }

    [Header("State")]
    public bool IsGameOver { get; private set; }
    public bool IsPaused { get; private set; }

    [Header("UI Events")]
    [SerializeField] private UnityEvent onGameOver;
    [SerializeField] private UnityEvent onPause;
    [SerializeField] private UnityEvent onResume;

    [Header("Settings")]
    [SerializeField] private KeyCode pauseKey = KeyCode.Escape;

    public event Action GameOverEvent;
    public event Action PauseEvent;
    public event Action ResumeEvent;

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
        if (IsGameOver)
            return;

        if (Input.GetKeyDown(pauseKey))
        {
            if (IsPaused) ResumeGame();
            else PauseGame();
        }
    }

    public void GameOverCaught()
    {
        if (IsGameOver)
            return;

        IsGameOver = true;
        Time.timeScale = 0f;

        onGameOver?.Invoke();
        GameOverEvent?.Invoke();

        Cursor.lockState = CursorLockMode.None;
        Cursor.visible = true;
    }

    public void RespawnFromCheckpoint()
    {
        Time.timeScale = 1f;
        IsGameOver = false;
        IsPaused = false;

        if (CheckpointSystem.Instance != null && CheckpointSystem.Instance.HasCheckpoint)
            CheckpointSystem.Instance.LoadLastCheckpoint();
        else
            SceneLoader.Instance?.ReloadCurrentScene();

        Cursor.lockState = CursorLockMode.Locked;
        Cursor.visible = false;
    }

    public void RestartScene()
    {
        Time.timeScale = 1f;
        IsGameOver = false;
        IsPaused = false;

        SceneLoader.Instance?.ReloadCurrentScene();

        Cursor.lockState = CursorLockMode.Locked;
        Cursor.visible = false;
    }

    public void PauseGame()
    {
        if (IsGameOver || IsPaused)
            return;

        IsPaused = true;
        Time.timeScale = 0f;

        onPause?.Invoke();
        PauseEvent?.Invoke();

        Cursor.lockState = CursorLockMode.None;
        Cursor.visible = true;
    }

    public void ResumeGame()
    {
        if (!IsPaused)
            return;

        IsPaused = false;
        Time.timeScale = 1f;

        onResume?.Invoke();
        ResumeEvent?.Invoke();

        Cursor.lockState = CursorLockMode.Locked;
        Cursor.visible = false;
    }

    public void QuitToMenu(string menuSceneName = "MainMenu")
    {
        Time.timeScale = 1f;
        IsGameOver = false;
        IsPaused = false;

        SceneLoader.Instance?.LoadSceneByName(menuSceneName);
    }

    public void VictoryLoadScene(string sceneName)
    {
        Time.timeScale = 1f;
        IsGameOver = false;
        IsPaused = false;

        SceneLoader.Instance?.LoadSceneByName(sceneName);
    }
}
```

---

# 2) StoryFlags.cs

**Ruta:** `Assets/Scripts/Core/StoryFlags.cs`
**Se coloca en:** `Bootstrap_Managers/CoreManagers/StoryFlags`

```csharp
using System;
using System.Collections.Generic;
using UnityEngine;

public class StoryFlags : MonoBehaviour
{
    public static StoryFlags Instance { get; private set; }

    public event Action<string> OnFlagSet;

    private readonly HashSet<string> flags = new();

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

    public void SetFlag(string flag)
    {
        if (string.IsNullOrWhiteSpace(flag))
            return;

        if (flags.Add(flag))
            OnFlagSet?.Invoke(flag);
    }

    public bool HasFlag(string flag)
    {
        if (string.IsNullOrWhiteSpace(flag))
            return false;

        return flags.Contains(flag);
    }

    public void ClearAll()
    {
        flags.Clear();
    }

    public List<string> GetAllFlags()
    {
        return new List<string>(flags);
    }

    public void RestoreFlags(List<string> restoredFlags)
    {
        flags.Clear();

        if (restoredFlags == null)
            return;

        foreach (string flag in restoredFlags)
        {
            if (!string.IsNullOrWhiteSpace(flag))
                flags.Add(flag);
        }
    }
}
```

---

# 3) ObjectiveTracker.cs

**Ruta:** `Assets/Scripts/Core/ObjectiveTracker.cs`
**Se coloca en:** `Bootstrap_Managers/CoreManagers/ObjectiveTracker`

```csharp
using System;
using UnityEngine;

public class ObjectiveTracker : MonoBehaviour
{
    public static ObjectiveTracker Instance { get; private set; }

    public event Action<string> OnObjectiveChanged;

    public string CurrentObjective { get; private set; }

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

    public void SetObjective(string objective)
    {
        CurrentObjective = objective;
        OnObjectiveChanged?.Invoke(CurrentObjective);
    }

    public void ClearObjective()
    {
        CurrentObjective = string.Empty;
        OnObjectiveChanged?.Invoke(CurrentObjective);
    }
}
```

---

# 4) SceneLoader.cs

**Ruta:** `Assets/Scripts/Core/SceneLoader.cs`
**Se coloca en:** `Bootstrap_Managers/CoreManagers/SceneLoader`

```csharp
using System.Collections;
using UnityEngine;
using UnityEngine.SceneManagement;

public class SceneLoader : MonoBehaviour
{
    public static SceneLoader Instance { get; private set; }

    [SerializeField] private CanvasGroup loadingCanvasGroup;
    [SerializeField] private float fadeDuration = 0.4f;

    private bool isLoading;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        Instance = this;
        DontDestroyOnLoad(gameObject);

        if (loadingCanvasGroup != null)
        {
            loadingCanvasGroup.alpha = 0f;
            loadingCanvasGroup.gameObject.SetActive(false);
        }
    }

    public void LoadSceneByName(string sceneName)
    {
        if (!isLoading && !string.IsNullOrWhiteSpace(sceneName))
            StartCoroutine(LoadRoutine(sceneName));
    }

    public void ReloadCurrentScene()
    {
        if (!isLoading)
            StartCoroutine(LoadRoutine(SceneManager.GetActiveScene().name));
    }

    private IEnumerator LoadRoutine(string sceneName)
    {
        isLoading = true;

        if (loadingCanvasGroup != null)
            yield return FadeLoading(true);

        AsyncOperation op = SceneManager.LoadSceneAsync(sceneName);

        while (!op.isDone)
            yield return null;

        if (loadingCanvasGroup != null)
            yield return FadeLoading(false);

        isLoading = false;
    }

    private IEnumerator FadeLoading(bool show)
    {
        loadingCanvasGroup.gameObject.SetActive(true);

        float start = loadingCanvasGroup.alpha;
        float target = show ? 1f : 0f;
        float t = 0f;

        while (t < fadeDuration)
        {
            t += Time.unscaledDeltaTime;
            loadingCanvasGroup.alpha = Mathf.Lerp(start, target, t / fadeDuration);
            yield return null;
        }

        loadingCanvasGroup.alpha = target;

        if (!show)
            loadingCanvasGroup.gameObject.SetActive(false);
    }
}
```

---

# 5) SaveSystem.cs

**Ruta:** `Assets/Scripts/Core/SaveSystem.cs`
**Se coloca en:** `Bootstrap_Managers/CoreManagers/SaveSystem`

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using UnityEngine;
using UnityEngine.SceneManagement;

[Serializable]
public struct SerializableVector3
{
    public float x;
    public float y;
    public float z;

    public SerializableVector3(Vector3 v)
    {
        x = v.x;
        y = v.y;
        z = v.z;
    }

    public Vector3 ToVector3()
    {
        return new Vector3(x, y, z);
    }
}

[Serializable]
public class InventoryEntry
{
    public string itemId;
    public int amount;
}

[Serializable]
public class InventorySnapshot
{
    public List<string> keys = new();
    public List<AccessCardLevel> cards = new();
    public List<InventoryEntry> items = new();
}

[Serializable]
public class SaveData
{
    public string sceneName;
    public SerializableVector3 playerPosition;
    public SerializableVector3 playerEuler;
    public float stamina;
    public InventorySnapshot inventory;
    public List<string> storyFlags;
}

public class SaveSystem : MonoBehaviour
{
    public static SaveSystem Instance { get; private set; }

    private string SavePath => Path.Combine(Application.persistentDataPath, "savegame.json");

    [Header("References")]
    [SerializeField] private Transform player;
    [SerializeField] private PlayerStamina playerStamina;
    [SerializeField] private InventorySystem inventorySystem;

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

    public void SaveGame()
    {
        ResolveReferences();

        if (player == null)
        {
            Debug.LogWarning("SaveSystem: Player not found.");
            return;
        }

        SaveData data = new SaveData
        {
            sceneName = SceneManager.GetActiveScene().name,
            playerPosition = new SerializableVector3(player.position),
            playerEuler = new SerializableVector3(player.eulerAngles),
            stamina = playerStamina != null ? playerStamina.CurrentStamina : 0f,
            inventory = inventorySystem != null ? inventorySystem.CreateSnapshot() : new InventorySnapshot(),
            storyFlags = StoryFlags.Instance != null ? StoryFlags.Instance.GetAllFlags() : new List<string>()
        };

        string json = JsonUtility.ToJson(data, true);
        File.WriteAllText(SavePath, json);

        Debug.Log($"Game saved at: {SavePath}");
    }

    public void LoadGame()
    {
        if (!File.Exists(SavePath))
        {
            Debug.LogWarning("SaveSystem: No save file found.");
            return;
        }

        string json = File.ReadAllText(SavePath);
        SaveData data = JsonUtility.FromJson<SaveData>(json);

        if (SceneManager.GetActiveScene().name != data.sceneName)
        {
            StartCoroutine(LoadSceneThenRestore(data.sceneName, data));
        }
        else
        {
            RestoreGame(data);
        }
    }

    private IEnumerator LoadSceneThenRestore(string sceneName, SaveData data)
    {
        AsyncOperation op = SceneManager.LoadSceneAsync(sceneName);

        while (!op.isDone)
            yield return null;

        yield return null;
        RestoreGame(data);
    }

    private void RestoreGame(SaveData data)
    {
        ResolveReferences();

        if (player != null)
        {
            CharacterController cc = player.GetComponent<CharacterController>();
            if (cc != null) cc.enabled = false;

            player.position = data.playerPosition.ToVector3();
            player.rotation = Quaternion.Euler(data.playerEuler.ToVector3());

            if (cc != null) cc.enabled = true;
        }

        if (playerStamina != null)
            playerStamina.SetCurrent(data.stamina);

        if (inventorySystem != null)
            inventorySystem.RestoreSnapshot(data.inventory);

        if (StoryFlags.Instance != null)
            StoryFlags.Instance.RestoreFlags(data.storyFlags);

        Debug.Log("Game loaded.");
    }

    public bool HasSaveFile()
    {
        return File.Exists(SavePath);
    }

    public void DeleteSave()
    {
        if (File.Exists(SavePath))
            File.Delete(SavePath);
    }

    private void ResolveReferences()
    {
        if (player == null)
            player = GameObject.FindGameObjectWithTag("Player")?.transform;

        if (playerStamina == null && player != null)
            playerStamina = player.GetComponent<PlayerStamina>();

        if (inventorySystem == null)
            inventorySystem = FindObjectOfType<InventorySystem>();
    }
}
```

---

# 6) CheckpointSystem.cs

**Ruta:** `Assets/Scripts/Core/CheckpointSystem.cs`
**Se coloca en:** `Bootstrap_Managers/CoreManagers/CheckpointSystem`

```csharp
using System;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

[Serializable]
public class CheckpointSaveData
{
    public string sceneName;
    public SerializableVector3 playerPosition;
    public SerializableVector3 playerEuler;
    public float stamina;
    public InventorySnapshot inventory;
    public List<string> storyFlags;
}

public class CheckpointSystem : MonoBehaviour
{
    public static CheckpointSystem Instance { get; private set; }

    private const string SaveKey = "LAST_CHECKPOINT";

    [Header("References")]
    [SerializeField] private Transform player;
    [SerializeField] private PlayerStamina playerStamina;
    [SerializeField] private InventorySystem inventory;

    public bool HasCheckpoint => PlayerPrefs.HasKey(SaveKey);

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

    public void SaveCheckpoint(Transform checkpointTransform)
    {
        ResolveReferences();

        CheckpointSaveData data = new CheckpointSaveData
        {
            sceneName = SceneManager.GetActiveScene().name,
            playerPosition = new SerializableVector3(checkpointTransform.position),
            playerEuler = new SerializableVector3(new Vector3(0f, player != null ? player.eulerAngles.y : 0f, 0f)),
            stamina = playerStamina != null ? playerStamina.CurrentStamina : 0f,
            inventory = inventory != null ? inventory.CreateSnapshot() : new InventorySnapshot(),
            storyFlags = StoryFlags.Instance != null ? StoryFlags.Instance.GetAllFlags() : new List<string>()
        };

        string json = JsonUtility.ToJson(data);
        PlayerPrefs.SetString(SaveKey, json);
        PlayerPrefs.Save();

        Debug.Log("Checkpoint saved.");
    }

    public void LoadLastCheckpoint()
    {
        if (!HasCheckpoint)
        {
            Debug.LogWarning("No checkpoint saved.");
            return;
        }

        string json = PlayerPrefs.GetString(SaveKey);
        CheckpointSaveData data = JsonUtility.FromJson<CheckpointSaveData>(json);

        if (SceneManager.GetActiveScene().name != data.sceneName)
        {
            SceneManager.sceneLoaded += OnSceneLoadedRestore;
            SceneManager.LoadScene(data.sceneName);
            return;
        }

        RestoreData(data);
    }

    private void OnSceneLoadedRestore(Scene scene, LoadSceneMode mode)
    {
        SceneManager.sceneLoaded -= OnSceneLoadedRestore;

        if (!HasCheckpoint)
            return;

        string json = PlayerPrefs.GetString(SaveKey);
        CheckpointSaveData data = JsonUtility.FromJson<CheckpointSaveData>(json);
        RestoreData(data);
    }

    private void RestoreData(CheckpointSaveData data)
    {
        ResolveReferences();

        if (player != null)
        {
            CharacterController cc = player.GetComponent<CharacterController>();
            if (cc != null) cc.enabled = false;

            player.position = data.playerPosition.ToVector3();
            player.rotation = Quaternion.Euler(data.playerEuler.ToVector3());

            if (cc != null) cc.enabled = true;
        }

        if (playerStamina != null)
            playerStamina.SetCurrent(data.stamina);

        if (inventory != null)
            inventory.RestoreSnapshot(data.inventory);

        if (StoryFlags.Instance != null)
            StoryFlags.Instance.RestoreFlags(data.storyFlags);

        Debug.Log("Checkpoint loaded.");
    }

    public void ClearCheckpoint()
    {
        PlayerPrefs.DeleteKey(SaveKey);
    }

    private void ResolveReferences()
    {
        if (player == null)
            player = GameObject.FindGameObjectWithTag("Player")?.transform;

        if (playerStamina == null && player != null)
            playerStamina = player.GetComponent<PlayerStamina>();

        if (inventory == null)
            inventory = FindObjectOfType<InventorySystem>();
    }
}
```

---

# 7) CheckpointTrigger.cs

**Ruta:** `Assets/Scripts/Core/CheckpointTrigger.cs`
**Se coloca en:** `PF_Checkpoint`

```csharp
using UnityEngine;

public class CheckpointTrigger : MonoBehaviour
{
    private bool activated;

    private void OnTriggerEnter(Collider other)
    {
        if (activated) return;
        if (!other.CompareTag("Player")) return;

        activated = true;
        CheckpointSystem.Instance?.SaveCheckpoint(transform);
    }
}
```

---

# 8) AccessCardLevel.cs

Este enum lo necesita el inventario y las puertas con tarjeta.

**Ruta:** `Assets/Scripts/Core/AccessCardLevel.cs`
**Se coloca en:** archivo suelto, no se añade a GameObject

```csharp
public enum AccessCardLevel
{
    None = 0,
    Blue = 1,
    Red = 2,
    Black = 3
}
```

---

# 9) InventorySystem.cs

Aunque lo meteremos luego otra vez en el bloque de interacción/items, **este script es dependencia directa de SaveSystem y CheckpointSystem**, así que conviene dejarlo ya puesto.

**Ruta:** `Assets/Scripts/Core/InventorySystem.cs`
**Se coloca en:** `Bootstrap_Managers/CoreManagers/InventorySystem`

```csharp
using System;
using System.Collections.Generic;
using UnityEngine;

public class InventorySystem : MonoBehaviour
{
    public static InventorySystem Instance { get; private set; }

    public event Action OnInventoryChanged;

    private readonly HashSet<string> keys = new();
    private readonly HashSet<AccessCardLevel> cards = new();
    private readonly Dictionary<string, int> items = new();

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

    public void AddKey(string keyId)
    {
        if (string.IsNullOrWhiteSpace(keyId))
            return;

        if (keys.Add(keyId))
            OnInventoryChanged?.Invoke();
    }

    public bool HasKey(string keyId)
    {
        return keys.Contains(keyId);
    }

    public void AddAccessCard(AccessCardLevel level)
    {
        if (level == AccessCardLevel.None)
            return;

        if (cards.Add(level))
            OnInventoryChanged?.Invoke();
    }

    public bool HasAccessCard(AccessCardLevel level)
    {
        return cards.Contains(level);
    }

    public void AddItem(string itemId, int amount = 1)
    {
        if (string.IsNullOrWhiteSpace(itemId) || amount <= 0)
            return;

        if (!items.ContainsKey(itemId))
            items[itemId] = 0;

        items[itemId] += amount;
        OnInventoryChanged?.Invoke();
    }

    public bool HasItem(string itemId, int amount = 1)
    {
        return items.TryGetValue(itemId, out int current) && current >= amount;
    }

    public bool RemoveItem(string itemId, int amount = 1)
    {
        if (!HasItem(itemId, amount))
            return false;

        items[itemId] -= amount;

        if (items[itemId] <= 0)
            items.Remove(itemId);

        OnInventoryChanged?.Invoke();
        return true;
    }

    public InventorySnapshot CreateSnapshot()
    {
        InventorySnapshot snapshot = new InventorySnapshot();

        foreach (string key in keys)
            snapshot.keys.Add(key);

        foreach (AccessCardLevel card in cards)
            snapshot.cards.Add(card);

        foreach (var pair in items)
        {
            snapshot.items.Add(new InventoryEntry
            {
                itemId = pair.Key,
                amount = pair.Value
            });
        }

        return snapshot;
    }

    public void RestoreSnapshot(InventorySnapshot snapshot)
    {
        keys.Clear();
        cards.Clear();
        items.Clear();

        if (snapshot != null)
        {
            foreach (string key in snapshot.keys)
                keys.Add(key);

            foreach (AccessCardLevel card in snapshot.cards)
                cards.Add(card);

            foreach (InventoryEntry entry in snapshot.items)
                items[entry.itemId] = entry.amount;
        }

        OnInventoryChanged?.Invoke();
    }
}
```

---

# Dónde poner este bloque exactamente

## En la escena Bootstrap o en tu prefab de managers:

Crea esta estructura:

```text
Bootstrap_Managers
└── CoreManagers
    ├── GameManager
    ├── SaveSystem
    ├── SceneLoader
    ├── StoryFlags
    ├── ObjectiveTracker
    ├── CheckpointSystem
    └── InventorySystem
```

Cada objeto puede ser un `Empty GameObject` con su script correspondiente.

---

# Referencias que debes asignar ahora

## SceneLoader

Asigna:

* `loadingCanvasGroup` → `Canvas_Gameplay/LoadingPanel`

## SaveSystem

Puedes dejarlo vacío al principio, porque intenta resolver:

* Player
* PlayerStamina
* InventorySystem

## CheckpointSystem

Igual: puede resolver referencias automáticamente, pero si quieres dejarlo fino:

* `player` → `PF_Player`
* `playerStamina` → componente del player
* `inventory` → `InventorySystem`

---

# Prueba rápida de este bloque

Antes de seguir al bloque 2, revisa esto:

## GameManager

* `Esc` pausa
* `Esc` reanuda
* `GameOverCaught()` congela el juego

## StoryFlags

Prueba desde otro script o botón:

```csharp
StoryFlags.Instance.SetFlag("TestFlag");
Debug.Log(StoryFlags.Instance.HasFlag("TestFlag"));
```

## ObjectiveTracker

Prueba:

```csharp
ObjectiveTracker.Instance.SetObjective("Find the flashlight");
```

## SceneLoader

Prueba un botón que llame:

```csharp
SceneLoader.Instance.LoadSceneByName("Level_01");
```

## CheckpointSystem

* entra en un checkpoint
* llama a `GameManager.Instance.RespawnFromCheckpoint()`
* debe recolocar al jugador

## SaveSystem

* guarda
* reinicia escena
* carga
* debe restaurar posición, stamina, inventario y flags

---

# Siguiente bloque

En el siguiente mensaje te paso el **Bloque 2 — Player** con código completo de:

* `PlayerMovement.cs`
* `PlayerStamina.cs` ya lo tienes, pero lo mantendremos coherente
* `PlayerHide.cs`
* `FlashlightController.cs`
* `PlayerItemHolder.cs`
* `PlayerAnimationBridge.cs`

Si quieres, seguimos justo con ese bloque.
