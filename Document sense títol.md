Perfecto. Eso le da “nervio” al juego: **latidos \+ respiración \+ viñeteado** cuando el enemigo está **inspeccionando TU escondite** (locker o bajo mesa). Y cuando termina, se apaga.

La forma más limpia: disparar **eventos** desde la inspección del enemigo y que el Player active/desactive los FX.

---

## **1\) UI: viñeteado (overlay) \+ CanvasGroup**

En `Canvas` (Game):

1. `UI > Image` a pantalla completa → `TensionVignette`  
2. Sprite: una viñeta (o un PNG negro con bordes)  
3. Color: negro, alpha 0 (o el que quieras)  
4. Añade **CanvasGroup** al `TensionVignette` (o a un panel contenedor)

---

## **2\) Sistema de eventos para inspección de escondites**

Crea `HideSpotEvents.cs`:

using System;  
using UnityEngine;

public static class HideSpotEvents  
{  
    // spotTransform \= el Transform que representa el spot del player (CurrentSpot)  
    // intensity 0..1  
    public static Action\<Transform, float\> OnThreatStart;  
    public static Action OnThreatEnd;

    public static void ThreatStart(Transform spotTransform, float intensity)  
        \=\> OnThreatStart?.Invoke(spotTransform, Mathf.Clamp01(intensity));

    public static void ThreatEnd()  
        \=\> OnThreatEnd?.Invoke();  
}

---

## **3\) FX del jugador: latidos \+ respiración \+ viñeteado (con “pulse”)**

Crea `PlayerTensionFX.cs` y ponlo en el Player:

using System.Collections;  
using UnityEngine;  
using UnityEngine.UI;

public class PlayerTensionFX : MonoBehaviour  
{  
    \[Header("Refs")\]  
    public PlayerHide playerHide;

    \[Header("UI Vignette")\]  
    public CanvasGroup vignetteGroup;     // CanvasGroup del overlay  
    public float vignetteMaxAlpha \= 0.55f;  
    public float vignetteFadeSpeed \= 6f;

    \[Header("Audio")\]  
    public AudioSource voiceSource;       // respiración  
    public AudioSource sfxSource;         // latidos  
    public AudioClip\[\] heartbeatClips;    // 1–3 clips en loop o one-shot repetido  
    public AudioClip breathTenseLoop;     // loop respiración tensa

    \[Header("Pulse")\]  
    public float pulseSpeed \= 2.2f;       // velocidad del “latido visual”  
    public float pulseAmount \= 0.12f;     // cuánto pulsa el alpha

    Transform threatenedSpot;  
    float targetIntensity;  
    bool active;

    Coroutine heartbeatRoutine;

    void Awake()  
    {  
        if (playerHide \== null) playerHide \= GetComponent\<PlayerHide\>();  
        if (vignetteGroup \!= null) vignetteGroup.alpha \= 0f;  
    }

    void OnEnable()  
    {  
        HideSpotEvents.OnThreatStart \+= HandleThreatStart;  
        HideSpotEvents.OnThreatEnd \+= HandleThreatEnd;  
    }

    void OnDisable()  
    {  
        HideSpotEvents.OnThreatStart \-= HandleThreatStart;  
        HideSpotEvents.OnThreatEnd \-= HandleThreatEnd;  
    }

    void Update()  
    {  
        // Solo activa si el player está realmente escondido en ese spot  
        bool shouldBeActive \=  
            active &&  
            playerHide \!= null &&  
            playerHide.IsHidden &&  
            playerHide.CurrentSpot \== threatenedSpot;

        float baseAlpha \= shouldBeActive ? vignetteMaxAlpha \* targetIntensity : 0f;

        // Pulso visual  
        if (shouldBeActive)  
            baseAlpha \+= Mathf.Sin(Time.unscaledTime \* pulseSpeed) \* pulseAmount;

        baseAlpha \= Mathf.Clamp01(baseAlpha);

        if (vignetteGroup \!= null)  
            vignetteGroup.alpha \= Mathf.Lerp(vignetteGroup.alpha, baseAlpha, Time.unscaledDeltaTime \* vignetteFadeSpeed);

        // Si dejó de aplicar (saliste del spot o dejó de estar escondido), parar audio  
        if (\!shouldBeActive)  
        {  
            StopAudio();  
        }  
        else  
        {  
            EnsureAudio();  
        }  
    }

    void HandleThreatStart(Transform spot, float intensity)  
    {  
        threatenedSpot \= spot;  
        targetIntensity \= Mathf.Clamp01(intensity);  
        active \= true;  
    }

    void HandleThreatEnd()  
    {  
        active \= false;  
        threatenedSpot \= null;  
        targetIntensity \= 0f;  
        StopAudio();  
    }

    void EnsureAudio()  
    {  
        // Respiración tensa (loop)  
        if (voiceSource \!= null && breathTenseLoop \!= null)  
        {  
            if (voiceSource.clip \!= breathTenseLoop || \!voiceSource.isPlaying)  
            {  
                voiceSource.clip \= breathTenseLoop;  
                voiceSource.loop \= true;  
                voiceSource.volume \= (AudioManager.I \!= null ? AudioManager.I.VoiceVol : 1f) \* Mathf.Lerp(0.4f, 1f, targetIntensity);  
                voiceSource.Play();  
            }  
        }

        // Latidos (simple: one-shot repetido)  
        if (heartbeatRoutine \== null && sfxSource \!= null && heartbeatClips \!= null && heartbeatClips.Length \> 0\)  
            heartbeatRoutine \= StartCoroutine(HeartbeatLoop());  
    }

    IEnumerator HeartbeatLoop()  
    {  
        while (true)  
        {  
            var clip \= heartbeatClips\[Random.Range(0, heartbeatClips.Length)\];  
            float bus \= (AudioManager.I \!= null ? AudioManager.I.SfxVol : 1f);  
            sfxSource.volume \= bus \* Mathf.Lerp(0.35f, 0.9f, targetIntensity);  
            sfxSource.pitch \= Mathf.Lerp(0.95f, 1.05f, targetIntensity);

            sfxSource.PlayOneShot(clip);

            // Intervalo: más intensidad \= más rápido  
            float interval \= Mathf.Lerp(0.85f, 0.45f, targetIntensity);  
            yield return new WaitForSecondsRealtime(interval);  
        }  
    }

    void StopAudio()  
    {  
        if (voiceSource \!= null && voiceSource.clip \== breathTenseLoop)  
        {  
            voiceSource.Stop();  
            voiceSource.clip \= null;  
        }

        if (heartbeatRoutine \!= null)  
        {  
            StopCoroutine(heartbeatRoutine);  
            heartbeatRoutine \= null;  
        }  
    }  
}

**Setup rápido**

* En Player:  
  * `PlayerHide`  
  * `PlayerTensionFX`  
  * `voiceSource` (respiración) y `sfxSource` (latidos)  
* En UI:  
  * `vignetteGroup` asignado al CanvasGroup del overlay  
* Clips:  
  * 1–3 heartbeat clips  
  * 1 loop de respiración tensa

---

## **4\) Disparar el evento cuando el enemigo revisa TU spot**

### **A) Bajo mesa/cama (EnemyInspectLow)**

En `EnemyInspectLow.OnInspectLowPeek()` (antes de chequear Game Over), añade:

HideSpotEvents.ThreatStart(currentSpot.hideAnchor, 0.9f);

Y al final de `OnInspectLowEnd()`:

HideSpotEvents.ThreatEnd();

Importante: `currentSpot.hideAnchor` debe ser el mismo Transform que el Player usa como `CurrentSpot`.

### **B) Locker (cuando el enemigo abre el armario)**

En `LockerHideSpot.EnemyInspectOpen()` añade al final:

HideSpotEvents.ThreatStart(transform, 1.0f);

Y en `EnemyInspectClose()` o `EnemyInspectEnd()`:

HideSpotEvents.ThreatEnd();

---

## **5\) Resultado**

* Si estás escondido y el enemigo empieza a revisar **ese** escondite:  
  * **aparece viñeteado** con pulso  
  * suenan **latidos \+ respiración tensa**  
* Si no estás escondido ahí, no pasa nada (no te “spoilea”)  
* Cuando termina la revisión, se apaga todo.

---

Si quieres subirlo otro nivel: que el sonido se “filtre” (low-pass) cuando estás dentro de un locker (como si oyeras desde dentro). Se hace con `AudioLowPassFilter` en el AudioListener o en los AudioSources del mundo cuando `PlayerHide.IsHidden` en locker. ¿Lo metemos?

