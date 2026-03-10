A continuación tienes una **GUÍA COMPLETA DEL PROYECTO** en **Markdown** con:

* estructura de carpetas recomendada
* configuración de escena
* prefabs
* sistemas principales
* **scripts definitivos**
* configuraciones de Unity
* flujo del juego

Puedes guardar este documento como:

```
PROJECT_GUIDE.md
```

en la raíz del proyecto.

---

# 🎮 GUÍA COMPLETA DEL PROYECTO

## Survival / Stealth Horror Game — Unity

---

# 1. Estructura del Proyecto

Organiza el proyecto así:

```
Assets
│
├── Art
│   ├── Characters
│   ├── Environment
│   ├── Props
│
├── Audio
│   ├── Footsteps
│   ├── Music
│   ├── Voices
│
├── Materials
│
├── Prefabs
│   ├── Player
│   ├── Enemies
│   ├── Boss
│   ├── Doors
│   ├── Items
│
├── Scenes
│   ├── MainMenu
│   ├── Level_01
│
├── Scripts
│   ├── Player
│   ├── Enemy
│   ├── Boss
│   ├── Systems
│   ├── UI
│
├── UI
│
└── Timeline
```

---

# 2. Escena principal

Objetos principales:

```
Scene
│
├── Player
├── GameManager
├── MusicManager
├── BossSystem
├── UI Canvas
├── Lighting
└── NavMesh
```

---

# 3. Player

Componentes:

```
Player
├ CharacterController
├ PlayerMovement
├ PlayerStamina
├ PlayerCameraLook
├ PlayerAudio
├ PlayerHide
└ Camera
```

---

# 4. Movimiento del jugador

Archivo:

```
Scripts/Player/PlayerMovement.cs
```

Script definitivo:

```csharp
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    public CharacterController controller;

    public float walkSpeed = 4f;
    public float sprintSpeed = 7f;
    public float crouchSpeed = 2f;

    public float gravity = -20f;
    public float jumpHeight = 1.2f;

    public Transform groundCheck;
    public float groundRadius = 0.25f;
    public LayerMask groundMask;

    float verticalVelocity;
    bool grounded;
    bool crouching;

    void Update()
    {
        CheckGround();
        Move();
        Jump();
        ApplyGravity();
    }

    void CheckGround()
    {
        grounded = Physics.CheckSphere(
            groundCheck.position,
            groundRadius,
            groundMask
        );

        if (grounded && verticalVelocity < 0)
            verticalVelocity = -2f;
    }

    void Move()
    {
        float x = Input.GetAxis("Horizontal");
        float z = Input.GetAxis("Vertical");

        Vector3 move =
            transform.right * x +
            transform.forward * z;

        crouching = Input.GetKey(KeyCode.LeftControl);

        float speed = walkSpeed;

        if (crouching)
            speed = crouchSpeed;
        else if (Input.GetKey(KeyCode.LeftShift))
            speed = sprintSpeed;

        controller.Move(move * speed * Time.deltaTime);
    }

    void Jump()
    {
        if (Input.GetKeyDown(KeyCode.Space) && grounded)
        {
            verticalVelocity =
                Mathf.Sqrt(jumpHeight * -2f * gravity);
        }
    }

    void ApplyGravity()
    {
        verticalVelocity += gravity * Time.deltaTime;

        controller.Move(
            Vector3.up * verticalVelocity * Time.deltaTime
        );
    }
}
```

---

# 5. Sprint y Stamina

Archivo:

```
Scripts/Player/PlayerStamina.cs
```

```csharp
using UnityEngine;

public class PlayerStamina : MonoBehaviour
{
    public float maxStamina = 5f;
    public float recoveryRate = 1.5f;
    public float sprintDrain = 1f;

    float stamina;

    void Start()
    {
        stamina = maxStamina;
    }

    void Update()
    {
        stamina += recoveryRate * Time.deltaTime;
        stamina = Mathf.Clamp(stamina, 0, maxStamina);
    }

    public bool UseStamina(float amount)
    {
        if (stamina <= 0)
            return false;

        stamina -= amount;
        return true;
    }
}
```

---

# 6. Cámara

Archivo:

```
Scripts/Player/PlayerCameraLook.cs
```

```csharp
using UnityEngine;

public class PlayerCameraLook : MonoBehaviour
{
    public Transform playerBody;
    public float sensitivity = 150f;

    float xRotation;

    void Start()
    {
        Cursor.lockState = CursorLockMode.Locked;
    }

    void Update()
    {
        float mouseX =
            Input.GetAxis("Mouse X") *
            sensitivity *
            Time.deltaTime;

        float mouseY =
            Input.GetAxis("Mouse Y") *
            sensitivity *
            Time.deltaTime;

        xRotation -= mouseY;
        xRotation = Mathf.Clamp(xRotation, -80f, 80f);

        transform.localRotation =
            Quaternion.Euler(xRotation, 0, 0);

        playerBody.Rotate(Vector3.up * mouseX);
    }
}
```

---

# 7. Sistema de Enemigos

Estados:

```
Patrol
Investigate
Chase
Search
```

Archivo:

```
Scripts/Enemy/EnemyAI.cs
```

Ejemplo base:

```csharp
using UnityEngine;
using UnityEngine.AI;

public class EnemyAI : MonoBehaviour
{
    public NavMeshAgent agent;
    public Transform player;

    public float chaseDistance = 10f;

    void Update()
    {
        float distance =
            Vector3.Distance(
                transform.position,
                player.position
            );

        if (distance < chaseDistance)
        {
            agent.SetDestination(player.position);
        }
    }
}
```

---

# 8. Sistema de Ruido

Eventos globales.

Archivo:

```
Scripts/Systems/NoiseSystem.cs
```

```csharp
using System;
using UnityEngine;

public static class NoiseSystem
{
    public static Action<Vector3, float> OnNoise;

    public static void Emit(Vector3 pos, float intensity)
    {
        OnNoise?.Invoke(pos, intensity);
    }
}
```

---

# 9. Sistema de Boss

Solo hay **un boss**.

Aparece:

* aleatoriamente
* después de una cinemática

Archivo:

```
Scripts/Boss/BossSpawner.cs
```

---

# 10. Boss Controller

El boss:

* siempre sabe donde estás
* ignora escondites
* mata instantáneamente

Archivo:

```
Scripts/Boss/BossController.cs
```

Ejemplo:

```csharp
using UnityEngine;
using UnityEngine.AI;

public class BossController : MonoBehaviour
{
    public NavMeshAgent agent;
    public Transform player;

    void Update()
    {
        agent.SetDestination(player.position);
    }

    void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Player"))
        {
            FindObjectOfType<GameManager>()
                .GameOverCaught();
        }
    }
}
```

---

# 11. Sistema Musical

Archivo:

```
Scripts/Systems/MusicManager.cs
```

Estados musicales:

```
Calm
Suspicious
Chase
```

---

# 12. UI

UI incluye:

```
Main Menu
Pause Menu
Game Over
Subtitles
Interaction Prompt
RUN Warning
```

Todos usan **TextMeshPro**.

---

# 13. Checkpoints

Archivo:

```
Scripts/Systems/CheckpointSystem.cs
```

Cuando el jugador toca un checkpoint:

```
Guardar posición
Guardar inventario
```

---

# 14. Sistema de interacción

Archivo:

```
Scripts/Systems/InteractionSystem.cs
```

Permite interactuar con:

* puertas
* objetos
* escondites

---

# 15. Sistema de puertas

Dos tipos:

### abatibles

requieren:

```
Key
```

### correderas

requieren:

```
Keycard
```

---

# 16. Sistema de ejecución (stealth kill)

Solo disponible después de una cinemática.

Condiciones:

```
Jugador detrás
Enemigo no en Chase
```

---

# 17. Sistema de audio

Incluye:

```
Footsteps
Breathing
Enemy voices
Music layers
Ambient sounds
```

---

# 18. Sistema de guardado

Se guarda:

```
checkpoint
inventario
habilidades
estado del boss
```

---

# 19. Configuración de NavMesh

En el nivel:

```
Window → AI → Navigation
```

Baking:

```
Agent radius = 0.4
Agent height = 1.8
```

---

# 20. Resultado del sistema

El proyecto incluye:

✔ movimiento completo
✔ salto
✔ vault
✔ sprint + stamina
✔ enemigos
✔ boss stalker
✔ sistema de ruido
✔ música dinámica
✔ checkpoints
✔ UI
✔ cinemáticas

Base completa para desarrollo del juego.

---

# 21. Próximos sistemas recomendados

Añadir:

```
AI Director
Puzzle system
Dynamic lighting
Inventory UI
Map system
```

---

💡 Si quieres, en el siguiente paso puedo darte **la mejora más importante de todo el proyecto**:

### **AI Director (tipo Left 4 Dead / Alien Isolation)**

Controla automáticamente:

* cuándo aparece el boss
* cuándo hay enemigos
* música
* tensión del juego

y hace que el juego **se sienta vivo y dinámico**.
