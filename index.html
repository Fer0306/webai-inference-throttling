<!DOCTYPE html>
<html lang="es">
<head>
	<meta charset="UTF-8"/>
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>WebAI Inference Throttling Demo</title>
	<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@4.13.0"></script>
	<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"></script>
	<style>
		body {
			font-family: sans-serif;
			background: #222;
			color: #eee;
			margin: 0;
			padding: 0;
			text-align: center;
		}

		h2 {
			margin-bottom: 50px; /* espacio debajo del título */
		}
	</style>

	<style>
		/* Contenedor de cámara y controles */
		#Camera, #InferenceControls {
			width: 90%; /* por defecto más fluido */
			margin: 5px auto;
			text-align: center;
		}

		#Camera {
			display: flex;
			justify-content: center; /* centra horizontal */
			align-items: center;     /* centra vertical */
			flex-direction: column;  /* mantiene título y controles debajo */
		}

		/* Ajuste de botones principales */
		#InferenceControls {
			display: flex;
			justify-content: space-between;
			align-items: center;
			gap: 10px;
			flex-wrap: nowrap;
		}

		#InferenceControls > div {
			flex: 1;
			min-width: 120px;
		}


/**************************************************************************/
/**************************************************************************/
/**************************************************************************/


		#cameravideo {
			width: 100%;
			height: 50vh; /* 50% de la altura de la ventana */
			background: black;
			overflow: hidden;
			border-radius: 12px;
			position: relative;
		}
		video {
			top: 0;
			left: 0;
			display: block;
			width: 100%; /* Hace que el video llene todo el contenedor horizontalmente */
		    height: 100%; /* Hace que el video llene todo el contenedor verticalmente */
			object-fit: fill; /* fuerza el estirado */
			position: absolute;
		}
		canvas {
			top: 0;
			left: 0;
			display: block;
			width: 100%;
			height: 100%;
			pointer-events: none;
			position: absolute;
		}


/**************************************************************************/
/**************************************************************************/
/**************************************************************************/


		/* Controles de cámara */
		#cameraControls {
			max-width: 600px;
			margin: 5px auto;
			display: flex;
			flex-wrap: wrap;
			gap: 12px;
			justify-content: center;
		}

		#cameraControls > div {
			flex: 1;
			min-width: 140px;
		}

		/* Botones */
		button, select, input[type=range] {
			padding: 10px 14px;
			font-size: 16px;
			border-radius: 8px;
			border: none;
			outline: none;
			background: #444;
			color: #eee;
			cursor: pointer;
			width: 100%; /* full en móvil */
			box-sizing: border-box;
		}


/**************************************************************************/
/**************************************************************************/
/**************************************************************************/


		label {
			display: block;
			font-size: 14px;
			margin-bottom: 4px;
			text-align:center;
		}
		#cameraMessages {
			margin-top: 5px;
			color: #ff6666;
			font-size: 14px;
			min-height: 20px;
		}


/**************************************************************************/
/**************************************************************************/
/**************************************************************************/


		/* ******* INFO CAMARA ******* */
		#infoButton {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 999;
            background: #444;
            color: #eee;
            border-radius: 50%;
            border: none;
            width: 70px;
            height: 70px;
            font-size: 32px;
            cursor: pointer;
		}
		/* Panel de información: adaptar tamaño */
		#cameraInfo {
			position: fixed;
			top: 80px;
			right: 20px;
			width: 300px;
			background: #333;
			color: #eee;
			border-radius: 12px;
			padding: 16px;
			box-shadow: 0 0 10px rgba(0,0,0,0.7);
			display: none;
			z-index: 998;
		}
		#cameraInfo > h3 {
			margin-top:0;
		}
		#closeCameraInfo {
			margin-top: 12px;
			padding: 6px 12px;
			border-radius: 8px;
			cursor: pointer;
			border: none;
			background: #555;
			background-color: #007bff;
			color: #eee;
		}


/**************************************************************************/
/**************************************************************************/
/**************************************************************************/

		/* ==== MEDIA QUERIES ==== */

		/* 📱 Teléfonos (pantallas < 600px) */
		@media (max-width: 600px) {
			#InferenceControls > div {
				flex: 1 1 auto;      /* No crecer demasiado, pero sí reducir si es necesario */
				min-width: 90px;     /* Ancho mínimo para que el contenido quepa */
				max-width: 120px;    /* Opcional: limitar el ancho máximo */
			}
			#cameraControls {
				display: grid;
				grid-template-columns: repeat(2, 1fr);
				gap: px;
			}
			#cameravideo {
				width: 55%; /* 50% */
				height: 41vh; /* 35vh*/
			}
			button, select, input[type=range] {
				font-size: 14px;
			}
			#divzoom {
				grid-column: 1 / -1; /* hace que ocupe toda la fila */
				width: 100%;
			}
			/* Zoom ocupa toda la fila */
			#ZoomRangeControl {
				width: 100%;
			}
		}

		/* 📲 Tablets (entre 600px y 900px) */
		@media (min-width: 601px) and (max-width: 900px) {
			#InferenceControls > div {
				flex: 1 1 auto;      /* No crecer demasiado, pero sí reducir si es necesario */
				min-width: 90px;     /* Ancho mínimo para que el contenido quepa */
				max-width: 120px;    /* Opcional: limitar el ancho máximo */
			}
			#cameraControls {
				display: grid;
				grid-template-columns: repeat(2, 1fr);
				gap: px;
			}
			#cameravideo {
				width: 45%; /* 50% */
				height: 55vh; /* 35vh*/
			}
			button, select, input[type=range] {
				font-size: 14px;
			}
			#divzoom {
				grid-column: 1 / -1; /* hace que ocupe toda la fila */
				width: 100%;
			}
			/* Zoom ocupa toda la fila */
			#ZoomRangeControl {
				width: 100%;
			}
		}

		/* 💻 PC (más de 900px) */
		@media (min-width: 901px) {
			h2 {
				margin-bottom: 0px; /* espacio debajo del título */
			}
			#Camera, #InferenceControls {
				width: 65%;
			}
			#cameraControls {
				display: grid;
				grid-template-columns: repeat(2, 1fr);
				gap: 12px;
			}
			#divzoom {
				grid-column: 1 / -1; /* hace que ocupe toda la fila */
				width: 100%;
			}
			/* Zoom ocupa toda la fila */
			#ZoomRangeControl {
				width: 100%;
			}
		}

		/* 📱 Teléfonos (pantallas < 600px) */
/*		@media (max-width: 600px) { body { background: #00ff0dff; } }
		/* 📲 Tablets (entre 600px y 900px) */
/*		@media (min-width: 601px) and (max-width: 900px) { body { background: #fff200ff; }}
		/* 💻 PC (más de 900px) */
/*		@media (min-width: 901px) { body { background: #003cffff; } }
*/
	</style>
</head>
<body>
	<h2>WebAI Inference Throttling Demo</h2>

	<!-- Botón flotante de información -->
	<button id="infoButton" title="Información de la cámara">ℹ️</button>

	<!-- Panel de información de la cámara -->
	<div id="cameraInfo">
		<h3>Info Cámara</h3>
		<div id="cameraInfoContent"></div>
		<button id="closeCameraInfo">Cerrar</button>
	</div>

	<!-- Controles Inferencia -->
	<div id="InferenceControls">
		<div>
			<button id="cameraToggleButton">Apagar<br>Cámara</button>
		</div>
		<div>
			<label for="frames">FpS:</label>
			<select id="frames">
					<option value="1" selected>1</option> <!-- seleccionado por defecto -->
					<option value="2">2</option>
					<option value="3">3</option>
					<option value="4">4</option>
					<option value="5">5</option>
					<option value="6">6</option>
					<option value="7">7</option>
					<option value="8">8</option>
					<option value="9">9</option>
					<option value="10">10</option>
				</select>
		</div>
		<div>
			<button id="toggleDetectionButton">Apagar<br>Detección</button>
		</div>
	</div>

	<div id="Camera">
		<!-- Video + Canvas Camara -->
		<div id="cameravideo">
			<video id="video" autoplay muted playsinline></video>
			<canvas id="canvas"></canvas>
		</div>

		<!-- Mensaje Camara-->
		<div id="cameraMessages"></div>

		<!-- Controles Camara -->
		<div id="cameraControls">
			<div>
				<label for="cameras">Cámara:</label>
				<select id="cameras"></select>
			</div>
			<div>
				<label for="ToggleTorchButton">Linterna:</label>
				<button id="ToggleTorchButton">Linterna OFF</button>
			</div>
			<div id="divzoom">
				<label for="ZoomRangeControl">Zoom:</label>
				<input type="range" id="ZoomRangeControl" min="1" max="5" step="0.1" value="1" />
			</div>
		</div>
	</div>

	<script>
		// ======================================================
		// 📦 MÓDULO PROFESIONAL: VideoController
		// ======================================================
		/**
		 * Controlador profesional para manejar video en tiempo real desde la cámara.
		 * Encapsula el acceso a `getUserMedia`, cambio de cámara, zoom y torch.
		 * 
		 * @class VideoController
		 * @example
		 * const videoCtrl = new VideoController(document.querySelector("#video"));
		 * await videoCtrl.init("HD", "environment");
		 */
		class VideoController {
			/**
			 * @constructor
			 * @param {HTMLVideoElement} videoElement - Elemento <video> donde se reproducirá el stream.
			 */
			constructor(videoElement) {
				if (!(videoElement instanceof HTMLVideoElement)) throw new Error("Debes pasar un elemento <video> válido.");
				this.videoElement = videoElement;
				this.stream = null;
				this.track = null;
				this.devices = [];

				// Callbacks configurables
				/** @type {Function|null} Ejecutado cuando el stream está listo */
				this.onReady = null;
				/** @type {Function|null} Ejecutado cuando ocurre un error */
				this.onError = null;
				/** @type {Function|null} Ejecutado cuando se detectan dispositivos */
				this.onDevices = null;

				// Estado interno
				this.currentDeviceId = null;
				this.torchEnabled = false;
				this.zoomSupported = false;
				this.torchSupported = false;
			}

			/**
			 * Verifica si el navegador soporta MediaDevices.
			 * @returns {boolean}
			 */
			static isSupported() {
				return 'mediaDevices' in navigator && 'getUserMedia' in navigator.mediaDevices;
			}

			/**
			 * Obtiene la lista de cámaras disponibles.
			 * @returns {Promise<MediaDeviceInfo[]>}
			 */
			async getDevices() {
				try {
					const devices = await navigator.mediaDevices.enumerateDevices();
					this.devices = devices.filter(d => d.kind === "videoinput");
					if (this.onDevices) this.onDevices(this.devices);
					return this.devices;
				} catch (err) {
					this._handleError(err, "Enumerando dispositivos");
					throw err;
				}
			}

			/**
			 * Inicializa la cámara.
			 * @param {"QVGA"|"VGA"|"HD"|"FullHD"|"4K"} resolutionType - Resolución deseada.
			 * @param {"user"|"environment"} facingMode - Orientación: frontal o trasera.
			 */
			async init(resolutionType = "HD", facingMode = "user") {
				try {
					this.stop();
					const resolution = this._getResolution(resolutionType);
					const constraints = { audio: false, video: { ...resolution, facingMode: facingMode } };

					this.stream = await navigator.mediaDevices.getUserMedia(constraints);
					this.videoElement.srcObject = this.stream;
					this.track = this.stream.getVideoTracks()[0];
					this.currentDeviceId = this.track.getSettings().deviceId;

					await this._waitForVideoReady();
					this._updateCapabilities();

					if (this.onReady) this.onReady();
					await this.getDevices();

				} catch (err) {
					this._handleError(err, "Inicializando video");
					throw err;
				}
			}

			/**
			 * Detiene el stream de video y libera memoria.
			 */
			stop() {
				if (this.stream) {
					this.stream.getTracks().forEach(t => t.stop());
					this.stream = null;
					this.track = null;
				}
				this.videoElement.srcObject = null;
			}

			/**
			 * Cambia a otra cámara por `deviceId`.
			 * @param {string} deviceId - ID de la cámara destino.
			 * @param {"QVGA"|"VGA"|"HD"|"FullHD"|"4K"} resolutionType - Resolución deseada.
			 */
			async switchCamera(deviceId, resolutionType = "HD") {
				try {
					this.stop();
					const resolution = this._getResolution(resolutionType);
					const constraints = { audio: false, video: { ...resolution, deviceId: { exact: deviceId } } };

					this.stream = await navigator.mediaDevices.getUserMedia(constraints);
					this.videoElement.srcObject = this.stream;
					this.track = this.stream.getVideoTracks()[0];
					this.currentDeviceId = this.track.getSettings().deviceId;

					await this._waitForVideoReady();
					this._updateCapabilities();

					if (this.onReady) this.onReady();
				} catch (err) {
					this._handleError(err, "Cambiando cámara");
					throw err;
				}
			}

			/**
			 * Indica si la cámara soporta zoom.
			 * @returns {boolean}
			 */
			isZoomSupported() {
				return !!(this.track && this.zoomSupported);
			}

			/**
			 * Obtiene la configuración de zoom disponible.
			 * @returns {Object|null} { min, max, step, value } o null si no es soportado.
			 */
			getZoomConfiguration() {
				if (!this.track || !this.zoomSupported) return null;
				const caps = this.track.getCapabilities();
				const settings = this.track.getSettings();
				if (caps.zoom) return { min: caps.zoom.min, max: caps.zoom.max, step: caps.zoom.step || 0.1, value: settings.zoom || caps.zoom.min };
				return null;
			}

			/**
			 * Aplica zoom en la cámara.
			 * @param {number} value - Nivel de zoom.
			 */
			async setZoom(value) {
				if (!this.track || !this.zoomSupported) return;
				try {
					await this.track.applyConstraints({ advanced: [{ zoom: value }] });
				} catch (err) {
					this._handleError(err, "Aplicando zoom");
				}
			}

			/**
			 * Indica si la cámara soporta linterna (torch).
			 * @returns {boolean}
			 */
			isTorchSupported() {
				return !!(this.track && this.torchSupported);
			}

			/**
			 * Activa/desactiva la linterna del dispositivo.
			 * @returns {Promise<boolean>} Estado actual de la linterna.
			 */
			async toggleTorch() {
				if (!this.track || !this.torchSupported) return false;
				try {
					this.torchEnabled = !this.torchEnabled;
					await this.track.applyConstraints({ advanced: [{ torch: this.torchEnabled }] });
					return this.torchEnabled;
				} catch (err) {
					this._handleError(err, "Cambiando torch");
					return false;
				}
			}

			/**
			 * Verifica si la cámara está activa y el stream de video está disponible para reproducirse.
			 * Este estado indica que se puede encender/apagar la cámara o cambiar de dispositivo.
			 * No asegura que el video ya haya renderizado ningún frame.
			 * 
			 * @returns {boolean} `true` si la cámara está encendida y el track existe.
			 */
			// La cámara está encendida, el stream y el track existen.
			isReady() {
				return !!(this.stream && this.track);
			}

			/**
			 * Verifica si el video ha renderizado al menos un frame y está listo para usar.
			 * Útil para operaciones que dependen de que el video tenga tamaño real, 
			 * como dibujar sobre un canvas o aplicar inferencias de ML.
			 * 
			 * @returns {boolean} `true` si el video tiene datos renderizados.
			 */
			// El video ya tiene datos renderizados y se puede usar en canvas
			isRendered() {
				// Eso está bien, pero puede fallar en algunos navegadores
				// si el video está en autoplay con muted
/*
				return (
					this.videoElement &&
					!this.videoElement.paused &&
					!this.videoElement.ended &&
					this.videoElement.videoWidth > 0 &&
					this.videoElement.videoHeight > 0
				);
//*/
				// Es Mejor utilizar : (readyState >= 2 = HAVE_CURRENT_DATA).
//				return this.stream && this.track && this.videoElement.readyState >= 2;
				return this.videoElement && this.videoElement.readyState >= 2; // HAVE_CURRENT_DATA
			}

			// ================== MÉTODOS PRIVADOS ==================

			/**
			 * Maneja errores internos del módulo.
			 * @param {Error} err - Error capturado.
			 * @param {string} ctx - Contexto opcional.
			 * @private
			 */
			_handleError(err, ctx = "") {
				console.error("VideoController Error:", ctx, err);
				if (this.onError) this.onError(err);
			}

			/**
			 * Actualiza las capacidades soportadas por la cámara: zoom y torch.
			 * @private
			 */
			_updateCapabilities() {
				if (!this.track || typeof this.track.getCapabilities !== "function") return;
				const caps = this.track.getCapabilities();
				this.zoomSupported = "zoom" in caps;
				this.torchSupported = "torch" in caps;
			}

			/**
			 * Retorna la resolución ideal según el tipo seleccionado.
			 * @param {string} type - Tipo de resolución: "QVGA"|"VGA"|"HD"|"FullHD"|"4K".
			 * @returns {{width: {ideal: number}, height: {ideal: number}}}
			 * @private
			 */
			_getResolution(type) {
				const resolutions = {
					"QVGA": { width: { ideal: 320 }, height: { ideal: 240 } },
					"VGA": { width: { ideal: 640 }, height: { ideal: 480 } },
					"HD": { width: { ideal: 1280 }, height: { ideal: 720 } },
					"FullHD": { width: { ideal: 1920 }, height: { ideal: 1080 } },
					"4K": { width: { ideal: 3840 }, height: { ideal: 2160 } }
				};
				return resolutions[type] || resolutions["HD"];
			}

			/**
			 * Retorna una promesa que se resuelve cuando el video está listo para reproducirse.
			 * @returns {Promise<void>}
			 * @private
			 */
			_waitForVideoReady() {
				return new Promise((resolve, reject) => {
					const onCanPlay = () => { cleanup(); resolve(); };
					const onError = (e) => { cleanup(); reject(e); };
					const cleanup = () => {
						this.videoElement.removeEventListener("canplay", onCanPlay);
						this.videoElement.removeEventListener("error", onError);
					};
					this.videoElement.addEventListener("canplay", onCanPlay, { once: true });
					this.videoElement.addEventListener("error", onError, { once: true });
				});
			}
		}
	</script>


	<script>
		// ======================================================
		// 📦 MÓDULO PROFESIONAL: ModelController
		// ======================================================
		/**
		 * Controlador profesional para manejar.....
		 */
		class ModelController {
			constructor(videoElement, options = {}) {
				if (!(videoElement instanceof HTMLVideoElement)) throw new Error("Debes pasar un elemento <video> válido.");
				this.videoElement = videoElement;

				// Configuración
				this.model = null;
				this.modelName = options.modelName || 'coco-ssd';
				this.throttleFrames = 1; // Inferencia cada n frames
				this.running = false;

				// Contador de frames
				this.frameCount = 0;

				// Callback para entregar predicciones
				this.onPredictions = options.onPredictions || function(preds){};

				// Nuevas clases a filtrar (opcional)
				// Asegurarse de que filterClasses siempre sea un array
				this.filterClasses = Array.isArray(options.filterClasses) ? options.filterClasses : null; // ej: ['person','cup','bottle']

				console.log("🧠 ModelController creado");
			}

			// Carga del modelo
			async loadModel() {
				console.log("🔄 Cargando modelo:", this.modelName);
				if (this.modelName === 'coco-ssd') {
					this.model = await cocoSsd.load();
				} else {
					throw new Error("Modelo no soportado: " + this.modelName);
				}
				console.log("✅ Modelo cargado");
			}

			// Inicia inferencia
			start() {
				if (!this.model) throw new Error("Primero debes cargar el modelo con loadModel()");
				this.running = true;
				this.frameCount = 0;
				this._loop();
			}

			// Detiene inferencia
			stop() {
				this.running = false;
			}

			// Ajusta la frecuencia de inferencia
			setThrottle(frames) {
				this.throttleFrames = frames > 0 ? frames : this.throttleFrames;
			}

			// Bucle interno de inferencia
			async _loop() {
				if (!this.running) return;

				this.frameCount++;
				if (this.frameCount % this.throttleFrames === 0) {
					if (this.videoElement.readyState >= 2) { // HAVE_CURRENT_DATA
						try {
							const predictions = await this.model.detect(this.videoElement);
							// Filtrar solo las clases deseadas
							const filterPredictions = this.filterClasses
								? predictions.filter(p => this.filterClasses.includes(p.class))
								: predictions;
							// Enviamos predicciones al callback
							this.onPredictions(filterPredictions);
						} catch (err) {
							console.error("❌ Error en inferencia:", err);
						}
					}
				}

				// requestAnimationFrame para sincronizar con el render del navegador
				requestAnimationFrame(() => this._loop());
			}
		}
	</script>


	<script>
		// ======================================================
		// 📦 MÓDULO PROFESIONAL: PredictionProcessor
		// ======================================================
		/**
		 * Controlador profesional para manejar.....
		 */
		class PredictionProcessor {
			constructor(canvasElement) {
				if (!(canvasElement instanceof HTMLCanvasElement)) throw new Error("Debes pasar un elemento <canvas> válido.");
				this.canvas = canvasElement;
				this.ctx = this.canvas.getContext('2d');
				this.lastPredictions = [];
			}

			// Compara predicciones actuales con las anteriores
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

			// Actualiza las predicciones y decide si dibujar
			process(predictions) {
				if (this.hasChanges(predictions)) {
					this.lastPredictions = predictions.map(p => ({...p})); // copia profunda
					this.draw(predictions);
				}
			}

			// Dibuja predicciones en el canvas
			draw(predictions) {
				const ctx = this.ctx;
				const canvas = this.canvas;
				ctx.clearRect(0, 0, canvas.width, canvas.height);

				// Asegurar que el canvas siga el tamaño actual del video
				if (canvas.width !== videoElement.videoWidth || canvas.height !== videoElement.videoHeight) {
					canvas.width = videoElement.videoWidth;
					canvas.height = videoElement.videoHeight;
				}

				predictions.forEach(pred => {
					const [x, y, width, height] = pred.bbox;
/*
					// Escalar según como se estiró el video
					const newX = x * scaleX;
					const newY = y * scaleY;
					const newW = width * scaleX;
					const newH = height * scaleY;
//*/
					ctx.strokeStyle = 'lime';
					ctx.lineWidth = 3;
					ctx.strokeRect(x, y, width, height);

					ctx.fillStyle = 'lime';
					ctx.font = '18px Arial';
					ctx.fillText(`${pred.class} (${(pred.score*100).toFixed(1)}%)`, x, y > 20 ? y - 5 : y + 20);
				});
			}
		}
	</script>


	<script>
		// Función para ajustar el canvas al tamaño del video
		function adjustCanvas() {
//*
			canvas.width = videoElement.videoWidth;
			canvas.height = videoElement.videoHeight;
			console.log(`Canvas ajustado a: ${canvas.width}x${canvas.height}`);
//*/
		}
	</script>


	<script>
		// ======================================================
		// 📸 CONEXIÓN DE LOS MÓDULOS
		//   - VideoController
		//   - ModelController
		//   - PredictionProcessor
		//  CON LA UI
		// ======================================================

		// Referencias a elementos del DOM
		const cameraToggleButton = document.getElementById("cameraToggleButton");
		const videoElement = document.getElementById("video");
		const cameraMessages = document.getElementById("cameraMessages")
		const selectCameras = document.getElementById("cameras");
		const torchBtn = document.getElementById("ToggleTorchButton");
		const zoomControl = document.getElementById("ZoomRangeControl");

		const frames = document.getElementById("frames");
		const toggleDetectionButton = document.getElementById("toggleDetectionButton");

// *********************************************************************************************
		// Instancia del controlador VideoController
		const videoCtrl = new VideoController(videoElement);

		// Callbacks del módulo
		videoCtrl.onReady = () => {
			console.log("✅ Video listo");
			configureCameraUI();
			adjustCanvas();
		};

		videoCtrl.onDevices = (devices) => {
			selectCameras.innerHTML = "";
			devices.forEach((d, i) => {
				const option = document.createElement("option");
				option.value = d.deviceId;
				option.textContent = d.label || `Cámara ${i+1}`;
				// Seleccionamos la cámara actualmente activa
				if (d.deviceId === videoCtrl.currentDeviceId) {
					option.selected = true;
				}
				selectCameras.appendChild(option);
			});
		};

		videoCtrl.onError = (err) => {
			console.error("❌ Error:", err);
			cameraMessages.innerHTML = `❌ Error al acceder a la cámara: ${err.message}`;
		};

		// Inicialización: cámara trasera en HD
		videoCtrl.init("HD", "environment");


// *********************************************************************************************
		// Instancia del controlador ModelController
		const modelCtrl = new ModelController(videoElement, {
			filterClasses: ['cup', 'bottle'], // Filtrar solo estas clases
			onPredictions: (preds) => {
				console.log("Predicciones recibidas:", preds);
				// Aquí enviarías datos a la capa 3
			}
		});

		// Cargar modelo y empezar
		(async () => {
			await modelCtrl.loadModel();
			// Configurar fps desde UI
			modelCtrl.setThrottle(parseInt(frames.value));
			modelCtrl.start();
		})();



		// *********************************************************************************************
		// Instancia del controlador PredictionProcessor
		const canvasElement = document.getElementById('canvas');
		const processor = new PredictionProcessor(canvasElement);

		// Conectar con Capa 2 (ModelController)
		modelCtrl.onPredictions = (preds) => {
			processor.process(preds);
		};



		// ======================================================
		// EVENTOS DE LA UI
		// ======================================================

		// Enciende/Apaga cámara
		cameraToggleButton.addEventListener("click", async () => {
			if (videoCtrl.isReady()) {
				videoCtrl.stop();
			} else {
				// Si ya había una cámara activa, la usamos; si no, fallback a trasera
				const olddeviceId = videoCtrl.currentDeviceId || null;
				if (olddeviceId) {
					await videoCtrl.switchCamera(olddeviceId, "HD");
				} else {
					await videoCtrl.init("HD", "environment");
				}
			}
			configureCameraUI();
		});

		// Cambio de frame (throttling) dinámicamente según UI
		frames.onchange = (e) => {
			modelCtrl.setThrottle(parseInt(e.target.value));
		};

		// Enciende/Apaga detección
		toggleDetectionButton.onclick = async () => {
			if (modelCtrl.running) {
				modelCtrl.stop();
				toggleDetectionButton.innerHTML = "Encender<br>Detección";
			} else {
				modelCtrl.start();
				toggleDetectionButton.innerHTML = "Apagar<br>Detección";
			}
		};

		// Cambio de cámara
		selectCameras.onchange = async (e) => {
			await videoCtrl.switchCamera(e.target.value, "HD");
			configureCameraUI();
			adjustCanvas();
		};

		// Zoom
		zoomControl.oninput = (e) => {
			videoCtrl.setZoom(e.target.value);
		};

		// Torchalert
		torchBtn.onclick = async () => {
			const state = await videoCtrl.toggleTorch();
			torchBtn.textContent = state ? "Linterna ON" : "Linterna OFF";
		};


		// Se detectan cambios dinámicos de dispositivos
		// Detecta cuando se agregan o quitan cámaras/micrófonos
		// Ej: Si un usuario conecta o desconecta una cámara externa, enumerateDevices() no
		// se vuelve a ejecutar automáticamente.
		navigator.mediaDevices.addEventListener('devicechange', async () => {
			console.log("🔄 Cambio de dispositivos detectado");
			await videoCtrl.getDevices(); // Re-obtenemos la lista y actualizamos el select
		});

	
		// ======================================================
		// CONFIGURACION CENTRAL DE LA UI
		// ======================================================
		function configureCameraUI() {
			// Zoom
			if (videoCtrl.isZoomSupported()) {
				const zoomConfig = videoCtrl.getZoomConfiguration();
				if (zoomConfig) {
					zoomControl.min = zoomConfig.min;
					zoomControl.max = zoomConfig.max;
					zoomControl.step = zoomConfig.step;
					zoomControl.value = zoomConfig.value;
					zoomControl.disabled = false;
					console.log("🔍 Zoom configurado:", zoomConfig);
				} else {
					zoomControl.disabled = true;
					console.warn("⚠️ Este dispositivo no soporta zoom");
				}
			} else {
					zoomControl.disabled = true;
					// Solo mostrar warning si la cámara está activa y no hay zoom
        			if (videoCtrl.isReady()) console.warn("⚠️ Este dispositivo no soporta zoom");
			}

			// Torch
			torchBtn.disabled = !videoCtrl.isTorchSupported();

			// Botón toggle
			cameraToggleButton.innerHTML  = videoCtrl.isReady()
				? "Apagar<br>Cámara"
				: "Encender<br>Cámara";

			// Cambio de cámara
			selectCameras.disabled = !videoCtrl.isReady();


			// Abilita/Desabilita Select de frame (throttling)
			frames.disabled  = videoCtrl.isReady()
				? false
				: true;

			// Abilita/Desabilita Boton detección
			toggleDetectionButton.disabled  = videoCtrl.isReady()
				? false
				: true;
		}

		window.addEventListener("resize", () => {
			if (videoCtrl.isReady()) adjustCanvas();
		});

		window.addEventListener("orientationchange", () => {
			setTimeout(() => {
				if (videoCtrl.isReady()) adjustCanvas();
			}, 300); // pequeño delay para dejar que el video se reajuste
		});


	</script>




	<script>
		const infoButton = document.getElementById("infoButton");
		const cameraInfo = document.getElementById("cameraInfo");
		const closeCameraInfo = document.getElementById("closeCameraInfo");

		// Mostrar panel al hacer click en el botón
		infoButton.addEventListener("click", () => {
			// Obtener información
			const zoomConfig = videoCtrl.getZoomConfiguration();
			const resolution = videoElement.videoWidth + "x" + videoElement.videoHeight;
			const zoomSupported = zoomConfig ? "Sí" : "No";
			const torchSupported = videoCtrl.isTorchSupported() ? "Sí" : "No";
			const deviceId = videoCtrl.currentDeviceId || "-";

			// Construir dinámicamente el contenido
			const contentHTML = `
				<p>Resolución: <span>${resolution}</span></p>
				<p>Zoom soportado: <span>${zoomSupported}</span></p>
				<p>Torch soportada: <span>${torchSupported}</span></p>
				<p>Dispositivo activo: <span>${deviceId}</span></p>
			`;
			document.getElementById("cameraInfoContent").innerHTML = contentHTML;

			// Mostrar panel
			cameraInfo.style.display = "block";
		});

		// Ocultar panel al cerrar
		closeCameraInfo.addEventListener("click", () => {
			cameraInfo.style.display = "none";
		});
	</script>


	<script>
/*

		// INICIO

		document.addEventListener("DOMContentLoaded", function() {
			// Tu código aquí
		});
		document.addEventListener("DOMContentLoaded", () => {
			// Tu código aquí
		});
//*/
	</script>

</body>
</html>
