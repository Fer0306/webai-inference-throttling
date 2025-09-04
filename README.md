# WebAI Inference Throttling Demo

Este proyecto demuestra cómo implementar **throttling de inferencias** en un modelo de IA ejecutado en el navegador usando **TensorFlow.js**.  
El objetivo es optimizar el uso de recursos en tiempo real, especialmente en dispositivos móviles, evitando bloqueos o sobrecarga cuando se realizan inferencias continuas desde la cámara.

---

## ✨ Características principales

- 📸 **Gestión avanzada de la cámara**  
  - Cambio de cámara (frontal/trasera).  
  - Soporte de diferentes resoluciones: QVGA, VGA, HD, FullHD, 4K.  
  - Control de zoom y linterna (si el dispositivo lo permite).  

- 🧠 **Inferencia optimizada con throttling**  
  - Basado en `coco-ssd` de TensorFlow.js.  
  - Configuración dinámica de frecuencia de inferencia (1–10 FPS).  
  - Filtrado de clases de objetos (ejemplo: `cup`, `bottle`).  

- 🎨 **Procesamiento de predicciones**  
  - Dibuja bounding boxes en canvas.  
  - Redibuja solo si las predicciones cambian → ahorro de recursos.  

- 📱 **Interfaz adaptable (responsive)**  
  - Controles intuitivos para cámara, linterna, zoom y FPS.  
  - Compatible con desktop, tablet y móvil.  

---

## 🏗️ Arquitectura del proyecto

Este proyecto sigue una arquitectura modular basada en capas inteligentes para optimizar el rendimiento:

1. **Capa de cámara (VideoController)**
   Flujo directo desde `getUserMedia()`, con soporte de cambio de cámara, encendido/apagado, zoom y linterna.
   Mejoras: cambio de resolución, pause y play.

2. **Capa de modelo (ModelController)**  
   Administra la carga y ejecución del modelo `coco-ssd` de TensorFlow.js.  
   - Permite configurar el *throttling* dinámico a las inferencias.  
   - Soporta filtrado de clases.

3. **Capa de procesamiento de predicciones (PredictionProcessor)**
   Procesa las predicciones recibidas y decide cuándo actualizar comparando predicciones (bounding boxes, clases, scores).
   la visualización en el canvas.   

4. **Capa de renderizado en canvas (pendiente de integrar)**
   En la arquitectura final, esta capa se encargará de manejar de forma independiente el dibujado en el canvas.  

---

## 🔔 Nota importante

En este demo, las capas 3 y 4 (**PredictionProcessor** y **Canvas**) todavía están unificadas dentro de una sola clase.  
Esto se debe a que el código se encuentra en una **versión inicial funcional**, pero la separación completa está en la hoja de ruta del proyecto.  

👉 Próximamente, se dividirá la lógica en dos capas independientes:  
- **PredictionProcessor** → solo procesará y filtrará predicciones.  
- **CanvasRenderer** → se encargará exclusivamente de renderizar en el canvas.  

De esta manera, la arquitectura quedará totalmente modular y alineada con lo descrito en el artículo de Medium.

---

## 🧪 Objetos detectados en la demo

Esta demo está configurada para detectar **únicamente los siguientes objetos**:

- 🥤 `cup`
- 🍼 `bottle`

Esto se debe a que se aplica un **filtro de clases** para reducir la carga de procesamiento y enfocarse en un caso de uso específico.  
Puedes modificar o ampliar este filtro desde el archivo JavaScript (`ModelController.js`).

---

## 🚀 Cómo probarlo

1. Clona este repositorio o descarga el archivo `index.html`.  
2. Abre `index.html` en tu navegador (se recomienda **Chrome** o **Edge**).  
3. Acepta permisos de cámara cuando el navegador los solicite.  
4. Ajusta la frecuencia de inferencia (FPS), cambia de cámara o activa la linterna desde la UI.  

Este proyecto demuestra cómo implementar **throttling de inferencias** en un modelo de IA ejecutado en el navegador usando **TensorFlow.js**.  
El objetivo es optimizar el uso de recursos en tiempo real, especialmente en dispositivos móviles, evitando bloqueos o sobrecarga cuando se realizan inferencias continuas desde la cámara.

🎮 **[¡Probar la demo en vivo! →](https://fer0306.github.io/fernandofa0306/proyectos/1-webai-inference-throttling/index.html)**

---

## 📂 Estructura del proyecto

webai-inference-throttling/
├── index.html # Demo principal
├── README.md # Documentación
└── LICENSE # Licencia (opcional)



---

## 📸 Capturas de pantalla (opcional)

![Limitación de inferencia en WebAI](img/inference-throttling-limit.jpg)

---

## 📝 Licencia

Este proyecto se distribuye bajo la licencia **MIT**.  
Puedes usarlo, modificarlo y compartirlo libremente, siempre dando el crédito correspondiente.  

---

## 👨‍💻 Autor

**Fernando Flores Alvarado**  
- 📧 fernandofa0306@gmail.com  
- 💼 [LinkedIn Profile](https://www.linkedin.com/in/fernando-flores-alvarado-2786b21b8/)  
- 🔗 [Medium Articles](https://medium.com/@fernandofa0306)
*“Compartir con responsabilidad es inspirar para construir el futuro.”* 🚀


---


