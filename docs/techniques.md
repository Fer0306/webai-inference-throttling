# Técnicas implementadas: WebAI Inference Throttling

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

Este documento describe las técnicas implementadas en la Capa 3 (`VectorProcessor`)
del sistema, los parámetros configurables, y las guías de afinación por hardware.

La Capa 3 es la que decide si lo que el modelo devolvió representa un cambio
real — o si es ruido, reordenamiento del array, o una detección momentánea.
Es el guardián del render.

---

## Evolución de la Capa 3

### Versión 1 — Comparación directa por índice
**Demo:** `v2-four-layers`

Compara las predicciones del frame actual contra las del frame anterior
**por posición en el array**. El objeto en `preds[0]` se compara contra
`lastVectors[0]`, y así sucesivamente.

Funciona bien con un solo objeto en pantalla. Falla cuando hay dos o más,
porque el modelo no garantiza orden consistente en su output.

**Limitaciones:**
- Dependiente del orden que devuelve el modelo
- Sin identidad de objeto entre frames
- Falsos cambios por reordenamiento del array
- Parpadeo visual en escenas estáticas con múltiples objetos

### Versión 2 — Comparación vectorial con umbral adaptado
**Demo:** `v3-vectordiff`

Mismo principio que v1 pero con tolerancias más generosas y un callback
`onStable` cuando el estado no cambia. Más robusto para hardware lento
donde el movimiento entre frames es mayor.

### Versión 3 — Tracking con IoU e identidad persistente
**Demo:** `v4-ioutracker`

Reemplaza la comparación por índice con un sistema de tracking temporal.
La pregunta ya no es _"¿cambió el arreglo?"_ sino _"¿este objeto sigue
existiendo en el tiempo?"_

Cada objeto recibe un `id` único que persiste entre frames. El matching
usa **IoU** como criterio primario y **distancia euclidiana entre centros**
como fallback.

---

## IoU — Intersection over Union

La métrica central del tracker. Mide qué porcentaje del área combinada
de dos bounding boxes se superpone:

```
         Área de intersección
IoU =  ─────────────────────────
            Área de unión
```

- `IoU = 1.0` → superposición perfecta (mismo objeto, no se movió)
- `IoU = 0.0` → sin solapamiento (objetos en posiciones completamente distintas)
- `IoU > 0.4` → umbral para considerar "mismo objeto" en este sistema

---

## Fallback — Distancia euclidiana entre centros

Cuando el IoU es insuficiente (el objeto se movió tanto que los bboxes
ya no se solapan), se aplica un segundo criterio:

```
distancia = √( (cx₁ - cx₂)² + (cy₁ - cy₂)² )
```

Si la distancia entre centros es menor a `DIST_THRESHOLD` y son de la
misma clase, se considera el mismo objeto.

---

## Parámetros configurables del VectorProcessor

```js
const vectorProcessor = new VectorProcessor({
  iouThreshold:  0.4,  // superposición mínima para matching primario
  distThreshold: 80,   // distancia máxima entre centros (fallback, en px)
  minScore:      0.5,  // score mínimo para procesar una predicción
  maxAge:        3,    // frames sin detección antes de eliminar el objeto
});
```

| Parámetro | Default | Descripción |
|-----------|---------|-------------|
| `iouThreshold` | `0.4` | Superposición mínima para considerar mismo objeto |
| `distThreshold` | `80` | Distancia máxima entre centros como fallback de matching |
| `minScore` | `0.5` | Score mínimo de confianza para aceptar una detección |
| `maxAge` | `3` | Frames de tolerancia antes de eliminar un objeto no visto |

---

## Afinación por hardware

### La fórmula del tiempo visible de caja fantasma

`maxAge` se mide en **frames de inferencia**, no en segundos. En dispositivos
lentos, cada frame tarda mucho más, por lo que un `maxAge` alto produce
cajas fantasma visibles durante muchos segundos reales:

```
tiempo visible ≈ maxAge × (ms por inferencia) / 1000

Ejemplo CPU lenta:  10 × 1500ms / 1000 = ~15 segundos  ← problema
Ejemplo CPU lenta:   3 × 1500ms / 1000 = ~4.5 segundos ← correcto
Ejemplo CPU rápida: 10 × 150ms  / 1000 = ~1.5 segundos ← correcto
```

### Perfiles recomendados según hardware

| Hardware | `maxAge` | `distThreshold` | `iouThreshold` | Resultado esperado |
|----------|----------|-----------------|----------------|--------------------|
| CPU lenta (2 núcleos, ~1500ms/inferencia) | `3` | `80` | `0.4` | Caja fantasma ~4.5s |
| CPU media (4 núcleos, ~500ms/inferencia) | `5` | `60` | `0.5` | Caja fantasma ~2.5s |
| CPU rápida (8+ núcleos, ~150ms/inferencia) | `10` | `50` | `0.5` | Caja fantasma ~1.5s |

**Regla general:**
- CPU lenta → bajar `maxAge`, subir `distThreshold`, bajar `iouThreshold`
- CPU rápida → subir `maxAge`, bajar `distThreshold`, subir `iouThreshold`

---

## Afinación de tolerancias en VectorProcessor v1

Para la versión 1 (comparación por índice), las tolerancias de bbox y score
también deben ajustarse según el hardware:

En CPUs lentas, el tiempo entre frames es grande — el objeto puede haberse
movido bastante físicamente, haciendo que tolerancias estrictas generen
falsos cambios y redraws innecesarios.

| Hardware | Tolerancia bbox | Tolerancia score | Efecto |
|----------|----------------|------------------|--------|
| CPU lenta (~1500ms/inferencia) | `8px` | `0.05` | Menos redraws, más estable |
| CPU media (~500ms/inferencia) | `4px` | `0.03` | Balance |
| CPU rápida (~150ms/inferencia) | `1px` | `0.01` | Máxima precisión |

**Regla general:**
- CPU lenta → subir ambas tolerancias (menos sensible, menos redraws)
- CPU rápida → bajar ambas tolerancias (más preciso)

---

## Retroalimentación visual por estabilidad

El `CanvasRenderer` aprovecha `stableFrames` para dar retroalimentación
visual al usuario sobre qué tan confiable es cada detección:

| Estado | `stableFrames` | Visual |
|--------|---------------|--------|
| Objeto recién detectado | `< 3` | Caja delgada, color gris |
| Objeto confirmado | `>= 3` | Caja gruesa, color completo + punto indicador |

Un objeto recién aparecido podría ser ruido del modelo, una detección
falsa, una sombra. Solo cuando llega a `stableFrames >= 3` se considera
un objeto confirmado.

---

## Técnicas implementadas — resumen

### ✅ Implementado

| Técnica | Detalle | Archivo |
|---------|---------|---------|
| Tolerancia de posición | Umbral configurable por pixel (1px / 8px / 2px) | todos |
| Tolerancia de score | Umbral mínimo de confianza (0.01 / 0.05 / 0.50) | todos |
| Comparación por índice | Detección de cambios comparando frame anterior vs actual | v2-four-layers |
| Throttle manual | Configurable por el usuario vía selector de FPS | v2-four-layers |
| Comparación por índice | Detección de cambios comparando frame anterior vs actual | v3-vectordiff |
| Comparación vectorial con umbral | Tolerancias adaptadas al hardware | v3-vectordiff |
| Control de frecuencia (Throttle) | Calibración automática + ajuste dinámico ±1 fps según CPU | v3-vectordiff |
| Control de frecuencia (Throttle) | Calibración automática + ajuste dinámico ±1 fps según CPU | v4-ioutracker |
| Matching independiente del orden | Asociación por IoU en lugar de posición en el array | v4-ioutracker |
| Reducción de falsos positivos | Score mínimo + contador de estabilidad por objeto (stableFrames ≥ 3) | v4-ioutracker |
| Anti-parpadeo básico | Envejecimiento de objetos no vistos (MAX_AGE = 3 frames) | v4-ioutracker |
| Tracking con identidad persistente | ID único por objeto entre frames via IoU + distancia de centroides | v4-ioutracker |

### 🔧 Identificado — pendiente de implementar

| Técnica | Detalle |
|---------|---------|
| Validación por tamaño de objeto | Filtrar detecciones por área mínima y máxima del bbox |
| Suavizado de coordenadas | Interpolación o promedio móvil entre frames para eliminar saltos bruscos |
| NMS manual | Non-Maximum Suppression para eliminar detecciones duplicadas superpuestas |
| Activación por detección de movimiento (pixel wakeup) | Reactivación automática por detección de movimiento en píxeles — referencia: `src/history/phase3-detection/p3g-spaghetti-pixel-wakeup.html` |

### 🔭 Fuera del alcance del proyecto (Roadmap)

Técnicas avanzadas que podrían aplicarse en versiones futuras:

- **Kalman Filter** — predicción de trayectoria entre frames
- **Hungarian Algorithm** — matching óptimo multi-objeto
- **DeepSORT** — re-identificación por apariencia visual
- **Modelos ligeros alternativos** — MobileNet, YOLOv8n, etc.
- **Autoconfiguración de parámetros** — ajuste dinámico de `IOU_THRESHOLD`, `DIST_THRESHOLD`, `MIN_SCORE` y `MAX_AGE` según el FPS real medido por el `ThrottleController`

---

## Documentos relacionados

| Archivo | Contenido |
|---------|-----------|
| [`architecture.md`](./architecture.md) | Descripción completa de las 4 capas |
| [`performance.md`](./performance.md) | Estrategias de optimización y resultados |
| [`evolution_vector_processor.md`](./project-evolution/evolution_vector_processor.md) | Historia detallada de la Capa 3 |
| [`evolution_throttle_controller.md`](./project-evolution/evolution_throttle_controller.md) | Historia del ThrottleController |

---

**© 2025 Fernando Flores Alvarado — Todos los derechos reservados bajo las licencias indicadas.**  
Este contenido forma parte del proyecto **WebAI Inference Throttling** bajo esquema de licencias dual  
(*MIT* para código + *CC BY 4.0* para documentación).
