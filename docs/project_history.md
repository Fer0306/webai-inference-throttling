# Historia del Proyecto: WebAI Inference Throttling

> **Proyecto:** WebAI Inference Throttling  
> **Autor:** Fernando Flores Alvarado  
> **OWASP Project Leader (México)**  
> **Contacto:** fernando.alvarado@owasp.org  
> **Contacto:** fernandofa0306@gmail.com  

> ℹ️ **Note:** This document is written in Spanish. You can use your browser to translate it into English.
>  
> The Spanish version is preserved intentionally as part of the project's authorship and intellectual identity.

---

## Sobre este documento

Este archivo es el mapa de evolución del proyecto. Explica de dónde vino, cómo creció, y en qué punto entró la inteligencia artificial como colaborador técnico. Está escrito para cualquier persona que llegue al repositorio y quiera entender no solo qué hace el sistema, sino **cómo llegó a ser lo que es**.

La historia no está editada para verse bien — está escrita para ser honesta.

---

## El origen: BotellaControl (2009–2014)

El proyecto no nació como un sistema de detección de objetos. Nació como una herramienta de inventario inteligente para el control de licores en bares y restaurantes, llamada **BotellaControl**.

BotellaControl requería medir el volumen de líquido en botellas usando visión por computadora directamente desde el navegador — sin servidores, sin apps nativas, sin infraestructura costosa. Eso llevó a trabajar con `getUserMedia`, canvas, y TensorFlow.js cuando estas tecnologías eran todavía muy nuevas y con poca documentación.

Fue durante ese desarrollo que aparecieron dos problemas reales que no tenían solución documentada:

1. **El modelo de detección colapsaba el CPU del dispositivo** cuando se ejecutaba en cada frame del video.
2. **El canvas se redibujaba constantemente** aunque los objetos no se hubieran movido, generando trabajo visual innecesario.

Esos dos problemas son el origen directo de WebAI Inference Throttling y de todo lo que vino después.

---

## Línea de tiempo del proyecto

```
2009 ──── Diseño de la herramienta para BotellaControl

(2010 - 2014) ──── Desarrollo activo de la herramienta para BotellaControl

     ──── Pausa en el proyecto (No existia Machine Learning) para detectar la botella

~2024 - 2025 ─── Retoma del proyecto con nueva perspectiva (Integrar Cámara y Machine Learning)

2025 ──── BotellaControl: primer contacto con los problemas reales:
            1. El dispositivo se calienta muy rápido.
            2. La batería se agota rápidamente al ejecutar la detección.
            3. Los cuadros delimitadores se actualizan con demasiada frecuencia, lo que provoca un efecto de parpadeo.
            4. El movimiento de la cámara es inestable debido al temblor de la mano.
            5. A veces, el modelo detecta objetos parciales, lo que provoca capturas incorrectas.

            (CPU colapsado por inferencias sin control)
            (el problema de throttling se vuelve bloqueante)

2025 ──── Busqueda de soluciones reales en Internet:
            (SIN SOLUCION)

2025 ──── Busqueda de soluciones con WebAI:
            BotellaControl presentado a Kenji Baheux (Google Chrome WebAI PM)
            (Buscando respuestas o soluciones - SIN SOLUCION)
            Kenji me compartió un consejo que cambió la forma en que enfoqué mi solución:

            “Incluso con un modelo eficiente, ejecutarlo constantemente es costoso. Si realmente necesitas una vista previa en vivo, no tienes que procesar cada fotograma. Puedes limitar la inferencia (throttle) para que se active una vez cada medio segundo, o incluso una vez por segundo. Esto le da al procesador un respiro.”

            💡 Esa conversación cambió la dirección y fue el punto de partida para crear una solución a los problemas de forma general.

~2025 ─── Crear solucion con nueva perspectiva
           Decisión: resolver el problema de forma general,
           no solo para BotellaControl

     ──── Primer control de cámara desde el navegador
           p1-camera-control.html

     ──── Canvas responsivo superpuesto al video
          Compatibilidad: móvil, tablet, escritorio
           p2-canvas-overlay-fixed.html

     ──── Primeros prototipos (4 iteraciones)
           - p3a-detection-history-v1.html        ← Primera comparación de historial, bug de acumulación
           - p3b-detection-throttle-time.html     ← Throttling por tiempo + resolución baja
           - p3c-detection-history-v2.html        ← Corrección del bug: reemplazar estado
           - p3d-detection-throttle-ui.html       ← Throttling controlado desde la UI

     ──── Primera detección real — código espagueti
           Todo funciona, nada está separado
           - p3f-spaghetti-controls-visible.html  ← Controles UI expuestos, base funcional

     ──── Demo presentado a Kenji Baheux (Google Chrome WebAI PM)
           camara6.php - Renombrado a: p3e-spaghetti-controls-hidden.html

     ──── Validación externa: Kenji confirma que el problema es real
           El proyecto deja de ser una hipótesis personal

     ──── Finalizando Auto-reactivación por detección de movimiento en píxeles
           - p3g-spaghetti-pixel-wakeup.html

     ──── Diseño de la arquitectura de 4 capas (sin código aún)
           La estructura se define antes de escribir una sola clase

     ──── Módulo heurístico autónomo: calibración automática por dispositivo
           p4-heuristic-module.html

     ──── Código con 3 capas estructuradas
           VideoController + ModelController + PredictionProcessor

     ──── Arquitectura de 4 capas completada
           VideoController + ModelController + PredictionProcessor + CanvasRenderer

     ──── Pausa de ~8 meses (Inicio de Proyecto en OWASP - OWASP Project Leader)

~2026 ─── Retoma del proyecto con
           Refactorización profunda de todos los módulos
           ThrottleController como clase encapsulada
           VectorProcessor: tracking por IoU
           CanvasRenderer: estabilidad visual por frames

presente ─ Estado actual: 4 capas inteligentes + módulo heurístico autónomo
```

---

## La fase de fundamentos

Antes de existir una sola clase, antes de hablar de capas o módulos, hubo una fase de aprendizaje y dominio progresivo que raramente se documenta en proyectos open source. Esta es esa fase.

Lo que se ve en esta secuencia no es desorden — es **método**. Cada archivo resolvió un problema concreto antes de avanzar al siguiente. Aunque en ese momento no se sintiera como arquitectura, lo era.

### Paso 1 — Dominar la cámara
**Archivo:** `p1-camera-control.html`

El primer problema a resolver fue el más básico: acceder a la cámara del dispositivo desde el navegador usando `getUserMedia`. Manejar permisos, streams, cambio entre cámara frontal y trasera, detener y reanudar.

Sin dominar esto, no había nada más que construir.

### Paso 2 — Canvas responsivo encima del video
**Archivo:** `p2-canvas-overlay-fixed.html`

El segundo problema fue más complejo de lo que parece: superponer un `<canvas>` exactamente encima del video, sincronizado en tamaño y posición, que funcionara correctamente en teléfonos, tablets y computadoras de escritorio.

Resolver algo que a primera vista parecía trivial. Ese es el proceso real de entender cómo se comporta el navegador en dispositivos reales — no en el emulador, en el dispositivo.

Algunos problemas que se presentaron:
 - El video que se estiraba diferente en móvil
 - El canvas que perdía sincronía al rotar la pantalla
 - Los píxeles HiDPI que desplazaban las cajas
 - El layout que colapsaba en pantallas pequeñas

### Paso 3 — Primeros prototipos
**Archivos:** `p3a-detection-history-v1.html`, `p3b-detection-throttle-time.html`, `p3c-detection-history-v2.html`, `p3d-detection-throttle-ui.htmlp`

Con la cámara bajo control y el canvas sincronizado, llegó el momento de resolver diferentes problema, hacer que la inferencia, el throttling, el canvas y la UI convivieran en el mismo bloque sin separaciónm, se pudiera separa en dunciones y resolviendo problemas

Esto requirió cuatro iteraciones. Cada una resolvía un problema:

p3a-detection-history-v1.html        ← Primera comparación de historial, bug de acumulación
p3b-detection-throttle-time.html     ← Throttling por tiempo + resolución baja
p3c-detection-history-v2.html        ← Corrección del bug: reemplazar estado
p3d-detection-throttle-ui.html       ← Throttling controlado desde la UI

### Paso 4 — Primera detección real: el código espagueti
**Archivo:** `p3f-spaghetti-controls-visible.html`

Con la cámara bajo control y el canvas sincronizado, llegó el momento de agregar el modelo de detección. Este es el código espagueti — el primero en el que la inferencia, el throttling, el canvas y la UI conviven en el mismo bloque péro con sepracion de funciones pero tdo mesclado.

Todo el sistema vivía en un par de bloques `async`. Variables globales, funciones sueltas, lógica mezclada.

Funciona. Detecta objetos. Dibuja cajas. Pero todo está mezclado y modificar cualquier parte implica riesgo de romper las demás.

El sistema funcionaba, pero era imposible de modificar sin romper algo más.

Este archivo es el punto de partida de toda la arquitectura que vino después. No se esconde — se documenta como lo que es: la base desde la que se construyó todo.

### Paso 5 — El demo para Kenji
**Archivo:** `camara6.php` - Renombrado a: `p3e-spaghetti-controls-hidden.html`

Con el sistema funcionando, se preparó un demo para presentar a **Kenji Baheux**, Product Manager de WebAI en Google Chrome. El objetivo era validar si el problema que el proyecto intentaba resolver era real y relevante desde la perspectiva del equipo que desarrolla las APIs del navegador.

Kenji confirmó que sí con un pulgar arriba — el problema existe, afecta a desarrolladores reales, y la dirección era correcta.

Este momento merece estar documentado porque marca una transición clara: el proyecto dejó de ser una hipótesis personal y se convirtió en un problema **validado externamente por alguien con autoridad técnica en el tema**. Esa validación fue lo que justificó invertir el tiempo en diseñar una arquitectura formal en lugar de seguir parcheando el código espagueti.

### Paso 6 — Diseño de la arquitectura (sin código)
**Sin archivo de código — solo decisiones.**

Antes de escribir una sola clase, se diseñó la arquitectura de 4 capas. La pregunta central fue: si cada responsabilidad tuviera su propio módulo, ¿cómo se comunicarían? ¿Qué sabría cada uno y qué no?

Esta decisión de diseñar antes de codificar es la razón por la que la arquitectura final tiene coherencia — no fue emergente del código, fue intencional desde el principio. La estructura existió en papel antes de existir en JavaScript.

### Paso 7 — El módulo heurístico autónomo
**Archivo:** `p4-heuristic-module.html`

Antes de construir los 4 módulos principales, se construyó el módulo de control heurístico de forma independiente. La idea era que el sistema pudiera calibrarse automáticamente para cualquier dispositivo — sin configuración manual, sin valores fijos que funcionaran en un teléfono pero colapsaran en otro.

Este módulo hacía un muestreo inicial de la cámara, medía el tiempo real de inferencia en ese hardware específico, y calculaba el FPS objetivo sostenible. La autonomía del sistema empieza aquí.

La evolución posterior de este módulo hacia `ThrottleController` está documentada en:

📄 [`evolution_throttle_controller.md`](./docs/project-evolution/evolution_throttle_controller.md)

### Paso 8 — Las 4 capas
Con el módulo heurístico probado y la arquitectura diseñada, se construyó el código completo. A partir de aquí la historia continúa en los documentos de evolución de cada módulo.

---

## Las etapas de modularización

### Etapa 1 — Código espagueti
**Archivo:** `p3f-spaghetti-controls-visible.html`

Todo el sistema en un único bloque. Variables globales, funciones sueltas, lógica mezclada. Funciona pero no puede crecer.

📄 [`EVOLUCION_THROTTLE_CONTROLLER.md`](docs/project-evolution/EVOLUCION_THROTTLE_CONTROLLER.md)

---

### Etapa 2 — Arquitectura de 3 capas
**Archivo:** `p5a-three-layers.html`

El primer intento de modularización real. Se definieron clases con responsabilidades separadas:

- `VideoController` — gestión de cámara
- `ModelController` — inferencia con throttling por frames
- `PredictionProcessor` — comparación de predicciones **y** renderizado (problema de doble responsabilidad)

El problema de esta etapa: `PredictionProcessor` hacía dos cosas. Comparaba predicciones y dibujaba en el canvas. Esas dos responsabilidades estaban acopladas — el procesador no podía existir sin un canvas.

---

### Etapa 3 — Arquitectura de 4 capas
**Archivo:** `p5b-four-layers.html`

La separación que desbloqueó todo lo que vino después. `PredictionProcessor` se dividió en dos módulos con responsabilidad única:

- `PredictionProcessor` — **solo** compara predicciones y detecta cambios reales
- `CanvasRenderer` — **solo** dibuja cuando hay cambios

A partir de esta separación, cada módulo pudo evolucionar de forma independiente sin afectar a los demás.

La historia completa de esta transición está en:

📄 [`evolution_3_to_4_layers.md`](./docs/project-evolution/evolution_3_to_4_layers.md)

---

### Etapa 4 — Refactorización profunda con asistencia de IA
**Archivos:** versiones refactorizadas de todos los módulos

Después de una pausa de aproximadamente 8 meses, el proyecto se retomó con asistencia de IA. La arquitectura de 4 capas ya estaba definida — lo que se hizo fue:

**En el módulo heurístico:**
- Encapsulación completa en clase `ThrottleController`
- Desacoplamiento de la calibración del modelo
- Corrección de un bug silencioso en el factor de resolución
- Ajuste dinámico de FPS extraído del loop principal

**En la Capa 3 (`VectorProcessor`):**
- Evolución del sistema de Comparación *matching* de objetos:
  - Inicialmente: comparación directa de vectores por índice
  - Luego: comparación vectorial con umbral adaptativo
  - Finalmente: tracking con identidad persistente
- Introducción de IoU (*Intersection over Union*) como algoritmo de *matching*
- Fallback por distancia euclidiana entre centros
- Control de objetos fantasma mediante `MAX_AGE`
- Filtro de detecciones falsas usando `MIN_SCORE` y `stableFrames`

**En la Capa 4 (`CanvasRenderer`):**
- Renderizado visual diferenciado según estabilidad del objeto
- Labels con ID persistente del tracker
- Indicador de punto de confirmación
- Soporte HiDPI completo

Estas dos evoluciones están documentadas en:

📄 [`evolution_vector_processor.md`](./docs/project-evolution/evolution_vector_processor.md)

---

## Estado actual de la arquitectura

```
┌───────────────────────────────────────────────────────────┐
│                 WebAI Inference Throttling                │
│                  Arquitectura 4 Capas                     │
└───────────────────────────────────────────────────────────┘

  ┌──────────────────┐
  │  VideoController │  Capa 1
  │  Flujo de cámara │  getUserMedia nativo, sin intermediarios
  │  nativo          │  Zoom, torch, cambio de dispositivo
  └────────┬─────────┘
           │ stream de video
           ▼
  ┌──────────────────┐   ┌─────────────────────┐
  │ ModelController  │◄──│ ThrottleController   │  Módulo auxiliar
  │ Inferencia con   │   │ Heurística adaptativa│  Calibra el dispositivo
  │ throttling       │   │ FPS según hardware   │  Ajuste dinámico continuo
  └────────┬─────────┘   └─────────────────────┘
           │ predicciones brutas
           ▼
  ┌──────────────────┐
  │ VectorProcessor  │  Capa 3
  │ Tracking por IoU │  Identidad persistente por objeto
  │ ¿Hubo cambio?    │  Matching primario IoU, fallback distancia
  └────────┬─────────┘
           │ solo si hay cambio real
           ▼
  ┌──────────────────┐
  │ CanvasRenderer   │  Capa 4
  │ Render           │  Dibuja solo cuando hay cambios
  │ inteligente      │  Visual diferenciado por estabilidad
  └──────────────────┘
```

---

## Sobre el uso de IA en este proyecto

A partir de la Etapa 4, el desarrollo se realizó con asistencia activa de IA.

El rol fue el siguiente:

**Del autor:** la arquitectura conceptual, la definición del problema, las decisiones de diseño, los algoritmos centrales, y la dirección de cada etapa de evolución.

**De la IA:** la implementación de clases encapsuladas a partir de la arquitectura definida, la identificación y corrección de bugs, y la documentación técnica de cada módulo.

El criterio fue simple: documentar el proceso real, no el proceso idealizado.

---

## Índice de documentos

| Archivo | Contenido |
|---------|-----------|
| `project_history.md` | Este archivo — mapa general de evolución |
| `evolution_throttle_controller.md` | Del código espagueti al módulo heurístico encapsulado |
| `evolution_3_to_4_layers.md` | La separación de PredictionProcessor y CanvasRenderer |
| `evolution_vector_processor.md` | De comparación por índice a tracking por IoU |

---

*"Compartir con responsabilidad, es inspirar para construir el futuro."*  
— Fernando Flores Alvarado

---

**© 2025 Fernando Flores Alvarado — Todos los derechos reservados bajo las licencias indicadas.**  
Este contenido forma parte del proyecto **WebAI Inference Throttling** bajo esquema de licencias dual  
(*MIT* para código + *CC BY 4.0* para documentación).
