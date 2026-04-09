# Arquitectura del sistema: WebAI Inference Throttling

> **Proyecto:** WebAI Inference Throttling  
> **Autor:** Fernando Flores Alvarado  
> **OWASP Project Leader (México)**  
> **Contacto:** fernando.alvarado@owasp.org  
> **Contacto:** fernandofa0306@gmail.com  

> ℹ️ **Note:** This document is written in Spanish. You can use your browser to translate it into English.
>  
> The Spanish version is preserved intentionally as part of the project's authorship and intellectual identity.

---

## El principio central

En lugar de correr el modelo en cada frame, la arquitectura desacopla el flujo
en cuatro capas independientes donde cada una tiene una responsabilidad clara
y se controla de forma autónoma.

La pregunta que guió el diseño no fue _"¿cómo hago que esto funcione?"_  
sino _"¿cómo hago que cada parte no sepa más de lo que necesita saber?"_

---

## Las 4 capas

| Capa | Módulo | Responsabilidad única |
|------|--------|-----------------------|
| 1 | `VideoController` | Flujo de cámara nativo sin intermediarios |
| 2 | `ModelController` + `ThrottleController` | Inferencia con throttling adaptativo |
| 3 | `VectorProcessor` | Decidir si hubo un cambio real |
| 4 | `CanvasRenderer` | Dibujar solo cuando hay cambios confirmados |

```
┌───────────────────────────────────────────────────┐
│             CAPA 1 — VideoController              │
│        Flujo directo desde getUserMedia()         │
│        Sin canvas intermedios, cero carga         │
└─────────────────────────┬─────────────────────────┘
                          │ frames
┌─────────────────────────▼─────────────────────────┐
│             CAPA 2 — ModelController              │
│       Inferencia con ThrottleController           │
│       Solo corre cada N ms (heurística CPU)       │
└─────────────────────────┬─────────────────────────┘
                          │ predicciones crudas
┌─────────────────────────▼─────────────────────────┐
│             CAPA 3 — VectorProcessor              │
│    Compara predicciones actuales vs anteriores    │
│    Decide si hay cambio real                      │
└─────────────────────────┬─────────────────────────┘
                          │ solo si hay cambios
┌─────────────────────────▼─────────────────────────┐
│             CAPA 4 — CanvasRenderer               │
│          Draw call solo cuando es necesario       │
│         Sin parpadeo, sin redraw innecesario      │
└───────────────────────────────────────────────────┘
```

---

## Capa 1 — VideoController

Gestión completa de la cámara. Esta capa no sabe que existe un modelo de ML.

- Acceso nativo vía `getUserMedia()` sin canvas intermedios
- Cambio de cámara en caliente
- Soporte de zoom y linterna donde el hardware lo permite
- Detección automática de capacidades reales del dispositivo
- Soporte de resoluciones: QVGA, VGA, HD, FullHD, 4K

---

## Capa 2 — ModelController + ThrottleController

Inferencia ML con throttling integrado. Esta capa no sabe que existe un canvas.

El `ThrottleController` es un módulo auxiliar que calibra el dispositivo con
runs de warmup y calcula el FPS objetivo según el costo real de inferencia
y la resolución de entrada. El ajuste es continuo: ±1 fps por ciclo para
evitar oscilaciones bruscas.

El `ModelController` ejecuta el loop de inferencia usando `requestAnimationFrame`
y solo corre el modelo cuando ha pasado el intervalo mínimo calculado por el
throttle. Filtra clases de interés configurables.

**Presupuesto de CPU:** ~200 ms/s dedicados a inferencia (~20% del CPU).

> ### Nota sobre autonomía
>
> La arquitectura de 4 capas es funcional y viable por sí sola.
> El `ThrottleController` representa **una solución particular** al problema
> de darle autonomía al sistema — no es la única forma posible.
> Cada desarrollador puede resolver la autonomía según su caso de uso.
>
> 👉 Historia completa de este módulo: [`evolution_throttle_controller.md`](./project-evolution/evolution_throttle_controller.md)

---

## Capa 3 — VectorProcessor

El guardián del render. Esta capa decide si lo que el modelo devolvió
representa un cambio real en el mundo, o si es ruido, reordenamiento
del array, o una detección momentánea.

La evolución de esta capa pasó por tres versiones:

| Versión | Técnica | Archivo de demo |
|---------|---------|-----------------|
| v1 | Comparación directa por índice | `v2-four-layers` |
| v2 | Comparación vectorial con umbral adaptado | `v3-vectordiff` |
| v3 | Tracking con IoU e identidad persistente | `v4-ioutracker` |

> 👉 Historia completa: [`evolution_vector_processor.md`](./project-evolution/evolution_vector_processor.md)

---

## Capa 4 — CanvasRenderer

Renderizado inteligente. Esta capa no sabe cuándo se llama — solo recibe
predicciones y las pinta. La decisión de cuándo llamarla la toma la Capa 3.

- Solo dibuja cuando `VectorProcessor` confirma cambios reales
- Soporte HiDPI automático via `devicePixelRatio`
- Colores por clase configurables
- Retroalimentación visual por estabilidad del objeto

---

## Por qué esta separación importa

**Cambiar cómo se dibuja** no requiere tocar la lógica de comparación.  
**Cambiar cómo se detectan cambios** no requiere tocar el renderizador.  
**Agregar otra salida** (logs, métricas, eventos) solo requiere cambiar el callback de la Capa 3.

Esto fue exactamente lo que hizo posible la evolución de la Capa 3 desde
comparación simple hasta tracking por IoU — sin tocar ninguna de las otras capas.

---

## El flujo completo de datos

```
Cámara → VideoController → ModelController → VectorProcessor → CanvasRenderer
          [stream nativo]    [inferencia]       [¿cambió?]        [dibuja]
```

Cada flecha es una responsabilidad. Cada módulo hace exactamente lo que su nombre dice.

---

## Documentos relacionados

| Archivo | Contenido |
|---------|-----------|
| [`project_history.md`](./project_history.md) | Origen y evolución completa del proyecto |
| [`evolution_3_to_4_layers.md`](./project-evolution/evolution_3_to_4_layers.md) | La separación de PredictionProcessor y CanvasRenderer |
| [`evolution_throttle_controller.md`](./project-evolution/evolution_throttle_controller.md) | Del código espagueti al ThrottleController encapsulado |
| [`evolution_vector_processor.md`](./project-evolution/evolution_vector_processor.md) | De comparación por índice a tracking por IoU |
| [`techniques.md`](./techniques.md) | Técnicas implementadas y parámetros configurables |
| [`performance.md`](./performance.md) | Estrategias de optimización y resultados |

---

**© 2025 Fernando Flores Alvarado — Todos los derechos reservados bajo las licencias indicadas.**  
Este contenido forma parte del proyecto **WebAI Inference Throttling** bajo esquema de licencias dual  
(*MIT* para código + *CC BY 4.0* para documentación).
