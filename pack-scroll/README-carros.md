# README-carros.md — animaciones pendientes

CA1 clonó `pack-scroll/` de `utec-motos-landing` (commit `972b233` BA1) para CARROS PRO.
Las 3 animaciones que eran específicas de moto se reemplazaron por un placeholder visible
(fondo negro, texto "Animación de carro — en producción"), pero el **contenedor/canvas y el
mecanismo JS de scroll-scrub siguen intactos y comentados** dentro de `index.html`, listos
para recibir material real de carro sin tocar CSS ni JS otra vez. Esta nota documenta
exactamente qué archivos espera cada una.

## 1) Despiece grande (`#despiece`, `<canvas id="dp-canvas">`)

- Carpetas esperadas (mismos nombres que en motos, dentro de este mismo repo así que no
  hay colisión): `pack-scroll/img/dp-frames/mob/` y `pack-scroll/img/dp-frames/desk/`.
- Archivos: `f_000.webp` a `f_0NN.webp` (padding de 3 dígitos — `f_'+String(i).padStart(3,'0')+'.webp'`).
  En motos eran 71 frames (`f_001.webp`...`f_071.webp`, el índice empieza en 1, no en 0).
  Ajustar la cantidad real al exportar y actualizar `n:71` en la llamada `initEscena(...)`
  si el conteo final es distinto.
- Canvas fijo a `width="1280" height="732"` (relación 1280/732, ver `.dp-stage canvas{aspect-ratio:1280/732}`).
- **Para reactivar:** en `index.html`, dentro de `<script>` al final, hay un bloque JS
  comentado con `/* CA1: despiece (carro grande + motor) ... */` que contiene `function
  initEscena(cfg){...}`, `function initMotor(cfg){...}` y las dos llamadas finales
  `initEscena({stage:'dp-stage',...})` / `initMotor({stage:'scr-motor-stage',...})`.
  Borrar el `/*` de apertura (línea con "CA1: despiece...") y el `*/` de cierre (justo
  antes de `</script>`) para volver a activar ambas animaciones de una vez.
- También borrar el `<div class="anim-ph">...</div>` que está pegado al `<canvas
  id="dp-canvas">` dentro de `#dp-stage` (y el de `#scr-motor-stage`), para que el canvas
  vuelva a verse.

## 2) Motor (`#escena-motor`, `<canvas id="scr-motor-canvas">`)

- Carpetas esperadas: `pack-scroll/img/scroll-frames/motor/mob/` y
  `pack-scroll/img/scroll-frames/motor/desk/`.
- Archivos: `f_000.webp`... En motos eran 137 frames (`n:137`) con una LUT de remuestreo
  por longitud de arco (`LUT_MOTOR`, ya en el bloque comentado) — esa LUT es específica de
  la cadencia de apertura de un motor de moto; si el motor de carro se abre con un ritmo
  visual distinto, lo más simple es borrar `lut:LUT_MOTOR` de la llamada final (`initMotor`
  usa `lutPos(p)` que cae de vuelta a progreso lineal 0..1 si no hay LUT... revisar
  `lutPos()` antes de asumirlo — si no, generar una LUT nueva midiendo los frames reales,
  mismo método que la nota ORDEN AY9 ya deja documentado ahí mismo).
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
