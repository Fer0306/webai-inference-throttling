# Optimización y rendimiento: WebAI Inference Throttling

> **Proyecto:** WebAI Inference Throttling  
> **Autor:** Fernando Flores Alvarado  
> **OWASP Project Leader (México)**  
> **Contacto:** fernando.alvarado@owasp.org  
> **Contacto:** fernandofa0306@gmail.com  

> ℹ️ **Note:** This document is written in Spanish. You can use your browser to translate it into English.
>  
> The Spanish version is preserved intentionally as part of the project's authorship and intellectual identity.

---

## El problema real

Procesar cada frame de la cámara con un modelo de visión por computadora
parece lógico, pero en la práctica genera problemas críticos especialmente
en dispositivos móviles y hardware limitado:

- El dispositivo se calienta muy rápido
- La batería se agota rápidamente
- Parpadeo constante en los cuadros delimitadores
- Movimiento inestable por el temblor natural de la mano
- Detección de objetos parciales (falsos positivos)
- CPU al límite en dispositivos de gama baja

Estos problemas no son de código — son consecuencia de un modelo de ejecución
que no considera las limitaciones reales del hardware.

---

## Estrategias implementadas

### 1. Throttling de inferencias

En lugar de correr el modelo en cada frame, se corre cada N milisegundos
según el presupuesto de CPU disponible.

**Presupuesto:** ~200 ms/s dedicados a inferencia (~20% del CPU).

El `ThrottleController` calibra el dispositivo con runs de warmup, mide
el tiempo real de inferencia en ese hardware específico, y calcula el FPS
objetivo sostenible. El ajuste es continuo: ±1 fps por ciclo.

```
FPS objetivo = BUDGET_MS / ms_por_inferencia
             = 200ms / tiempo_real_medido
```

### 2. Procesamiento basado en cambios

La Capa 3 (`VectorProcessor`) compara cada predicción contra el estado
anterior. Si no hubo cambio real, no se ejecuta ningún draw call.

**Cero redraws en frames sin cambios** — el canvas solo se toca cuando
hay algo diferente que mostrar.

### 3. Eliminación de draw calls innecesarios

La Capa 4 (`CanvasRenderer`) no decide cuándo dibuja — solo ejecuta
cuando la Capa 3 lo confirma. Esto elimina completamente el redibujado
por ruido del modelo, reordenamiento del array, o detecciones momentáneas.

### 4. Pausa automática por inactividad

Después de 25 segundos sin detecciones, el sistema pausa automáticamente
la inferencia. Se reactiva con un botón explícito.

Esto protege la batería en escenas estáticas donde el usuario olvidó
cerrar la aplicación.

### 5. Pausa automática por visibilidad

Cuando el tab pierde el foco (`visibilitychange`), la inferencia se detiene.
Se reanuda automáticamente cuando el tab vuelve a estar activo.

### 6. Ajuste de resolución

La calibración del throttle considera la resolución de entrada:

```js
const base   = 1280 * 720;  // HD como referencia
const factor = Math.max(0.5, Math.min(1.0, base / pixCount));
// A mayor resolución que HD → penalizar FPS objetivo
```

En dispositivos con poca CPU, usar resolución menor que HD reduce
significativamente el tiempo de inferencia.

---

## Resultados obtenidos

- Renderizado limpio y sin parpadeos innecesarios
- Reducción notable del sobrecalentamiento
- Consumo de batería mucho más bajo
- Tracking de objetos con identidad persistente entre frames
- Cero draw calls en frames sin cambios reales
- Cámara estable en la mayoría de navegadores (incluyendo zoom y linterna)

---

## Perfiles de rendimiento por hardware

El sistema se comporta diferente según el hardware. Estos son los perfiles
observados durante el desarrollo, incluyendo pruebas en hardware limitado
(AMD Athlon Silver 3050U, 2 núcleos, ~1500ms por inferencia):

| Hardware | ms/inferencia | FPS real | `maxAge` recomendado |
|----------|---------------|----------|----------------------|
| CPU lenta (2 núcleos) | ~1500ms | ~1 fps | `3` |
| CPU media (4 núcleos) | ~500ms | ~3 fps | `5` |
| CPU rápida (8+ núcleos) | ~150ms | ~8 fps | `10` |

### La fórmula del tiempo visible de caja fantasma

`maxAge` se mide en **frames de inferencia**, no en segundos. En dispositivos
lentos, cada frame tarda mucho más:

```
tiempo visible de caja fantasma ≈ maxAge × (ms por inferencia) / 1000

Ejemplo CPU lenta:  3 × 1500ms / 1000 = ~4.5 segundos
Ejemplo CPU rápida: 10 × 150ms / 1000 = ~1.5 segundos
```

> Esta fórmula es importante para ajustar `maxAge` correctamente según
> el hardware donde corre el sistema. Ver tabla completa en `techniques.md`.

---

## Impacto más allá del dispositivo

Este enfoque no solo mejora la experiencia en un teléfono. También abre
la puerta a un cambio de paradigma:

Hoy en día, muchos servidores en la nube que procesan imágenes a gran
escala se calientan al punto de necesitar sistemas de enfriamiento con agua.
Con esta solución, gran parte de ese procesamiento puede trasladarse al
cliente, distribuyendo la carga de manera eficiente y reduciendo la presión
sobre la infraestructura en la nube.

- Reduce el consumo de CPU/GPU
- Disminuye el sobrecalentamiento
- Optimiza el uso de batería
- Reduce la carga en infraestructura cloud

👉 Promueve un modelo más **distribuido, eficiente y sostenible**.

---

## Documentos relacionados

| Archivo | Contenido |
|---------|-----------|
| [`architecture.md`](./architecture.md) | Descripción completa de las 4 capas |
| [`techniques.md`](./techniques.md) | Técnicas implementadas y parámetros configurables |
| [`evolution_throttle_controller.md`](./project-evolution/evolution_throttle_controller.md) | Historia del ThrottleController |

---

**© 2025 Fernando Flores Alvarado — Todos los derechos reservados bajo las licencias indicadas.**  
Este contenido forma parte del proyecto **WebAI Inference Throttling** bajo esquema de licencias dual  
(*MIT* para código + *CC BY 4.0* para documentación).
