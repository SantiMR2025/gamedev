Perfecto. Seguimos con el **Bloque 2 — Player**.
Este bloque controla el movimiento, la stamina, esconderse, la linterna, el punto donde sujetas objetos y el puente con animaciones.

Te lo dejo otra vez en formato **copiar y pegar**.

---

# 1) PlayerStamina.cs

**Ruta:** `Assets/Scripts/Player/PlayerStamina.cs`
**Se coloca en:** `PF_Player` (objeto raíz)

```csharp
using System;
using UnityEngine;

public class PlayerStamina : MonoBehaviour
{
    [Header("Stamina")]
    [SerializeField] private float maxStamina = 5f;
    [SerializeField] private float sprintDrainPerSecond = 1f;
    [SerializeField] private float recoveryPerSecond = 1.5f;
    [SerializeField] private float recoveryDelay = 1f;

    public float CurrentStamina { get; private set; }
    public float MaxStamina => maxStamina;

    public bool IsDepleted => CurrentStamina <= 0.01f;
    public bool CanSprint => CurrentStamina > 0.05f;

    public event Action<float, float> OnStaminaChanged;
    public event Action OnStaminaDepleted;
    public event Action OnStaminaRecoveredFromZero;

    private float recoveryTimer;
    private bool wasDepleted;

    private void Awake()
    {
        CurrentStamina = maxStamina;
        Notify();
    }

    private void Update()
    {
        if (recoveryTimer > 0f)
            recoveryTimer -= Time.deltaTime;
        else
            Recover(Time.deltaTime * recoveryPerSecond);
    }

    public bool TryUseSprint(float deltaTime)
    {
        if (!CanSprint)
            return false;

        Consume(deltaTime * sprintDrainPerSecond);
        recoveryTimer = recoveryDelay;
        return true;
    }

    public void Consume(float amount)
    {
        float previous = CurrentStamina;
        CurrentStamina = Mathf.Clamp(CurrentStamina - amount, 0f, maxStamina);

        if (previous > 0f && CurrentStamina <= 0f)
        {
            wasDepleted = true;
            OnStaminaDepleted?.Invoke();
        }

        Notify();
    }

    public void Recover(float amount)
    {
        float previous = CurrentStamina;
        CurrentStamina = Mathf.Clamp(CurrentStamina + amount, 0f, maxStamina);

        if (wasDepleted && previous <= 0.01f && CurrentStamina > 0.25f)
        {
            wasDepleted = false;
            OnStaminaRecoveredFromZero?.Invoke();
        }

        Notify();
    }

    public void SetCurrent(float value)
    {
        CurrentStamina = Mathf.Clamp(value, 0f, maxStamina);
        Notify();
    }

    private void Notify()
    {
        OnStaminaChanged?.Invoke(CurrentStamina, maxStamina);
    }
}
```

---

# 2) PlayerMovement.cs

**Ruta:** `Assets/Scripts/Player/PlayerMovement.cs`
**Se coloca en:** `PF_Player` (objeto raíz)

```csharp
using UnityEngine;

[RequireComponent(typeof(CharacterController))]
[RequireComponent(typeof(PlayerStamina))]
public class PlayerMovement : MonoBehaviour
{
    public enum MoveState
    {
        Idle,
        Walking,
        Sprinting,
        Crouching,
        Airborne
    }

    [Header("References")]
    [SerializeField] private CharacterController controller;
    [SerializeField] private Transform groundCheck;

    [Header("Movement")]
    [SerializeField] private float walkSpeed = 4f;
    [SerializeField] private float sprintSpeed = 7f;
    [SerializeField] private float crouchSpeed = 2f;
    [SerializeField] private float acceleration = 12f;
    [SerializeField] private float airControl = 0.45f;

    [Header("Jump / Gravity")]
    [SerializeField] private float gravity = -20f;
    [SerializeField] private float jumpHeight = 1.2f;
    [SerializeField] private float groundedGravity = -2f;

    [Header("Ground Check")]
    [SerializeField] private float groundRadius = 0.25f;
    [SerializeField] private LayerMask groundMask;

    [Header("Crouch")]
    [SerializeField] private float standingHeight = 1.8f;
    [SerializeField] private float crouchHeight = 1.1f;
    [SerializeField] private float crouchLerpSpeed = 10f;

    public MoveState CurrentState { get; private set; } = MoveState.Idle;
    public Vector3 HorizontalVelocity => new Vector3(currentVelocity.x, 0f, currentVelocity.z);
    public bool IsGrounded { get; private set; }
    public bool IsCrouching { get; private set; }
    public bool IsSprinting => CurrentState == MoveState.Sprinting;
    public float NormalizedSpeed { get; private set; }

    private PlayerStamina stamina;
    private Vector3 currentVelocity;
    private float verticalVelocity;

    private void Reset()
    {
        controller = GetComponent<CharacterController>();
    }

    private void Awake()
    {
        if (controller == null)
            controller = GetComponent<CharacterController>();

        stamina = GetComponent<PlayerStamina>();
    }

    private void Update()
    {
        CheckGround();
        HandleCrouch();
        HandleMovement();
        HandleJump();
        ApplyGravity();
        UpdateState();
    }

    private void CheckGround()
    {
        if (groundCheck == null)
            return;

        IsGrounded = Physics.CheckSphere(
            groundCheck.position,
            groundRadius,
            groundMask,
            QueryTriggerInteraction.Ignore
        );

        if (IsGrounded && verticalVelocity < 0f)
            verticalVelocity = groundedGravity;
    }

    private void HandleMovement()
    {
        float x = Input.GetAxisRaw("Horizontal");
        float z = Input.GetAxisRaw("Vertical");

        Vector3 inputDir = (transform.right * x + transform.forward * z).normalized;
        bool hasMoveInput = inputDir.sqrMagnitude > 0.001f;

        bool sprintHeld = Input.GetKey(KeyCode.LeftShift);
        bool wantsSprint = sprintHeld && hasMoveInput && !IsCrouching && IsGrounded;

        float targetSpeed = walkSpeed;

        if (IsCrouching)
        {
            targetSpeed = crouchSpeed;
        }
        else if (wantsSprint && stamina.TryUseSprint(Time.deltaTime))
        {
            targetSpeed = sprintSpeed;
        }

        Vector3 targetHorizontalVelocity = inputDir * targetSpeed;

        float accel = IsGrounded ? acceleration : acceleration * airControl;

        Vector3 currentHorizontal = new Vector3(currentVelocity.x, 0f, currentVelocity.z);
        currentHorizontal = Vector3.Lerp(currentHorizontal, targetHorizontalVelocity, accel * Time.deltaTime);

        currentVelocity.x = currentHorizontal.x;
        currentVelocity.z = currentHorizontal.z;

        controller.Move(currentHorizontal * Time.deltaTime);

        NormalizedSpeed = targetSpeed <= 0.01f ? 0f : currentHorizontal.magnitude / sprintSpeed;
    }

    private void HandleJump()
    {
        if (Input.GetKeyDown(KeyCode.Space) && IsGrounded)
        {
            verticalVelocity = Mathf.Sqrt(jumpHeight * -2f * gravity);
        }
    }

    private void ApplyGravity()
    {
        verticalVelocity += gravity * Time.deltaTime;
        controller.Move(Vector3.up * verticalVelocity * Time.deltaTime);
    }

    private void HandleCrouch()
    {
        IsCrouching = Input.GetKey(KeyCode.LeftControl);

        float targetHeight = IsCrouching ? crouchHeight : standingHeight;
        controller.height = Mathf.Lerp(controller.height, targetHeight, crouchLerpSpeed * Time.deltaTime);

        Vector3 center = controller.center;
        center.y = controller.height * 0.5f;
        controller.center = center;
    }

    private void UpdateState()
    {
        if (!IsGrounded)
        {
            CurrentState = MoveState.Airborne;
            return;
        }

        float horizontalSpeed = HorizontalVelocity.magnitude;

        if (horizontalSpeed < 0.1f)
        {
            CurrentState = MoveState.Idle;
        }
        else if (IsCrouching)
        {
            CurrentState = MoveState.Crouching;
        }
        else if (Input.GetKey(KeyCode.LeftShift) && stamina.CanSprint)
        {
            CurrentState = MoveState.Sprinting;
        }
        else
        {
            CurrentState = MoveState.Walking;
        }
    }
}
```

---

# 3) PlayerHide.cs

**Ruta:** `Assets/Scripts/Player/PlayerHide.cs`
**Se coloca en:** `PF_Player` (objeto raíz)

```csharp
using System;
using UnityEngine;

[RequireComponent(typeof(CharacterController))]
[RequireComponent(typeof(PlayerMovement))]
public class PlayerHide : MonoBehaviour
{
    public static event Action<PlayerHide, HideSpot> OnPlayerEnteredHideSpot;
    public static event Action<PlayerHide, HideSpot> OnPlayerExitedHideSpot;

    [SerializeField] private Camera playerCamera;
    [SerializeField] private Transform cameraHiddenAnchor;
    [SerializeField] private float hiddenCameraLerpSpeed = 8f;

    public bool IsHidden { get; private set; }
    public HideSpot CurrentSpot { get; private set; }

    private CharacterController characterController;
    private PlayerMovement playerMovement;
    private Vector3 cameraOriginalLocalPos;
    private Quaternion cameraOriginalLocalRot;

    private void Awake()
    {
        characterController = GetComponent<CharacterController>();
        playerMovement = GetComponent<PlayerMovement>();

        if (playerCamera != null)
        {
            cameraOriginalLocalPos = playerCamera.transform.localPosition;
            cameraOriginalLocalRot = playerCamera.transform.localRotation;
        }
    }

    private void Update()
    {
        if (!IsHidden)
            return;

        if (Input.GetKeyDown(KeyCode.E))
            ExitHideSpot();

        UpdateHiddenCamera();
    }

    public void EnterHideSpot(HideSpot spot)
    {
        if (spot == null || IsHidden)
            return;

        CurrentSpot = spot;
        IsHidden = true;

        spot.SetOccupied(true, this);

        characterController.enabled = false;
        transform.position = spot.HidePoint.position;
        transform.rotation = spot.HidePoint.rotation;
        characterController.enabled = true;

        playerMovement.enabled = false;

        OnPlayerEnteredHideSpot?.Invoke(this, spot);
    }

    public void ExitHideSpot()
    {
        if (!IsHidden || CurrentSpot == null)
            return;

        HideSpot previousSpot = CurrentSpot;

        characterController.enabled = false;
        transform.position = previousSpot.ExitPoint.position;
        characterController.enabled = true;

        previousSpot.SetOccupied(false, null);

        IsHidden = false;
        CurrentSpot = null;

        playerMovement.enabled = true;

        if (playerCamera != null)
        {
            playerCamera.transform.localPosition = cameraOriginalLocalPos;
            playerCamera.transform.localRotation = cameraOriginalLocalRot;
        }

        OnPlayerExitedHideSpot?.Invoke(this, previousSpot);
    }

    private void UpdateHiddenCamera()
    {
        if (playerCamera == null || cameraHiddenAnchor == null)
            return;

        playerCamera.transform.position = Vector3.Lerp(
            playerCamera.transform.position,
            cameraHiddenAnchor.position,
            hiddenCameraLerpSpeed * Time.deltaTime
        );

        playerCamera.transform.rotation = Quaternion.Slerp(
            playerCamera.transform.rotation,
            cameraHiddenAnchor.rotation,
            hiddenCameraLerpSpeed * Time.deltaTime
        );
    }
}
```

---

# 4) FlashlightController.cs

**Ruta:** `Assets/Scripts/Player/FlashlightController.cs`
**Se coloca en:** `PF_Player` (objeto raíz)

```csharp
using UnityEngine;

public class FlashlightController : MonoBehaviour
{
    [Header("References")]
    [SerializeField] private Light flashlightLight;
    [SerializeField] private AudioSource toggleAudio;

    [Header("Settings")]
    [SerializeField] private KeyCode toggleKey = KeyCode.F;
    [SerializeField] private bool startsUnlocked = false;
    [SerializeField] private bool useBattery = false;
    [SerializeField] private float maxBattery = 120f;
    [SerializeField] private float batteryDrainPerSecond = 1f;

    public bool IsUnlocked { get; private set; }
    public bool IsOn { get; private set; }
    public float CurrentBattery { get; private set; }

    private void Awake()
    {
        IsUnlocked = startsUnlocked;
        CurrentBattery = maxBattery;
        ApplyVisual(false);
    }

    private void Update()
    {
        if (IsUnlocked && Input.GetKeyDown(toggleKey))
        {
            Toggle();
        }

        if (useBattery && IsOn)
        {
            CurrentBattery -= batteryDrainPerSecond * Time.deltaTime;

            if (CurrentBattery <= 0f)
            {
                CurrentBattery = 0f;
                IsOn = false;
                ApplyVisual(false);
            }
        }
    }

    public void UnlockFlashlight(bool autoTurnOn = true)
    {
        IsUnlocked = true;

        if (autoTurnOn)
            IsOn = true;

        ApplyVisual(false);
    }

    public void Toggle()
    {
        if (!IsUnlocked)
            return;

        if (useBattery && CurrentBattery <= 0f)
            return;

        IsOn = !IsOn;
        ApplyVisual(true);
    }

    private void ApplyVisual(bool playSound)
    {
        if (flashlightLight != null)
            flashlightLight.enabled = IsUnlocked && IsOn;

        if (playSound && toggleAudio != null && IsUnlocked)
            toggleAudio.Play();
    }
}
```

---

# 5) PlayerItemHolder.cs

**Ruta:** `Assets/Scripts/Player/PlayerItemHolder.cs`
**Se coloca en:** `PF_Player` (objeto raíz)

```csharp
using UnityEngine;

public class PlayerItemHolder : MonoBehaviour
{
    [SerializeField] private Transform holdPoint;
    public Transform HoldPoint => holdPoint;
}
```

---

# 6) PlayerAnimationBridge.cs

**Ruta:** `Assets/Scripts/Player/PlayerAnimationBridge.cs`
**Se coloca en:** `PF_Player` (objeto raíz o en el objeto donde esté el Animator del player)

```csharp
using UnityEngine;

[RequireComponent(typeof(Animator))]
public class PlayerAnimationBridge : MonoBehaviour
{
    [SerializeField] private PlayerMovement movement;
    [SerializeField] private Animator animator;

    [Header("Parameters")]
    [SerializeField] private string speedParam = "Speed";
    [SerializeField] private string groundedParam = "Grounded";
    [SerializeField] private string crouchParam = "Crouching";
    [SerializeField] private string hiddenParam = "Hidden";

    private PlayerHide playerHide;

    private void Reset()
    {
        animator = GetComponent<Animator>();
        movement = GetComponent<PlayerMovement>();
    }

    private void Awake()
    {
        playerHide = GetComponent<PlayerHide>();
    }

    private void Update()
    {
        if (animator == null || movement == null)
            return;

        animator.SetFloat(speedParam, movement.NormalizedSpeed);
        animator.SetBool(groundedParam, movement.IsGrounded);
        animator.SetBool(crouchParam, movement.IsCrouching);
        animator.SetBool(hiddenParam, playerHide != null && playerHide.IsHidden);
    }

    public void TriggerStealthKill()
    {
        animator?.SetTrigger("StealthKill");
    }

    public void TriggerUse()
    {
        animator?.SetTrigger("Use");
    }

    public void TriggerDeath()
    {
        animator?.SetTrigger("Death");
    }
}
```

---

# 7) Cómo debe quedar el prefab PF_Player

Estructura recomendada:

```text
PF_Player
├── Visual
├── CameraRoot
│   └── MainCamera
├── GroundCheck
├── HoldPoint
└── FlashlightLight
```

---

# 8) Scripts que van en PF_Player

En el **objeto raíz** `PF_Player` añade:

* `CharacterController`
* `PlayerStamina`
* `PlayerMovement`
* `PlayerHide`
* `FlashlightController`
* `PlayerItemHolder`
* `PlayerAnimationBridge`

Luego, en bloques posteriores, también llevará:

* `InteractionSystem`
* `FootstepNoiseEmitter`
* `FootstepSurfaceAudio`

---

# 9) Referencias que debes asignar

## PlayerMovement

* `controller` → CharacterController del mismo objeto
* `groundCheck` → hijo `GroundCheck`
* `groundMask` → layer `Ground`

## PlayerHide

* `playerCamera` → `MainCamera`
* `cameraHiddenAnchor` → opcional, un punto/cámara objetivo para cuando estés escondido

## FlashlightController

* `flashlightLight` → hijo `FlashlightLight`
* `toggleAudio` → opcional

## PlayerItemHolder

* `holdPoint` → hijo `HoldPoint`

## PlayerAnimationBridge

* `movement` → `PlayerMovement`
* `animator` → Animator del player

---

# 10) Prueba rápida de este bloque

Antes de seguir, revisa esto:

## Movimiento

* camina con WASD
* corre con `Shift`
* salta con `Space`
* se agacha con `Ctrl`

## Stamina

* al correr baja
* al parar se recupera

## Linterna

* si empieza bloqueada, no hace nada con `F`
* al desbloquearla, puede encenderse y apagarse

## Escondite

Todavía no funcionará hasta que montemos `HideSpot`, pero `PlayerHide` ya debe compilar bien.

## Animación

* `Speed` cambia al moverte
* `Grounded` cambia al saltar
* `Crouching` cambia al agacharte

---

# 11) Dependencias de este bloque

Este bloque depende de:

* `HideSpot.cs` → te lo paso en el siguiente bloque
* `InteractionSystem.cs` → siguiente bloque
* `IInteractable.cs` → siguiente bloque

Así que si ves error por `HideSpot`, es normal hasta el siguiente mensaje.

---

# Siguiente bloque

En el siguiente mensaje te paso el **Bloque 3 — Interaction / Items**, con código completo de:

* `IInteractable.cs`
* `InteractionSystem.cs`
* `KeyPickup.cs`
* `KeycardPickup.cs`
* `FlashlightPickup.cs`
* `FusePickup.cs`
* `DoorInteract.cs`
* `KeycardDoor.cs`
* `HideSpot.cs`
* `PhoneInteract.cs`
* `MirrorInteract.cs`
* `WalkieTalkieInteract.cs`
* `ElevatorAccess.cs`
* `SavePointInteract.cs`
* `ThrowableItem.cs`

Seguimos con ese.
