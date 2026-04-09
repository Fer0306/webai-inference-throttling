# Evolución de la Capa 3 y Capa 4: VectorProcessor y CanvasRenderer

> **Proyecto:** WebAI Inference Throttling  
> **Autor:** Fernando Flores Alvarado  
> **OWASP Project Leader (México)**  
> **Contacto:** fernando.alvarado@owasp.org  
> **Contacto:** fernandofa0306@gmail.com  

> ℹ️ **Note:** This document is written in Spanish. You can use your browser to translate it into English.
>  
> The Spanish version is preserved intentionally as part of the project's authorship and intellectual identity.

---

## Contexto arquitectural

El sistema opera en 4 capas independientes:

| Capa | Módulo | Responsabilidad |
|------|--------|-----------------|
| 1 | `VideoController` | Flujo de cámara nativo sin intermediarios |
| 2 | `ModelController` + `ThrottleController` | Inferencia con throttling adaptativo |
| **3** | **`VectorProcessor`** | **Interpretación y tracking de predicciones** |
| **4** | **`CanvasRenderer`** | **Renderizado inteligente (solo cuando hay cambios)** |

Este documento cubre la evolución de las Capas 3 y 4, que son las que procesan e interpretan lo que el modelo devuelve. La Capa 2 (throttling) se documenta por separado en `evolution_throttle_controller.md`.

---

## El problema que resolvía la Versión 1

La primera versión del `VectorProcessor` ya era un avance importante respecto al código espagueti original: introducía la idea de **no redibujar si nada cambió**. Eso eliminaba draw calls innecesarios y era el núcleo correcto de la Capa 3.

```js
// Versión 1 — comparación simple por índice
_hasChanges(preds) {
  if (preds.length !== this.lastVectors.length) return true;

  for (let i = 0; i < preds.length; i++) {
    const curr = preds[i];
    const prev = this.lastVectors[i]; // ← el problema está aquí

    if (curr.class !== prev.class) return true;
    if (Math.abs(curr.score - prev.score) > 0.05) return true;
    for (let j = 0; j < 4; j++) {
      if (Math.abs(curr.bbox[j] - prev.bbox[j]) > 8) return true;
    }
  }
  return false;
}
```

La lógica asumía que `preds[0]` del frame actual era el mismo objeto que `preds[0]` del frame anterior. Eso funciona bien cuando solo hay un objeto en pantalla, pero falla en cuanto hay dos o más.

---

## El problema implícito: identidad por posición de array

Imagina este escenario con dos objetos en pantalla:

```
Frame N:    modelo devuelve → [ botella, persona ]
Frame N+1:  modelo devuelve → [ persona, botella ]  ← mismo orden en canvas, distinto en array
```

El `VectorProcessor` v1 comparaba índice con índice:
- `preds[0]` (persona) vs `lastVectors[0]` (botella) → clase diferente → **cambio detectado**
- `preds[1]` (botella) vs `lastVectors[1]` (persona) → clase diferente → **cambio detectado**

Resultado: **dos falsos positivos de cambio** aunque ningún objeto se movió. El `CanvasRenderer` redibujaba innecesariamente, negando el beneficio central de la arquitectura.

El modelo COCO-SSD no garantiza orden consistente en su output. La comparación por índice era frágil por diseño.

---

## La solución: tracking por identidad con IoU

La Versión 2 introduce un cambio de paradigma en la Capa 3. En lugar de comparar arrays posición a posición, el módulo mantiene un **estado persistente de objetos con identidad propia**:

```js
// Versión 2 — cada objeto tiene un ID que persiste entre frames
this.objects = []; // estado del mundo
this.nextId  = 1;  // contador de identidades
```

Ahora la botella es `#1` y la persona es `#2`. Sin importar en qué posición del array las devuelva el modelo en el siguiente frame, el tracker las reconoce por su ubicación en el canvas — no por su posición en el array.

### El algoritmo de matching

Para cada predicción nueva, el módulo busca el objeto existente que mejor corresponde:

**Paso 1 — Matching primario por IoU (Intersection over Union):**

```js
_iou(a, b) {
  const x1 = Math.max(a[0], b[0]);
  const y1 = Math.max(a[1], b[1]);
  const x2 = Math.min(a[0] + a[2], b[0] + b[2]);
  const y2 = Math.min(a[1] + a[3], b[1] + b[3]);
  const inter = Math.max(0, x2 - x1) * Math.max(0, y2 - y1);
  const union = a[2] * a[3] + b[2] * b[3] - inter;
  return union === 0 ? 0 : inter / union;
}
```

IoU mide qué porcentaje del área combinada de dos bounding boxes se superpone. Un valor de `1.0` significa superposición perfecta; `0.0` significa que no se tocan. Si el IoU entre una predicción nueva y un objeto conocido supera `IOU_THRESHOLD = 0.4`, se considera el mismo objeto.

**Paso 2 — Fallback por distancia euclidiana entre centros:**

```js
_centerDistance(a, b) {
  const dx = (a[0] + a[2] / 2) - (b[0] + b[2] / 2);
  const dy = (a[1] + a[3] / 2) - (b[1] + b[3] / 2);
  return Math.sqrt(dx * dx + dy * dy);
}
```

Si IoU falla (el objeto se movió tanto que los bboxes ya no se solapan), se intenta el match por proximidad de centros. Si la distancia es menor a `DIST_THRESHOLD = 80px`, se acepta como el mismo objeto.

---

## Por qué estos umbrales específicos

Los valores no son arbitrarios — son consecuencia directa del throttling de la Capa 2.

Con throttling activo, el sistema puede estar corriendo a **3-5 FPS reales**. Entre un frame y el siguiente pueden pasar 200-300ms. En ese tiempo, un objeto en movimiento puede haberse desplazado significativamente en el canvas. Esto hace que:

- **`IOU_THRESHOLD = 0.4`** en lugar de 0.5 — más permisivo porque con frames lentos el solapamiento entre posiciones consecutivas es menor.
- **`DIST_THRESHOLD = 80px`** en lugar de 50 — más tolerante al movimiento real entre frames separados.
- **`MAX_AGE = 3`** en lugar de 10 — con FPS bajos, 10 frames son varios segundos. Un objeto que desapareció debe eliminarse rápido para que el sistema no muestre "fantasmas". Con 3 frames, el objeto desaparece en menos de 1 segundo.

Si el throttling fuera más agresivo o el dispositivo más lento, estos valores serían candidatos a autoconfiguración futura.

---

## Las variables nuevas de la Capa 3

```js
this.IOU_THRESHOLD  = 0.4   // solapamiento mínimo para considerar "mismo objeto"
this.DIST_THRESHOLD = 80    // distancia máxima entre centros como fallback de matching
this.MIN_SCORE      = 0.5   // confianza mínima para procesar una predicción
this.MAX_AGE        = 3     // frames sin detección antes de eliminar el objeto
// implícita en cada objeto:
obj.stableFrames            // cuántos frames consecutivos ha sido detectado este objeto
```

`stableFrames` es la variable más importante conceptualmente. Un objeto recién aparecido tiene `stableFrames = 1` — podría ser ruido del modelo, una detección falsa, una sombra. Solo cuando llega a `stableFrames >= 3` se considera un objeto **confirmado**.

---

## El impacto en la Capa 4: CanvasRenderer

La Capa 4 no necesitó cambios estructurales — pero sí aprovechó la información nueva que ahora llega de la Capa 3. Esto es la arquitectura funcionando correctamente: un módulo mejora y el siguiente aprovecha esa mejora sin acoplarse.

### Renderizado visual según estabilidad

```js
// Versión 1 — todos los objetos se dibujan igual
ctx.strokeStyle = color;
ctx.lineWidth   = 2;

// Versión 2 — el dibujo refleja la madurez del objeto
const isStable = obj.stableFrames >= 3;
ctx.strokeStyle = isStable ? color : '#888';   // gris si recién apareció
ctx.lineWidth   = isStable ? 2.5 : 1;          // delgado si no está confirmado
```

Un objeto recién detectado aparece en gris con línea fina. Si se confirma en los siguientes frames, adopta su color completo con línea más gruesa. El usuario ve exactamente qué tan confiable es cada detección.

### Label con identidad persistente

```js
// Versión 1
const label = `${pred.class}  ${(pred.score * 100).toFixed(1)}%`;

// Versión 2 — incluye el ID del tracker
const label = `#${obj.id} ${obj.class} (${(obj.score * 100).toFixed(0)}%)`;
```

Esto fue posible porque la Capa 3 ahora asigna y mantiene identidades. La Capa 4 simplemente las muestra.

### Indicador de punto de estabilidad

```js
// Punto visual en la esquina del bbox cuando el objeto está confirmado
if (isStable) {
  ctx.fillStyle = color;
  ctx.beginPath();
  ctx.arc(x + w - 6, y + 6, 3, 0, Math.PI * 2);
  ctx.fill();
}
```

Un detalle pequeño con valor real: el punto confirma visualmente que el objeto superó el umbral de estabilidad.

---

## Flujo completo de un objeto nuevo

```
Frame 1:  modelo detecta botella
          → VectorProcessor: no hay match → crea objeto #3, stableFrames=1
          → CanvasRenderer: dibuja en gris, línea fina (no confirmado)

Frame 2:  modelo detecta botella en posición similar
          → VectorProcessor: IoU > 0.4 → match con #3, stableFrames=2
          → CanvasRenderer: sigue en gris (aún no confirmado)

Frame 3:  modelo detecta botella en posición similar
          → VectorProcessor: IoU > 0.4 → match con #3, stableFrames=3
          → CanvasRenderer: cambia a color verde, línea gruesa, punto de confirmación ✓

Frame 4-N: objeto sigue detectado → se mantiene confirmado

Frame X:  modelo no detecta la botella
          → VectorProcessor: obj.updated=false → obj.age++ (age=1)
          → CanvasRenderer: objeto sigue visible (todavía dentro de MAX_AGE=3)

Frame X+3: age supera MAX_AGE
           → VectorProcessor: objeto eliminado del estado
           → CanvasRenderer: desaparece del canvas
```

---

## Tabla comparativa

| Aspecto | Versión 1 | Versión 2 |
|---|---|---|
| Método de comparación | Por índice del array | Por identidad (IoU + distancia) |
| Identidad de objetos | Ninguna (anónimos) | ID persistente entre frames |
| Tolerancia al cambio de orden | Ninguna (falsos positivos) | Total (orden del array no importa) |
| Objetos "fantasma" | No aplica | Controlados por MAX_AGE |
| Detecciones falsas | Se procesan igual | Filtradas por MIN_SCORE y stableFrames |
| Feedback visual de confianza | No existe | Gris→color según stableFrames |
| Información en label | clase + score | ID + clase + score |
| Dependencia del throttling | Tolerancias fijas | Tolerancias ajustadas al FPS real |

---

## Lo que sigue

Las variables `IOU_THRESHOLD`, `DIST_THRESHOLD`, `MIN_SCORE` y `MAX_AGE` actualmente tienen valores fijos definidos en el constructor. La siguiente etapa natural es que el `ThrottleController` o un módulo de autoconfiguración las ajuste dinámicamente según el FPS real medido del dispositivo — completando el ciclo de autonomía de la arquitectura.

---

**© 2025 Fernando Flores Alvarado — Todos los derechos reservados bajo las licencias indicadas.**  
Este contenido forma parte del proyecto **WebAI Inference Throttling** bajo esquema de licencias dual  
(*MIT* para código + *CC BY 4.0* para documentación).
