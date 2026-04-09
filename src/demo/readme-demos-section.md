## 🎮 Demos Interactivos

> **Proyecto:** WebAI Inference Throttling  
> **Autor:** Fernando Flores Alvarado  
> **OWASP Project Leader (México)**  
> **Contacto:** fernando.alvarado@owasp.org  
> **Contacto:** fernandofa0306@gmail.com  

> ℹ️ **Note:** This document is written in Spanish. You can use your browser to translate it into English.
>  
> The Spanish version is preserved intentionally as part of the project's authorship and intellectual identity.

---

Esta sección muestra la evolución progresiva de la arquitectura modular de **WebAI Inference Throttling**. Cada demo agrega una capa de inteligencia sobre el anterior, manteniendo la misma interfaz visual para que la comparación sea directa.

> **Todos los demos funcionan directamente en el navegador**, sin instalación, sin servidor, sin dependencias externas. Solo cámara web y un navegador moderno.

---

### Demo 1 — 3 Módulos

**[▶ Ver Demo 1](./v1-three-layers/index.html)**

Arquitectura inicial de 3 capas. El throttling es **manual**: el usuario selecciona la cantidad de FPS desde un control en la interfaz. El renderizado de bounding boxes usa `PredictionProcessor` con comparación directa frame a frame.

| Módulo | Responsabilidad |
|---|---|
| `VideoController` | Gestión de cámara, zoom, linterna, cambio de dispositivo |
| `ModelController` | Inferencia con coco-ssd, loop por `requestAnimationFrame` |
| `PredictionProcessor` | Comparación frame a frame y render directo en canvas |

**Características:**
- ✅ Throttling manual por selección de FPS (1–10)
- ✅ Filtrado de clases configurable (`bottle`, `cup`, `person`)
- ✅ Soporte de zoom y linterna (según dispositivo)
- ✅ Responsivo para móvil, tablet y escritorio

---

### Demo 2 — 4 Módulos

**[▶ Ver Demo 2](./v2-four-layers/index.html)**


En esta versión se introduce una **separación explícita entre procesamiento y renderizado**, agregando el módulo CanvasRenderer. Esto permite desacoplar la lógica de inferencia del dibujo en pantalla, mejorando la mantenibilidad y preparando la arquitectura para optimizaciones posteriores.

El throttling sigue siendo **manual**, pero ahora el pipeline está estructurado para evolucionar hacia un control más avanzado del flujo de inferencia.

| Módulo                | Responsabilidad                                                |
| --------------------- | -------------------------------------------------------------- |
| `VideoController`     | Gestión de cámara, dispositivos, zoom y linterna               |
| `ModelController`     | Inferencia con coco-ssd usando `requestAnimationFrame`         |
| `PredictionProcessor` | Detección de cambios entre frames (evita renders innecesarios) |
| `CanvasRenderer`      | Render desacoplado en canvas basado en predicciones            |

**Características:**

- ✅ **Separación real de responsabilidades** → el procesamiento (`PredictionProcessor`) deja de estar acoplado al render
- ✅ **Pipeline explícito de datos** → inferencia → procesamiento → render (`Model → Processor → Renderer`)
- ✅ **Render condicionado por cambios** → se evita trabajo gráfico innecesario cuando el estado no cambia
- ✅ **Primer paso hacia optimización** → la arquitectura ya permite introducir throttling inteligente sin refactorizar
- ✅ **Modularidad funcional** → cada componente puede evolucionar o reemplazarse de forma independiente

---

### Demo 3 — Vector Diff · 4 Módulos

**[▶ Ver Demo 3](./v3-vectordiff/index.html)**

Se introduce `ThrottleController`, que **calibra el dispositivo automáticamente** mediante 3 warmup runs al inicio y ajusta el FPS objetivo ±1 por ciclo según el presupuesto de CPU asignado. El `VectorProcessor` reemplaza a `PredictionProcessor` y **evita redraws innecesarios** comparando el array de predicciones por índice entre frames consecutivos.

| Módulo | Responsabilidad |
|---|---|
| `VideoController` | Gestión de cámara, zoom, linterna, cambio de dispositivo |
| `ThrottleController` | Calibración automática de FPS, presupuesto de CPU configurable |
| `ModelController` | Inferencia + throttle integrado por tiempo (no por frame count) |
| `VectorProcessor` | Diff por índice, elimina draw calls cuando el estado no cambia |
| `CanvasRenderer` | Render HiDPI con soporte `devicePixelRatio`, colores por clase |

**Características nuevas respecto a Demo 1:**
- ✅ Throttling **automático** — calibración en warmup de 3 inferencias reales
- ✅ Ajuste dinámico continuo: el FPS sube o baja según el rendimiento actual
- ✅ Select de FPS como **indicador de solo lectura** (muestra el valor calibrado)
- ✅ `VectorProcessor` evita redraws cuando las predicciones no cambian
- ✅ `CanvasRenderer` con soporte HiDPI (`devicePixelRatio`)
- ✅ Loading overlay con mensajes de progreso durante calibración
- ✅ Pausa automática por inactividad (25 segundos sin cambios)
- ✅ Pausa automática cuando la pestaña pierde el foco (`visibilitychange`)

---

### Demo 4 — IoU Tracker · 4 Módulos

**[▶ Ver Demo 4](./v4-ioutracker/index.html)**

El `VectorProcessor` evoluciona a un **Object Tracker** real. Cada objeto detectado recibe una **identidad persistente** (`#id`) que se mantiene entre frames. El matching usa **Intersection over Union (IoU)** como criterio primario, con fallback por **distancia euclidiana de centros**. Los objetos no vistos envejecen y son eliminados al superar `MAX_AGE` frames.

| Módulo | Responsabilidad |
|---|---|
| `VideoController` | Gestión de cámara, zoom, linterna, cambio de dispositivo |
| `ThrottleController` | Calibración automática de FPS, presupuesto de CPU configurable |
| `ModelController` | Inferencia + throttle integrado por tiempo |
| `VectorProcessor (IoU)` | Tracker con identidad persistente, matching IoU + centroide |
| `CanvasRenderer` | Render con `#id`, indicador de estabilidad, HiDPI |

**Características nuevas respecto a Demo 2:**
- ✅ **Identidad persistente por objeto** — cada detección tiene un `#id` que no cambia entre frames
- ✅ **Matching por IoU** — criterio estándar en computer vision para asociar detecciones entre frames
- ✅ **Fallback por distancia euclidiana de centros** — para casos de movimiento rápido o baja superposición
- ✅ **`stableFrames`** — contador por objeto: el render diferencia visualmente objetos nuevos vs confirmados
- ✅ **`MAX_AGE`** — los objetos no vistos envejecen y desaparecen limpiamente (sin parpadeo)
- ✅ **`MIN_SCORE`** — filtro de confianza mínima antes del tracking (evita trackear detecciones ruidosas)
- ✅ Label enriquecida: `#id clase (score%)`
- ✅ Punto visual de estabilidad en la esquina del bounding box cuando `stableFrames >= 3`

---

### Comparativa de los 4 Demos

| Característica | Demo 1 | Demo 2 | Demo 3 | Demo 4 |
|---|:---:|:---:|:---:|:---:|
| Módulos | 3 | 4 | 5 | 5 |
| Throttling | Manual | Manual | AUTO | AUTO |
| Calibración warmup | ✗ | ✗ | ✅ | ✅ |
| Ajuste dinámico de FPS | ✗ | ✗ | ✅ | ✅ |
| Pausa por inactividad | ✗ | ✗ | ✅ | ✅ |
| Pausa por visibilidad | ✗ | ✗ | ✅ | ✅ |
| HiDPI canvas | ✗ | ✅ | ✅ | ✅ |
| Evita redraws innecesarios | ✗ | ✅ | ✅ | ✅ |
| Identidad persistente por objeto | ✗ | ✗ | ✗ | ✅ |
| Matching por IoU | ✗ | ✗ | ✗ | ✅ |
| Fallback por centroide | ✗ | ✗ | ✗ | ✅ |
| stableFrames visual | ✗ | ✗ | ✗ | ✅ |
| MAX_AGE / envejecimiento | ✗ | ✗ | ✗ | ✅ |

---

### Arquitectura del pipeline (Demo 4)

```
VideoController → ThrottleController → ModelController → VectorProcessor (IoU) → CanvasRenderer
```

Cada módulo es **independiente y reemplazable** sin modificar el resto del pipeline. Esta es la propiedad central del diseño: la arquitectura modular permite evolucionar cada capa de forma aislada, como se demuestra en la progresión de los 4 demos.

---

**© 2025 Fernando Flores Alvarado — Todos los derechos reservados bajo las licencias indicadas.**  
Este contenido forma parte del proyecto **WebAI Inference Throttling** bajo esquema de licencias dual  
(*MIT* para código + *CC BY 4.0* para documentación).
