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
- Frames en `pack-scroll/img/dp-frames/desk/` (800×533, 110 archivos, ~3.1MB) y
  `pack-scroll/img/dp-frames/mob/` (440×293, 110 archivos, ~1.3MB), `f_001.webp`...`f_110.webp`,
  re-encodeados a libwebp q=20 (ffmpeg `-c:v libwebp`). CA7 también quitó el `preloadAll()`
  de `initMotor()`: ahora carga por `IntersectionObserver` (keyframes cada 8 + relleno con
  `requestIdleCallback`), ver comentario ORDEN CA7 junto a `initMotor` en `index.html`.
- Usa `initMotor()` (NO `initEscena()`): sin pin/sticky, igual que `#escena-motor` en motos
  (AY6-AY9 + AZ2/AZ3/AZ5) — `initEscena()` se quitó del script por quedar sin uso.
- CSS: `.dp-stage{max-width:900px}`, `.dp-stage canvas{aspect-ratio:1152/768}` (video horizontal;
  en motos esta sección era 1280/732). `#dp-bar` se quitó del HTML (`initMotor` no lo usa).

## 2) Motor (`#escena-motor`, `<canvas id="scr-motor-canvas">`)

- Carpetas esperadas: `pack-scroll/img/scroll-frames/motor/mob/` y
  `pack-scroll/img/scroll-frames/motor/desk/`.
- Archivos: `f_000.webp`... `initMotor()` ya está activo en el `<script>` (lo usa `#despiece`,
  ver sección 1) — para esta escena solo hace falta una segunda llamada
  `initMotor({stage:'scr-motor-stage',canvas:'scr-motor-canvas',n:<N>,lut:LUT_<...>,
  dirMob:'img/scroll-frames/motor/mob/',dirDesk:'img/scroll-frames/motor/desk/'})` (ver el
  comentario que queda justo después de la llamada de `#despiece`). Generar una LUT propia
  midiendo los frames reales de ESTA escena (mismo método que `LUT_DESPIECE`/`LUT_MOTOR` de
  motos, por longitud de arco) — no reusar `LUT_DESPIECE`, es específica de la cadencia de
  ese otro video. Sin LUT, `lutPos(p)` cae a progreso lineal 0..1 (revisar que siga siendo
  aceptable antes de asumirlo).
- Canvas con `width="784" height="1361"` como tamaño inicial, pero `initMotor()` lo
  redimensiona solo via `devicePixelRatio` al tamaño real renderizado — no hace falta que
  los frames exportados sean exactamente 784×1361, solo mantener su proporción.

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
