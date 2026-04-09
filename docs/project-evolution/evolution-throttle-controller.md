# Evolución del Módulo Heurístico: ThrottleController

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

Este documento registra la evolución real del módulo de control heurístico del sistema de inferencia por capas. Es un registro honesto del proceso: desde el código espagueti inicial, pasando por los intentos de estructuración propios, hasta la refactorización asistida con IA.

El propósito no es esconder ese proceso — es documentarlo como lo que fue: un desarrollo iterativo donde la IA actuó como acelerador técnico, pero donde la arquitectura conceptual y el problema original siempre fueron del autor.

---

## Etapa 1 — El problema original (código espagueti)

El primer código funcionaba. Detectaba objetos, controlaba FPS, calibraba el dispositivo. Pero todo vivía junto, revuelto en un único bloque `async IIFE`:

- Variables de estado globales (`targetFPS`, `lastInferMsAvg`, `lastInferMs`) dispersas por el script
- La heurística de calibración (`warmupAndCalibrate`) acoplada directamente al modelo (`modelo.detect`)
- El ajuste dinámico de FPS embebido **dentro del loop de inferencia**, mezclado con el filtrado de clases y el redibujado del canvas
- La función `estimateAutoFPS` como función suelta sin contexto propio

```js
// Así vivía el estado — variables globales sin dueño claro
let lastInferMsAvg = 120;
let targetFPS = 5;
let lastInferTime = 0;

// La calibración sabía del modelo directamente
async function warmupAndCalibrate() {
    for (let i = 0; i < WARMUP_RUNS; i++) {
        await modelo.detect(video); // acoplamiento directo
    }
    // ...
}

// El ajuste dinámico estaba DENTRO del loop de inferencia
// mezclado con la lógica de detección y dibujo
const budgetFps = Math.max(1, Math.floor(BUDGET_MS_PER_SEC / ...));
if (budgetFps < targetFPS) targetFPS = Math.max(1, targetFPS - 1);
```

**El síntoma concreto:** si se quería cambiar la heurística de throttling, había que buscar en tres lugares distintos del archivo. Si se quería reusar la lógica en otro contexto, no era posible sin copiar y pegar.

Esto no es un error de principiante — es la consecuencia natural de construir algo iterativamente sin saber todavía cuál sería su forma final. El código cumplía su función; simplemente no estaba listo para crecer.

---

## Etapa 2 — La decisión de las capas

Antes de refactorizar el código, se tomó una decisión arquitectural importante: si el sistema iba a tener múltiples responsabilidades (cámara, modelo, renderizado, heurística), cada responsabilidad necesitaba su propio módulo con límites claros.

Esa decisión vino del autor. La estructura de 4 capas fue diseñada conceptualmente antes de escribir una sola clase:

1. `VideoController` — gestión de cámara y stream
2. `ModelController` — carga, calibración y loop de inferencia
3. `ThrottleController` — heurística de rendimiento
4. `CanvasRenderer` / `VectorProcessor` — visualización y comparación de estados

La idea central: **que cada módulo pudiera existir sin saber cómo funcionan los demás**.

---

## Etapa 3 — Refactorización asistida con IA

Con la arquitectura definida, se usó asistencia de IA, para implementar los módulos como clases bien encapsuladas. El módulo que más cambió estructuralmente fue el heurístico.

### De funciones sueltas a clase encapsulada

```js
// ANTES: lógica dispersa, sin dueño
let targetFPS = 5;
function estimateAutoFPS(avgMs, pixCount) { ... }
async function warmupAndCalibrate() { ... }
// ajuste dinámico dentro del loop...

// DESPUÉS: todo encapsulado con responsabilidad única
class ThrottleController {
  constructor(options = {}) {
    this.BUDGET_MS   = options.budgetMs   || 200;
    this.WARMUP_RUNS = options.warmupRuns || 3;
    this.targetFPS   = options.initialFPS || 5;
    this.avgInferMs  = options.initialAvgMs || 120;
    this.lastInferMs = 120;
    this.calibrated  = false;
  }
  // ...
}
```

### Desacoplamiento de la calibración

Este fue el cambio de diseño más importante. En el código original, `warmupAndCalibrate()` llamaba directamente a `modelo.detect()` — la heurística **sabía** que existía un modelo COCO-SSD. En la clase nueva, recibe `detectFn` como parámetro:

```js
// ANTES: acoplamiento directo al modelo
async function warmupAndCalibrate() {
    await modelo.detect(video); // ThrottleController sabe del modelo
}

// DESPUÉS: el ThrottleController no sabe qué modelo existe
async calibrate(detectFn, videoElement) {
    await detectFn(videoElement); // recibe la función, no el modelo
}
```

**Por qué importa:** si mañana se cambia COCO-SSD por otro modelo de detección, el `ThrottleController` no necesita modificarse. Es el principio de inversión de dependencias aplicado a un contexto práctico.

### Separación del ajuste dinámico

El ajuste continuo de FPS salió del loop de inferencia y se convirtió en un método propio:

```js
// ANTES: embebido en el loop, mezclado con todo lo demás
const budgetFps = Math.max(1, Math.floor(BUDGET_MS_PER_SEC / Math.max(1, lastInferTime)));
if (budgetFps < targetFPS) targetFPS = Math.max(1, targetFPS - 1);
else if (budgetFps > targetFPS && targetFPS < 30) targetFPS = targetFPS + 1;

// DESPUÉS: método propio, una sola responsabilidad
update(actualInferMs) {
    this.lastInferMs = actualInferMs;
    const budgetFps = Math.max(1, Math.floor(this.BUDGET_MS / Math.max(1, actualInferMs)));
    if      (budgetFps < this.targetFPS) this.targetFPS = Math.max(1, this.targetFPS - 1);
    else if (budgetFps > this.targetFPS && this.targetFPS < 30) this.targetFPS++;
}
```

Ahora el loop de inferencia simplemente llama `throttle.update(ms)` — una línea, sin saber nada de la lógica interna.

### Corrección del bug en el factor de resolución

Durante la refactorización se identificó y corrigió un bug silencioso en la heurística de penalización por resolución:

```js
// ANTES: el límite superior de 1.5 era letra muerta
// porque (base / Math.max(base, pixCount)) NUNCA supera 1.0
const factor = Math.max(0.5, Math.min(1.5, base / Math.max(base, pixCount)));

// DESPUÉS: rango real y honesto [0.5, 1.0]
const factor = Math.max(0.5, Math.min(1.0, base / Math.max(base, pixCount)));
```

El comportamiento del sistema no cambiaba (el 1.5 nunca se alcanzaba), pero el código ahora refleja correctamente su intención: a más resolución que 720p, penalizar FPS; a igual o menor resolución, no bonificar.

### El getter `interval`

Un detalle menor pero de buena práctica: en lugar de que el loop calcule `1000 / targetFPS` en cada ciclo, el controlador expone directamente el intervalo como propiedad computada:

```js
get interval() { return 1000 / this.targetFPS; }
```

Un solo punto de verdad para ese cálculo.

---

## Tabla comparativa

| Aspecto | Etapa 1 (original) | Etapa 3 (refactorizado) |
|---|---|---|
| Estructura | Funciones sueltas + variables globales | Clase encapsulada con responsabilidad única |
| Calibración | Acoplada a `modelo.detect()` | Recibe `detectFn` (desacoplada) |
| Ajuste dinámico | Dentro del loop, mezclado con detección y dibujo | Método `update()` separado |
| Estado interno | Variables globales sin dueño | Propiedades de instancia de la clase |
| Bug del factor | Límite 1.5 inalcanzable | Corregido a 1.0 |
| Reusabilidad | Imposible sin copiar y pegar | Instanciable con opciones configurables |

---

## Lo que no cambió

La **lógica de la heurística** es esencialmente la misma que en el código original:

- Presupuesto de CPU en ms/s dividido entre el costo de inferencia → FPS objetivo
- Penalización por resolución superior a 720p
- Suavizado de ±1 FPS por ciclo para evitar oscilaciones

La refactorización no inventó una heurística nueva. La encapsuló, la desaclopó y corrigió un bug. El algoritmo es del autor.

---

## Estado actual

El `ThrottleController` está operativo y conectado a `ModelController`. Existen variables adicionales candidatas a autoconfiguración que aún no han sido incorporadas al módulo — esa es la siguiente etapa de desarrollo.

---

**© 2025 Fernando Flores Alvarado — Todos los derechos reservados bajo las licencias indicadas.**  
Este contenido forma parte del proyecto **WebAI Inference Throttling** bajo esquema de licencias dual  
(*MIT* para código + *CC BY 4.0* para documentación).
