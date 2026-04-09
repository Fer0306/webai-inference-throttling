# Evolución de 3 Capas a 4 Capas: La Separación de PredictionProcessor y CanvasRenderer

> **Proyecto:** WebAI Inference Throttling  
> **Autor:** Fernando Flores Alvarado  
> **OWASP Project Leader (México)**  
> **Contacto:** fernando.alvarado@owasp.org  
> **Contacto:** fernandofa0306@gmail.com  

> ℹ️ **Note:** This document is written in Spanish. You can use your browser to translate it into English.
>  
> The Spanish version is preserved intentionally as part of the project's authorship and intellectual identity.

---

## Contexto

Este documento registra el momento arquitectural más importante del proyecto: la transición de una arquitectura de 3 capas a 4 capas. No fue un cambio menor — fue el punto donde el sistema dejó de tener una capa híbrida que hacía dos cosas a la vez, y cada responsabilidad encontró su propio módulo.

---

## La arquitectura de 3 capas

El código de 3 capas ya era un avance significativo respecto al código espagueti original. Tenía módulos separados, clases bien definidas, y una conexión limpia entre ellos:

| Capa | Módulo | Responsabilidad |
|------|--------|-----------------|
| 1 | `VideoController` | Flujo de cámara nativo |
| 2 | `ModelController` | Inferencia con throttling |
| 3 | `PredictionProcessor` | Comparar, decidir Y dibujar |

El problema estaba en la Capa 3. El módulo `PredictionProcessor` hacía **dos cosas**:

```js
// Versión 3 capas — PredictionProcessor tenía DOBLE responsabilidad
class PredictionProcessor {
  constructor(canvasElement) {        // ← recibía el canvas directamente
    this.canvas = canvasElement;      // ← guardaba referencia al canvas
    this.ctx = this.canvas.getContext('2d');
    this.lastPredictions = [];
  }

  process(predictions) {
    if (this.hasChanges(predictions)) {
      this.lastPredictions = predictions.map(p => ({...p}));
      this.draw(predictions);          // ← comparaba Y dibujaba en el mismo módulo
    }
  }

  draw(predictions) {
    // Lógica de renderizado mezclada con lógica de comparación
    const ctx = this.ctx;
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    // ...
  }
}
```

Responsabilidad 1: **comparar** predicciones actuales contra anteriores para detectar cambios reales.
Responsabilidad 2: **dibujar** en el canvas cuando hay cambios.

Estas dos cosas no pertenecen al mismo módulo. Comparar vectores es lógica de procesamiento. Dibujar es lógica de presentación. Mezclarlas significa que si mañana quieres cambiar cómo se dibujan las cajas, tienes que modificar el mismo módulo que gestiona el estado de las predicciones.

---

## El síntoma concreto: el canvas acoplado al procesador

La señal más clara del problema era el constructor:

```js
// 3 capas — el procesador dependía del canvas para existir
const processor = new PredictionProcessor(canvasElement);
```

Para instanciar un procesador de predicciones — que es lógica pura de comparación — necesitabas tener un canvas disponible. Si alguna vez quisieras usar el procesador sin canvas (para pruebas, para logs, para otro tipo de salida), no podías. El procesador y el renderizador eran inseparables.

Y en la conexión con la Capa 2:

```js
// 3 capas — ModelController enviaba predicciones directamente al processor+renderer
modelCtrl.onPredictions = (preds) => {
  processor.process(preds);  // process() internamente llamaba draw()
};
```

Una sola llamada hacía dos cosas. El flujo era opaco.

---

## La separación en 4 capas

La solución fue dividir la Capa 3 en dos módulos con responsabilidades únicas:

| Capa | Módulo | Responsabilidad única |
|------|--------|-----------------------|
| 1 | `VideoController` | Flujo de cámara nativo |
| 2 | `ModelController` | Inferencia con throttling |
| **3** | **`PredictionProcessor`** | **Solo comparar — detectar cambios reales** |
| **4** | **`CanvasRenderer`** | **Solo dibujar — renderizar cuando hay cambios** |

### El nuevo PredictionProcessor — solo compara

```js
// 4 capas — PredictionProcessor tiene UNA sola responsabilidad
class PredictionProcessor {
  constructor(onChangeCallback) {         // ← ya no recibe canvas
    if (typeof onChangeCallback !== "function") {
      throw new Error("Debes pasar una función callback para manejar cambios.");
    }
    this.onChange = onChangeCallback;     // ← recibe lo que debe hacer cuando hay cambio
    this.lastPredictions = [];
  }

  process(predictions) {
    if (this.hasChanges(predictions)) {
      this.lastPredictions = predictions.map(p => ({ ...p }));
      this.onChange(predictions);         // ← avisa al siguiente módulo, no dibuja él mismo
    }
  }
  // hasChanges() — solo lógica de comparación, sin ninguna referencia a canvas
}
```

El `PredictionProcessor` ya no sabe que existe un canvas. No sabe que hay un navegador. No sabe que hay pantalla. Solo sabe comparar vectores y avisar cuando algo cambió.

### El nuevo CanvasRenderer — solo dibuja

```js
// 4 capas — CanvasRenderer tiene UNA sola responsabilidad
class CanvasRenderer {
  constructor(canvasElement, videoElement) {
    this.canvas = canvasElement;
    this.video = videoElement;
    this.ctx = this.canvas.getContext("2d");
  }

  draw(predictions) {
    // Solo lógica de presentación — sin comparaciones, sin estado de predicciones
    const ctx = this.ctx;
    if (canvas.width !== this.video.videoWidth || ...) {
      canvas.width = this.video.videoWidth;
      canvas.height = this.video.videoHeight;
    }
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    predictions.forEach(pred => {
      // dibuja cajas y etiquetas
    });
  }
}
```

El `CanvasRenderer` no sabe cuándo se llama. No decide si hay cambios. Solo recibe predicciones y las pinta.

---

## La conexión entre los 4 módulos

Con la separación hecha, la conexión quedó explícita y legible:

```js
// 4 capas — el flujo es visible y cada módulo hace una sola cosa
const renderer  = new CanvasRenderer(canvasElement, videoElement);
const processor = new PredictionProcessor(preds => renderer.draw(preds));
//                                         ↑
//                     cuando hay cambios, avisa al renderer

modelCtrl.onPredictions = (preds) => {
  processor.process(preds);  // process() compara → si hay cambio → renderer.draw()
};
```

El flujo completo de datos es ahora:

```
Cámara → ModelController → PredictionProcessor → CanvasRenderer
  [stream]    [inferencia]      [¿cambió?]           [dibuja]
```

Cada flecha es una responsabilidad. Cada módulo hace exactamente lo que su nombre dice.

---

## Lo que cambió y lo que no

La lógica de comparación `hasChanges()` es **idéntica** en ambas versiones — no se tocó. Lo que cambió fue únicamente dónde vive esa lógica y a quién le pertenece el canvas.

```js
// hasChanges() — igual en 3 capas y en 4 capas
hasChanges(predictions) {
  if (predictions.length !== this.lastPredictions.length) return true;
  for (let i = 0; i < predictions.length; i++) {
    const curr = predictions[i];
    const prev = this.lastPredictions[i];
    if (!prev) return true;
    if (
      curr.class !== prev.class ||
      Math.abs(curr.score - prev.score) > 0.01 ||
      Math.abs(curr.bbox[0] - prev.bbox[0]) > 1 ||
      Math.abs(curr.bbox[1] - prev.bbox[1]) > 1 ||
      Math.abs(curr.bbox[2] - prev.bbox[2]) > 1 ||
      Math.abs(curr.bbox[3] - prev.bbox[3]) > 1
    ) return true;
  }
  return false;
}
```

Esta es la misma función en ambas versiones — la refactorización no cambió el algoritmo, solo lo reubicó correctamente.

---

## Tabla comparativa

| Aspecto | 3 Capas | 4 Capas |
|---|---|---|
| Número de módulos | 3 | 4 |
| `PredictionProcessor` constructor | Recibe `canvasElement` | Recibe `onChangeCallback` |
| Responsabilidad de comparación | `PredictionProcessor` | `PredictionProcessor` |
| Responsabilidad de dibujo | `PredictionProcessor` (mezclada) | `CanvasRenderer` (separada) |
| Acoplamiento entre lógica y presentación | Sí — inseparables | No — módulos independientes |
| Para cambiar cómo se dibuja | Modificar el procesador | Modificar solo el renderer |
| Para usar el procesador sin canvas | Imposible | Posible — recibe cualquier callback |
| Legibilidad del flujo de datos | Implícita | Explícita en la conexión |

---

## Por qué importa esta separación

La arquitectura de 4 capas no es más compleja que la de 3 — tiene un módulo más, pero cada módulo es más simple. La diferencia real es que:

**Cambiar cómo se dibuja** (colores, fuentes, efectos visuales) no requiere tocar la lógica de comparación de vectores.

**Cambiar cómo se detectan cambios** (tolerancias, algoritmos de matching) no requiere tocar el renderizador.

**Agregar otro tipo de salida** (logs, métricas, eventos externos) solo requiere cambiar el callback que recibe `PredictionProcessor` — sin tocar ninguna de las capas anteriores.

Esta es exactamente la base que hizo posible la evolución posterior hacia tracking por IoU (documentada en `evolution_vector_processor.md`) — porque cuando llegó el momento de mejorar la Capa 3, la Capa 4 no se vio afectada.

---

## Nota sobre VideoController

`VideoController` es idéntico en ambas versiones — la única diferencia es que en el código de 4 capas se añadió una validación defensiva en `init()`:

```js
// 4 capas — validación agregada al inicio de init()
async init(resolutionType = "HD", facingMode = "user") {
  if (!(this.videoElement instanceof HTMLVideoElement)) {
    console.warn("⚠️ Advertencia: El elemento proporcionado no es un <video>.");
    this._handleError(new Error("Debes pasar un elemento <video> válido."), "init()");
    return;
  }
  // ...
}
```

Un detalle menor de robustez — el módulo falla limpiamente con un mensaje claro en lugar de lanzar una excepción críptica.

---

**© 2025 Fernando Flores Alvarado — Todos los derechos reservados bajo las licencias indicadas.**  
Este contenido forma parte del proyecto **WebAI Inference Throttling** bajo esquema de licencias dual  
(*MIT* para código + *CC BY 4.0* para documentación).
