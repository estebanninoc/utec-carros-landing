# README-carros.md — animaciones pendientes

CA1 clonó `pack-scroll/` de `utec-motos-landing` (commit `972b233` BA1) para CARROS PRO.
Las 3 animaciones que eran específicas de moto se reemplazaron por un placeholder visible
(fondo negro, texto "Animación de carro — en producción"), pero el **contenedor/canvas y el
mecanismo JS de scroll-scrub siguen intactos y comentados** dentro de `index.html`, listos
para recibir material real de carro sin tocar CSS ni JS otra vez. Esta nota documenta
exactamente qué archivos espera cada una.

## 1) Despiece grande (`#despiece`, `<canvas id="dp-canvas">`) — LIVE desde ORDEN CA2

- Ya NO es placeholder: usa el despiece real del motor (Higgsfield minimax_h3_max,
  `carro-motor-despiece.mp4` en `/opt/guardian-shared/fabrica/carros-c/vid-hf/`), originalmente
  145 cuadros interpolados (ffmpeg `minterpolate`) y remuestreados por longitud de arco
  (`LUT_DESPIECE` en el `<script>`, mismo método que `LUT_MOTOR` documentaba para motos).
  ORDEN CA7 (perf) los bajó a 110 cuadros (subsample por índice viejo, LUT re-escalada al
  nuevo rango 1..110 preservando el mismo ritmo) para entrar en presupuesto de peso.
- Frames en `pack-scroll/img/dp-frames/desk/` (1024×683, 110 archivos, ~7.7MB) y
  `pack-scroll/img/dp-frames/mob/` (660×440, 110 archivos, ~3.7MB), `f_001.webp`...`f_110.webp`.
  ORDEN CA11 los re-extrajo del mismo `.mp4` (mismo índice nativo por cada `f_NNN`, matcheado
  por MSE contra el frame viejo para no tocar recorte/ritmo/LUT) a mayor resolución/calidad
  (libwebp q40 mob / q55 desk, método 6) — CA7 los había bajado a 440/800px q=20 por peso, pero
  eso se veía "muy GIF" en celular (DPR3 estira 440px ~2.6x) y con bloques visibles. CA7 también
  quitó el `preloadAll()` de `initMotor()`: ahora carga por `IntersectionObserver` (keyframes
  cada 4 desde CA11 + relleno con `requestIdleCallback`), ver comentario ORDEN CA7/CA11 junto a
  `initMotor` en `index.html`.
- Usa `initMotor()` (NO `initEscena()`): sin pin/sticky, igual que `#escena-motor` en motos
  (AY6-AY9 + AZ2/AZ3/AZ5) — `initEscena()` se quitó del script por quedar sin uso.
- CSS: `.dp-stage{max-width:900px}`, `.dp-stage canvas{aspect-ratio:1152/768}` (video horizontal;
  en motos esta sección era 1280/732). `#dp-bar` se quitó del HTML (`initMotor` no lo usa).

## 2) Arnés eléctrico (`#escena-motor`, `<canvas id="scr-motor-canvas">`) — LIVE desde ORDEN CA10

- Ya NO es placeholder: usa el arnés eléctrico real en rayos X (Higgsfield minimax_h3_max,
  `carro-arnes-rayosx.mp4` en `/opt/guardian-shared/fabrica/carros-c/vid-hf/`, 124 cuadros
  nativos 24fps, sin interpolar). El tramo final (105-124) casi no cambia (conectores ya
  flotando), así que se quedó a la mitad por muestreo: 104 cuadros completos (1-104) + 11 del
  tramo final (105,107,...,123,124) = 115 cuadros, remuestreados por longitud de arco
  (`LUT_ARNES`, mismo método que `LUT_DESPIECE` pero medido sobre esta secuencia propia).
- Frames en `pack-scroll/img/arnes-frames/desk/` (1100×538, 115 archivos, ~7.6MB) y
  `pack-scroll/img/arnes-frames/mob/` (700×342, 115 archivos, ~3.4MB), `f_001.webp`...`f_115.webp`.
  Re-extraídos en ORDEN CA11 con el mismo estándar de calidad que el despiece (libwebp q40 mob /
  q55 desk, método 6, mismos índices nativos de `f_NNN` matcheados por MSE); antes estaban a
  495/900px q=20 igual que el despiece pre-CA11. Mismo mecanismo de carga diferida IO+idle
  (keyframes cada 4 desde CA11) de CA7.
- Sin bbox-crop: la unión de todos los cuadros toca casi todos los bordes (los conectores
  llegan al borde derecho en el final) — recortar se habría comido piezas reales.
- Video horizontal (1440×704, ~2:1), a diferencia de las escenas chicas portrait de moto que
  originaron `.dp-stage-mini`: el canvas se pisa por id (`#scr-motor-stage`) con el mismo
  patrón full-width/aspect-ratio de `.dp-stage` (`#despiece`) en vez del `max-height:vh`
  heredado, para que en móvil sea horizontal de borde a borde como el motor grande.
- Usa `initMotor()` (la misma función de la sección 1, segunda llamada al final del `<script>`).
- Chips `.dp-piezas` en español (Arnés principal, Conectores, Caja de fusibles, Computadora
  (ECU), Relés, Sensores, Tierras, Luces): estáticos bajo el canvas, igual que en motos (ahí
  tampoco hay lógica de mostrar/ocultar por progreso de scroll, solo lista fija).

## 3) Carrusel del hero (mockup del celular, `.device .pv`)

No es scroll-scrub sino 3 clips de video con crossfade CSS puro (`@keyframes dvSlide` +
`animation-delay`). Hoy `.pv` solo tiene el placeholder `<div class="anim-ph">`. Para
reactivar, reemplazar ese div por esta estructura (idéntica a la de motos, solo cambiando
`vid/g1.mp4` → `vid/c1.mp4`, etc., y los textos de `tag`/`cap` por los reales de carro):

```html
<div class="slide s1">
  <video src="vid/c1.mp4" poster="vid/c1.webp" muted loop playsinline autoplay preload="auto"></video>
  <span class="tag">[nombre de la pieza]</span>
  <div class="cap">[Título del clip]<small>[Descripción corta]</small></div>
</div>
<div class="slide s2">
  <video src="vid/c2.mp4" poster="vid/c2.webp" muted loop playsinline autoplay preload="auto"></video>
  <span class="tag">[nombre de la pieza]</span>
  <div class="cap">[Título del clip]<small>[Descripción corta]</small></div>
</div>
<div class="slide s3">
  <video src="vid/c3.mp4" poster="vid/c3.webp" muted loop playsinline autoplay preload="auto"></video>
  <span class="tag">[nombre de la pieza]</span>
  <div class="cap">[Título del clip]<small>[Descripción corta]</small></div>
</div>
```

- Archivos esperados: `pack-scroll/vid/c1.mp4` + `c1.webp` (poster), `c2.mp4`+`c2.webp`,
  `c3.mp4`+`c3.webp`. Clips livianos (en motos pesaban ~130 KB cada uno).
- Opcional: la franja `.tl .thumbs` (mini-thumbnails con barra de progreso debajo del
  video) también se quitó por completo junto con los videos. Si se reactiva el carrusel,
  se puede restaurar copiando el bloque `.tl` de `utec-motos-landing/pack-scroll/index.html`
  (líneas ~487-493 del commit 972b233), apuntando los `background-image` a `vid/c1.webp`,
  `vid/c2.webp`, `vid/c3.webp`.
- No hace falta tocar JS: el `forEach` que llama `.play()` sobre `.device video` ya está
  activo y simplemente no encuentra elementos hasta que se agreguen los `<video>`.

## Otros pendientes de contenido (no son animaciones, ver `[PENDIENTE]` en `index.html`)

Cifras, lista de marcas, nombres de bonos y contenido del curso quedan marcados con
`<span class="pend">[PENDIENTE]</span>` en el HTML — buscar `pend` en `index.html` para
encontrarlos todos. El precio US$12.99 y la garantía de 7 días quedaron **heredados de
motos tal cual**, marcados con comentarios `<!-- PROVISIONAL: confirmar con Esteban -->`.
