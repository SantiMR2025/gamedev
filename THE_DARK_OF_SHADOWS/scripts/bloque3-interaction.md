Perfecto. Vamos con el **Bloque 3 — Interaction / Items**.
Aquí está la parte que une al jugador con el mundo: interactuar, recoger, abrir, esconderse, usar teléfono, espejo, walkie, ascensor y lanzar objetos.

Te lo dejo listo para **copiar y pegar**.

---

# 1) IInteractable.cs

**Ruta:** `Assets/Scripts/Interaction/IInteractable.cs`
**Se coloca en:** archivo suelto, no se añade a GameObject

```csharp
public interface IInteractable
{
    string GetPrompt();
    bool CanInteract(InteractionSystem interactor);
    void Interact(InteractionSystem interactor);
}
```

---

# 2) InteractionSystem.cs

**Ruta:** `Assets/Scripts/Interaction/InteractionSystem.cs`
**Se coloca en:** `PF_Player` (objeto raíz)

```csharp
using TMPro;
using UnityEngine;

public class InteractionSystem : MonoBehaviour
{
    [Header("References")]
    [SerializeField] private Camera playerCamera;
    [SerializeField] private TMP_Text promptText;

    [Header("Interaction")]
    [SerializeField] private float interactDistance = 3f;
    [SerializeField] private LayerMask interactMask = ~0;
    [SerializeField] private KeyCode interactKey = KeyCode.E;

    public IInteractable CurrentInteractable { get; private set; }
    public GameObject CurrentObject { get; private set; }

    private void Update()
    {
        DetectInteractable();

        if (CurrentInteractable != null && Input.GetKeyDown(interactKey))
        {
            if (CurrentInteractable.CanInteract(this))
                CurrentInteractable.Interact(this);
        }
    }

    private void DetectInteractable()
    {
        ClearCurrent();

        if (playerCamera == null)
        {
            HidePrompt();
            return;
        }

        Ray ray = new Ray(playerCamera.transform.position, playerCamera.transform.forward);

        if (Physics.Raycast(ray, out RaycastHit hit, interactDistance, interactMask, QueryTriggerInteraction.Collide))
        {
            IInteractable interactable = hit.collider.GetComponentInParent<IInteractable>();

            if (interactable != null)
            {
                CurrentInteractable = interactable;
                CurrentObject = hit.collider.gameObject;

                string prompt = interactable.GetPrompt();

                if (promptText != null)
                {
                    promptText.text = prompt;
                    promptText.gameObject.SetActive(!string.IsNullOrWhiteSpace(prompt));
                }

                return;
            }
        }

        HidePrompt();
    }

    private void ClearCurrent()
    {
        CurrentInteractable = null;
        CurrentObject = null;
    }

    private void HidePrompt()
    {
        if (promptText != null)
            promptText.gameObject.SetActive(false);
    }
}
```

---

# 3) KeyPickup.cs

**Ruta:** `Assets/Scripts/Items/KeyPickup.cs`
**Se coloca en:** objeto llave en escena

```csharp
using UnityEngine;

public class KeyPickup : MonoBehaviour, IInteractable
{
    [SerializeField] private string keyId = "DefaultKey";
    [SerializeField] private string prompt = "Press E to pick up key";

    public string GetPrompt() => prompt;

    public bool CanInteract(InteractionSystem interactor)
    {
        return InventorySystem.Instance != null;
    }

    public void Interact(InteractionSystem interactor)
    {
        InventorySystem.Instance.AddKey(keyId);
        Destroy(gameObject);
    }
}
```

---

# 4) KeycardPickup.cs

**Ruta:** `Assets/Scripts/Items/KeycardPickup.cs`
**Se coloca en:** objeto keycard en escena

```csharp
using UnityEngine;

public class KeycardPickup : MonoBehaviour, IInteractable
{
    [SerializeField] private AccessCardLevel cardLevel = AccessCardLevel.Blue;
    [SerializeField] private string prompt = "Press E to pick up keycard";

    public string GetPrompt() => prompt;

    public bool CanInteract(InteractionSystem interactor)
    {
        return InventorySystem.Instance != null;
    }

    public void Interact(InteractionSystem interactor)
    {
        InventorySystem.Instance.AddAccessCard(cardLevel);
        Destroy(gameObject);
    }
}
```

---

# 5) FlashlightPickup.cs

**Ruta:** `Assets/Scripts/Items/FlashlightPickup.cs`
**Se coloca en:** objeto linterna del nivel 1

```csharp
using UnityEngine;

public class FlashlightPickup : MonoBehaviour, IInteractable
{
    [SerializeField] private string prompt = "Press E to pick up flashlight";

    public string GetPrompt() => prompt;

    public bool CanInteract(InteractionSystem interactor)
    {
        FlashlightController flashlight = interactor.GetComponent<FlashlightController>();
        return flashlight != null && !flashlight.IsUnlocked;
    }

    public void Interact(InteractionSystem interactor)
    {
        FlashlightController flashlight = interactor.GetComponent<FlashlightController>();
        if (flashlight == null)
            return;

        flashlight.UnlockFlashlight(true);
        Destroy(gameObject);
    }
}
```

---

# 6) FusePickup.cs

**Ruta:** `Assets/Scripts/Items/FusePickup.cs`
**Se coloca en:** objeto fusible en escena

```csharp
using UnityEngine;

public class FusePickup : MonoBehaviour, IInteractable
{
    [SerializeField] private string fuseItemId = "Fuse";
    [SerializeField] private string prompt = "Press E to pick up fuse";

    public string GetPrompt() => prompt;

    public bool CanInteract(InteractionSystem interactor)
    {
        return InventorySystem.Instance != null;
    }

    public void Interact(InteractionSystem interactor)
    {
        InventorySystem.Instance?.AddItem(fuseItemId, 1);
        Destroy(gameObject);
    }
}
```

---

# 7) DoorInteract.cs

**Ruta:** `Assets/Scripts/Interaction/DoorInteract.cs`
**Se coloca en:** `PF_Door_Standard`

```csharp
using UnityEngine;

public class DoorInteract : MonoBehaviour, IInteractable
{
    [Header("Lock")]
    [SerializeField] private bool requiresKey = false;
    [SerializeField] private string requiredKeyId = "DefaultKey";

    [Header("Animation")]
    [SerializeField] private Animator animator;
    [SerializeField] private string openForwardTrigger = "OpenForward";
    [SerializeField] private string openBackwardTrigger = "OpenBackward";

    [Header("State")]
    [SerializeField] private bool isOpen = false;
    [SerializeField] private string openPrompt = "Press E to open door";
    [SerializeField] private string lockedPrompt = "Door is locked";

    public string GetPrompt()
    {
        if (isOpen)
            return string.Empty;

        if (requiresKey && !HasRequiredKey())
            return lockedPrompt;

        return openPrompt;
    }

    public bool CanInteract(InteractionSystem interactor)
    {
        if (isOpen)
            return false;

        if (requiresKey && !HasRequiredKey())
            return false;

        return animator != null;
    }

    public void Interact(InteractionSystem interactor)
    {
        if (animator == null || isOpen)
            return;

        Vector3 toPlayer = (interactor.transform.position - transform.position).normalized;
        float dot = Vector3.Dot(transform.forward, toPlayer);

        if (dot >= 0f)
            animator.SetTrigger(openBackwardTrigger);
        else
            animator.SetTrigger(openForwardTrigger);

        isOpen = true;
    }

    private bool HasRequiredKey()
    {
        if (!requiresKey)
            return true;

        return InventorySystem.Instance != null && InventorySystem.Instance.HasKey(requiredKeyId);
    }
}
```

---

# 8) KeycardDoor.cs

**Ruta:** `Assets/Scripts/Interaction/KeycardDoor.cs`
**Se coloca en:** `PF_Door_Keycard`

```csharp
using UnityEngine;

public class KeycardDoor : MonoBehaviour, IInteractable
{
    [SerializeField] private AccessCardLevel requiredLevel = AccessCardLevel.Blue;
    [SerializeField] private Animator animator;
    [SerializeField] private string openTrigger = "Open";
    [SerializeField] private bool isOpen = false;

    [SerializeField] private string prompt = "Press E to use keycard";
    [SerializeField] private string lockedPrompt = "Access denied";

    public string GetPrompt()
    {
        if (isOpen)
            return string.Empty;

        return HasAccess() ? prompt : lockedPrompt;
    }

    public bool CanInteract(InteractionSystem interactor)
    {
        return !isOpen && animator != null && HasAccess();
    }

    public void Interact(InteractionSystem interactor)
    {
        if (isOpen || animator == null)
            return;

        animator.SetTrigger(openTrigger);
        isOpen = true;
    }

    private bool HasAccess()
    {
        return InventorySystem.Instance != null &&
               InventorySystem.Instance.HasAccessCard(requiredLevel);
    }
}
```

---

# 9) HideSpot.cs

**Ruta:** `Assets/Scripts/Interaction/HideSpot.cs`
**Se coloca en:** `PF_HideSpot_Locker`

```csharp
using UnityEngine;

public class HideSpot : MonoBehaviour, IInteractable
{
    [SerializeField] private Transform hidePoint;
    [SerializeField] private Transform exitPoint;
    [SerializeField] private Transform inspectPoint;
    [SerializeField] private string enterPrompt = "Press E to hide";
    [SerializeField] private string occupiedPrompt = "Occupied";

    public bool IsOccupied { get; private set; }

    public Transform HidePoint => hidePoint;
    public Transform ExitPoint => exitPoint != null ? exitPoint : transform;
    public Transform InspectPoint => inspectPoint != null ? inspectPoint : transform;

    private PlayerHide hiddenPlayer;

    public string GetPrompt()
    {
        return IsOccupied ? occupiedPrompt : enterPrompt;
    }

    public bool CanInteract(InteractionSystem interactor)
    {
        PlayerHide hide = interactor.GetComponent<PlayerHide>();
        if (hide == null)
            return false;

        if (hide.IsHidden)
            return hide.CurrentSpot == this;

        return !IsOccupied;
    }

    public void Interact(InteractionSystem interactor)
    {
        PlayerHide hide = interactor.GetComponent<PlayerHide>();
        if (hide == null)
            return;

        if (hide.IsHidden && hide.CurrentSpot == this)
            hide.ExitHideSpot();
        else if (!IsOccupied)
            hide.EnterHideSpot(this);
    }

    public void SetOccupied(bool occupied, PlayerHide playerHide = null)
    {
        IsOccupied = occupied;
        hiddenPlayer = occupied ? playerHide : null;
    }

    public bool HasHiddenPlayer()
    {
        return IsOccupied && hiddenPlayer != null && hiddenPlayer.IsHidden && hiddenPlayer.CurrentSpot == this;
    }

    public PlayerHide GetHiddenPlayer()
    {
        return HasHiddenPlayer() ? hiddenPlayer : null;
    }
}
```

---

# 10) PhoneInteract.cs

**Ruta:** `Assets/Scripts/Interaction/PhoneInteract.cs`
**Se coloca en:** teléfono del nivel 1

```csharp
using UnityEngine;
using UnityEngine.Events;

public class PhoneInteract : MonoBehaviour, IInteractable
{
    [SerializeField] private string prompt = "Press E to answer phone";
    [SerializeField] private string missingFlashlightPrompt = "You need the flashlight first";
    [SerializeField] private UnityEvent onPhoneUsed;

    public string GetPrompt()
    {
        return HasFlashlight() ? prompt : missingFlashlightPrompt;
    }

    public bool CanInteract(InteractionSystem interactor)
    {
        return HasFlashlight();
    }

    public void Interact(InteractionSystem interactor)
    {
        onPhoneUsed?.Invoke();
    }

    private bool HasFlashlight()
    {
        FlashlightController flashlight = FindObjectOfType<FlashlightController>();
        return flashlight != null && flashlight.IsUnlocked;
    }
}
```

---

# 11) MirrorInteract.cs

**Ruta:** `Assets/Scripts/Interaction/MirrorInteract.cs`
**Se coloca en:** espejo del nivel 2

```csharp
using UnityEngine;
using UnityEngine.Events;

public class MirrorInteract : MonoBehaviour, IInteractable
{
    [SerializeField] private string prompt = "Press E to inspect mirror";
    [SerializeField] private UnityEvent onMirrorUsed;

    public string GetPrompt() => prompt;

    public bool CanInteract(InteractionSystem interactor)
    {
        return true;
    }

    public void Interact(InteractionSystem interactor)
    {
        onMirrorUsed?.Invoke();
    }
}
```

---

# 12) WalkieTalkieInteract.cs

**Ruta:** `Assets/Scripts/Interaction/WalkieTalkieInteract.cs`
**Se coloca en:** walkie-talkie del nivel 3

```csharp
using UnityEngine;
using UnityEngine.Events;

public class WalkieTalkieInteract : MonoBehaviour, IInteractable
{
    [SerializeField] private string prompt = "Press E to inspect walkie-talkie";
    [SerializeField] private UnityEvent onWalkieUsed;
    [SerializeField] private bool oneUseOnly = true;

    private bool used;

    public string GetPrompt()
    {
        if (used && oneUseOnly)
            return string.Empty;

        return prompt;
    }

    public bool CanInteract(InteractionSystem interactor)
    {
        return !(used && oneUseOnly);
    }

    public void Interact(InteractionSystem interactor)
    {
        if (used && oneUseOnly)
            return;

        used = true;
        onWalkieUsed?.Invoke();
    }
}
```

---

# 13) ElevatorAccess.cs

**Ruta:** `Assets/Scripts/Interaction/ElevatorAccess.cs`
**Se coloca en:** panel o puerta del ascensor

```csharp
using UnityEngine;
using UnityEngine.Events;

public class ElevatorAccess : MonoBehaviour, IInteractable
{
    [Header("Requirements")]
    [SerializeField] private bool requiresPower = true;
    [SerializeField] private bool requiresKeycard = true;
    [SerializeField] private AccessCardLevel requiredCardLevel = AccessCardLevel.Black;

    [Header("Prompt")]
    [SerializeField] private string usePrompt = "Press E to use elevator";
    [SerializeField] private string noPowerPrompt = "The elevator has no power";
    [SerializeField] private string noCardPrompt = "Access card required";

    [Header("Events")]
    [SerializeField] private UnityEvent onElevatorUsed;

    public string GetPrompt()
    {
        if (requiresPower && !HasPower())
            return noPowerPrompt;

        if (requiresKeycard && !HasCard())
            return noCardPrompt;

        return usePrompt;
    }

    public bool CanInteract(InteractionSystem interactor)
    {
        if (requiresPower && !HasPower())
            return false;

        if (requiresKeycard && !HasCard())
            return false;

        return true;
    }

    public void Interact(InteractionSystem interactor)
    {
        onElevatorUsed?.Invoke();
    }

    private bool HasPower()
    {
        return StoryFlags.Instance != null && StoryFlags.Instance.HasFlag("GeneratorPowered");
    }

    private bool HasCard()
    {
        return InventorySystem.Instance != null &&
               InventorySystem.Instance.HasAccessCard(requiredCardLevel);
    }
}
```

---

# 14) SavePointInteract.cs

**Ruta:** `Assets/Scripts/Interaction/SavePointInteract.cs`
**Se coloca en:** `PF_SavePoint`

```csharp
using UnityEngine;

public class SavePointInteract : MonoBehaviour, IInteractable
{
    [SerializeField] private string prompt = "Press E to save game";

    public string GetPrompt() => prompt;

    public bool CanInteract(InteractionSystem interactor)
    {
        return SaveSystem.Instance != null;
    }

    public void Interact(InteractionSystem interactor)
    {
        SaveSystem.Instance?.SaveGame();
    }
}
```

---

# 15) ThrowableItem.cs

**Ruta:** `Assets/Scripts/Items/ThrowableItem.cs`
**Se coloca en:** botellas, cajas, herramientas, etc.

```csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody))]
public class ThrowableItem : MonoBehaviour, IInteractable
{
    [Header("Pickup")]
    [SerializeField] private string pickupPrompt = "Press E to pick up object";
    [SerializeField] private Transform holdPoint;

    [Header("Throw")]
    [SerializeField] private float throwForce = 10f;
    [SerializeField] private float upwardForce = 1.5f;

    [Header("Noise")]
    [SerializeField] private float impactNoiseIntensity = 1f;
    [SerializeField] private float minImpactVelocity = 1.25f;

    private Rigidbody rb;
    private Collider[] colliders;
    private Transform currentHolder;
    private bool isHeld;

    private void Awake()
    {
        rb = GetComponent<Rigidbody>();
        colliders = GetComponentsInChildren<Collider>();
    }

    private void Update()
    {
        if (!isHeld || currentHolder == null)
            return;

        transform.position = currentHolder.position;
        transform.rotation = currentHolder.rotation;

        if (Input.GetMouseButtonDown(0))
            Throw();
    }

    public string GetPrompt()
    {
        return isHeld ? string.Empty : pickupPrompt;
    }

    public bool CanInteract(InteractionSystem interactor)
    {
        return !isHeld;
    }

    public void Interact(InteractionSystem interactor)
    {
        if (isHeld)
            return;

        Transform targetHoldPoint = holdPoint;

        if (targetHoldPoint == null)
        {
            PlayerItemHolder holder = interactor.GetComponent<PlayerItemHolder>();
            if (holder != null)
                targetHoldPoint = holder.HoldPoint;
        }

        if (targetHoldPoint == null)
            return;

        PickUp(targetHoldPoint);
    }

    private void PickUp(Transform holder)
    {
        isHeld = true;
        currentHolder = holder;

        rb.isKinematic = true;
        rb.velocity = Vector3.zero;
        rb.angularVelocity = Vector3.zero;

        foreach (Collider c in colliders)
            c.enabled = false;
    }

    private void Throw()
    {
        isHeld = false;

        rb.isKinematic = false;
        transform.SetParent(null);

        foreach (Collider c in colliders)
            c.enabled = true;

        Vector3 force = currentHolder.forward * throwForce + currentHolder.up * upwardForce;
        rb.AddForce(force, ForceMode.Impulse);

        currentHolder = null;
    }

    private void OnCollisionEnter(Collision collision)
    {
        if (rb == null || isHeld)
            return;

        if (collision.relativeVelocity.magnitude >= minImpactVelocity)
        {
            NoiseSystem.Emit(
                transform.position,
                impactNoiseIntensity,
                NoiseSourceType.ThrowableImpact,
                NoisePriority.High,
                transform
            );
        }
    }
}
```

---

# 16) Dónde poner cada script de este bloque

## En PF_Player

Añade:

* `InteractionSystem`

Y asígnale:

* `playerCamera` → `MainCamera`
* `promptText` → texto UI de interacción
* `interactMask` → capas `Interactable`, `HideSpot`, etc.

---

## En objetos de escena

### Pickups

* llaves → `KeyPickup`
* keycards → `KeycardPickup`
* linterna → `FlashlightPickup`
* fusibles → `FusePickup`

### Puertas

* puerta abatible → `DoorInteract`
* puerta con tarjeta → `KeycardDoor`

### Escondites

* armario/taquilla/caja → `HideSpot`

### Interacciones de historia

* teléfono → `PhoneInteract`
* espejo → `MirrorInteract`
* walkie → `WalkieTalkieInteract`
* ascensor → `ElevatorAccess`
* save point → `SavePointInteract`

### Objetos lanzables

* botella/caja/herramienta → `ThrowableItem` + `Rigidbody` + collider

---

# 17) Referencias que debes asignar

## InteractionSystem

* `promptText` → UI de interacción
* `interactMask` → layers interactuables

## DoorInteract

* `animator` → animator de la puerta
* triggers:

  * `OpenForward`
  * `OpenBackward`

## KeycardDoor

* `animator` → animator
* `requiredLevel` → Blue / Red / Black

## HideSpot

* `hidePoint`
* `exitPoint`
* `inspectPoint`

## Phone / Mirror / Walkie / Elevator

Conecta sus `UnityEvent` a:

* Timeline
* cambio de escena
* flags
* subtítulos
* objetivos

## ThrowableItem

* `holdPoint` puede dejarse vacío si usas el `HoldPoint` del jugador

---

# 18) Prueba rápida de este bloque

## Interacción

* al mirar un objeto interactivo sale el prompt
* con `E` interactúa

## Pickups

* llave se recoge
* keycard se recoge
* linterna se desbloquea
* fusible entra al inventario

## Puertas

* puerta normal abre
* puerta con llave no abre sin llave
* puerta con keycard no abre sin tarjeta

## Escondites

* `E` entra
* `E` sale
* no puedes entrar si está ocupado

## Objetos lanzables

* `E` recoge
* click izquierdo lanza
* al chocar emite ruido

## Teléfono / espejo / walkie

* disparan sus eventos

---

# 19) Dependencias de este bloque

Este bloque necesita:

* `NoiseSystem`
* `NoisePriority`
* `NoiseSourceType`

Eso te lo voy a dejar ya resuelto en el siguiente bloque de **Enemy / Audio base**, para que no te salten errores.

---

# Siguiente bloque

En el siguiente mensaje te paso el **Bloque 4 — Enemy + ruido base**, con código completo de:

* `NoisePriority.cs`
* `NoiseSourceType.cs`
* `NoiseEventData.cs`
* `NoiseSystem.cs`
* `EnemyInvestigationMemory.cs`
* `EnemyCommunicationSystem.cs`
* `EnemyHideSpotSearch.cs`
* `EnemyFlankHelper.cs`
* `TimedNoiseMemory.cs`
* `EnemyAnimatorBridge.cs`
* `EnemyAI.cs`
* `StealthKillTarget.cs`
* `StealthKillTargetKeyDrop.cs`

Seguimos con ese.
