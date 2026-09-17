# Instalacion Interactiva de Agua y Tiburones

Instalacion audiovisual interactiva en TouchDesigner: una vista aerea de agua
oscura con tiburones 3D nadando de forma autonoma, sobre un efecto de agua
psicodelico tipo feedback que reacciona al movimiento de las personas frente
a un sensor Kinect.

## Que hace el proyecto

- **Deteccion de movimiento**: lee la imagen de profundidad de un sensor
  Kinect y calcula una mascara de movimiento por diferencia de frames
  (adaptado para GPUs que no soportan el Optical Flow nativo de Nvidia).
- **Agua reactiva**: un sistema de feedback (blur + edge + zoom + mezcla
  dry/wet) que genera ondas y un efecto psicodelico tipo "trail" que
  reacciona al movimiento detectado y decae solo con el tiempo.
- **Tiburones 3D**: 4 modelos de tiburon (malla suavizada en Blender) que
  nadan de forma autonoma con rumbo propio, orientandose siempre hacia su
  direccion de avance, virando de vuelta al acercarse al borde del area
  (nunca desaparecen de la pantalla).
- **Render final**: todo compuesto en una sola imagen (agua + tiburones)
  lista para proyectar.

## Requisitos

### Software
- **TouchDesigner 2025.32460 (Non-Commercial o Commercial)** o superior.
  Version recomendada: la misma build (2025.32460) para evitar diferencias
  de comportamiento en operadores como Feedback TOP/POP y Random POP.
- Sistema operativo: Windows 10/11 (el proyecto fue desarrollado y probado
  en Windows 10).

### Hardware
- GPU: cualquier GPU compatible con TouchDesigner. **No hace falta una
  GPU Nvidia RTX 3000 o superior** (el proyecto evita a proposito el
  Optical Flow TOP nativo, que si requiere esa gama, y usa diferencia de
  frames en su lugar). Probado en NVIDIA GTX 1660 Super.
- CPU: cualquier procesador moderno (probado en AMD Ryzen 7).
- RAM: 8 GB o mas recomendado.

### Dispositivos externos
- **Sensor Kinect** (v1 o v2). Si usas Kinect v2, necesitas instalar el
  **Kinect SDK v2.0 (Build 1410)** de Microsoft. Sin el sensor conectado,
  el proyecto sigue funcionando (el agua queda en reposo, sin reaccionar),
  pero no habra interaccion.

## Como abrir y ejecutar

1. Instalar TouchDesigner (ver version recomendada arriba).
2. Si vas a usar el sensor Kinect v2, instalar el Kinect SDK v2.0 antes de
   abrir el proyecto.
3. Abrir el archivo `proyecto claude+touchdesinger.toe` con TouchDesigner.
4. Los assets externos (modelo 3D de tiburon) estan incluidos en la carpeta
   `assets/` con rutas relativas al .toe, no deberia hacer falta
   reconfigurar nada.
5. Para ver el resultado final en una ventana dedicada: usar el boton de
   **Perform Mode** de TouchDesigner (arriba a la izquierda, debajo del
   menu File). Esta configurado para mostrar el render final
   (`/project1/FINAL_RENDER/out1`).
6. Revisar la consola de TouchDesigner (barra de errores) al abrir: si el
   Kinect no esta conectado o falta el SDK, va a aparecer un error en el
   componente `KINECT_INPUT/kinect1`; esto es esperado si no tenes el
   sensor a mano y no impide ver el resto de la instalacion funcionando
   (el agua y los tiburones se ven igual, solo sin interaccion).

## Estructura del proyecto (dentro del .toe)

- `KINECT_INPUT` -- lectura del sensor Kinect (imagen de profundidad).
- `OPTICAL_FLOW` -- deteccion de movimiento por diferencia de frames.
- `container1/feedbackEdgepppp` -- sistema de agua (feedback + edge +
  zoom + mezcla dry/wet).
- `FISH_SIMULATION` -- sistema de peces/tiburones procedurales en GPU
  (partículas POP), usado en etapas iniciales del proyecto.
- `FINAL_RENDER` -- composicion final: agua + los 4 tiburones 3D
  (`rig1` a `rig4`) + render y salida (`out1`).

## Configurar parametros importantes

Todos los parametros de control estan expuestos como paginas de
parametros custom en sus respectivos componentes:

- **KINECT_INPUT** (pagina "Kinect"): `Enable`, `Smoothing`,
  `Movementthreshold`.
- **OPTICAL_FLOW** (pagina "Optical Flow"): `Flowstrength` (sensibilidad
  al movimiento), `Scale`, `Threshold` (umbral minimo para considerar que
  hay movimiento).
- **container1/feedbackEdgepppp** (pagina "Feedback"): `Edgecolor`
  (color del efecto), `Scale` (velocidad de zoom del feedback),
  `Feedbackgain` (que tan rapido decae el efecto), `Drywet` (mezcla entre
  imagen cruda y efecto procesado).
- **FINAL_RENDER/shark_ai** (Execute DAT, dentro del codigo): velocidad,
  frecuencia de giro y fase de cada tiburon se configuran en el diccionario
  `rigs_cfg` al principio del script. El radio de patrullaje se controla
  con `SOFT_BOUND` / `HARD_BOUND`.
- **FINAL_RENDER/light_shark**: intensidad (`Dimmer`) y color de la luz
  que ilumina a los tiburones.

## Notas

- El proyecto incluye un componente `mcp_webserver_base` usado durante el
  desarrollo para permitir edicion asistida por IA (Claude) via API. No es
  necesario para el funcionamiento de la instalacion y se puede eliminar
  con seguridad si se va a distribuir el proyecto de forma independiente.
