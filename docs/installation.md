# Instalación: WebAI Inference Throttling

> **Proyecto:** WebAI Inference Throttling  
> **Autor:** Fernando Flores Alvarado  
> **OWASP Project Leader (México)**  
> **Contacto:** fernando.alvarado@owasp.org  
> **Contacto:** fernandofa0306@gmail.com  

> ℹ️ **Note:** This document is written in Spanish. You can use your browser to translate it into English.
>  
> The Spanish version is preserved intentionally as part of the project's authorship and intellectual identity.

---

## 🚀 Cómo ejecutarlo

Este proyecto no requiere instalación de dependencias.
Funciona directamente en el navegador — todas las librerías
se cargan desde CDN en tiempo de ejecución.

### Opción 1 — Demo en vivo

🎮 **[Abrir demo →](https://fer0306.github.io/webai-inference-throttling/index.html)**

### Opción 2 — Ejecución local

1. Clona el repositorio o descarga el ZIP.
2. Abre `index.html` en tu navegador.
3. Selecciona la versión de demo que quieres explorar.
4. Acepta los permisos de cámara cuando el navegador los solicite.
5. Ajusta la frecuencia de inferencia (FPS) (solo disponible en modo manual).
6. Cambia de cámara, utiliza el zoom o activa la linterna desde la UI (según disponibilidad del dispositivo).

> ✅ Recomendado: **Chrome** o **Edge** (soporte completo de WebAPI de cámara, zoom y linterna).

### Versiones disponibles

| Versión | Descripción |
|---|---|
| `v1-three-layers` | Prototipo inicial — 3 capas |
| `v2-four-layers` | Arquitectura de 4 capas completa |
| `v3-vectordiff` | 4 capas + comparación vectorial |
| `v4-ioutracker` | 4 capas + tracking con IoU |

---

## ⚙️ Requisitos

- Navegador moderno con soporte de `getUserMedia`
- Cámara (integrada o externa)
- Conexión a internet (para cargar TensorFlow.js y coco-ssd desde CDN)

> Sin conexión a internet el modelo no cargará.
> En futuras versiones se puede considerar bundling local.

---

👉 Regresar al inicio: [`README.md`](../README.md)

---

**© 2025 Fernando Flores Alvarado — Todos los derechos reservados bajo las licencias indicadas.**  
Este contenido forma parte del proyecto **WebAI Inference Throttling** bajo esquema de licencias dual  
(*MIT* para código + *CC BY 4.0* para documentación).
