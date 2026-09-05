# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Qué es este proyecto

Taller de producción de video para secciones **hero** de landing pages que el usuario
construye y vende a clientes. No es una aplicación: no hay build, ni tests, ni
dependencias. El trabajo es generar footage con la **MCP de Higgsfield**, post-procesarlo
con ffmpeg y entregar el MP4 junto con el markup que lo reproduce.

Cada tanda es footage para el sitio de un cliente distinto. Guardá el material por
cliente/rubro, no suelto en la raíz.

## Las dos skills del proyecto

Viven en `.claude/skills/` y son el procedimiento real de trabajo. **Leelas antes de
generar nada**: contienen la estructura de los prompts, las composiciones válidas para
hero y los errores concretos que arruinan cada tipo de clip.

| Skill | Cuándo | Técnica |
|---|---|---|
| `loop-hero-video` | Fondo animado que se repite sin corte | **Una sola imagen** pasada como `start_image` *y* `end_image` |
| `generar-video-scrub` | Video controlado con el scroll (GSAP ScrollTrigger) | **Dos imágenes distintas**: frame inicial → frame final |

El criterio de ruteo es qué hace el video en la landing, no cómo lo nombre el usuario:
si se reproduce solo de fondo → loop; si el usuario lo avanza con la rueda → scrub.
Un pedido vago ("un video para el hero") se resuelve preguntando, no eligiendo.

Las skills globales de **HyperFrames** son para componer y renderizar HTML animado. Acá
el video es footage fotorrealista generado por modelo. No las mezcles salvo que el
usuario pida montar el footage dentro de una composición.

## Regla que gobierna todo: no gastar créditos sin aprobación

Las dos skills la declaran como regla número uno y no admite atajos. Cada generación es
dinero real del usuario.

- Preflight obligatorio con `get_cost: true` antes de cada llamada, y decir el número.
- Una generación por turno. Nunca encadenar imagen → video en la misma respuesta.
- `balance` al principio y al final de la tanda; informar lo gastado.
- Única excepción: el usuario dice explícitamente "hacé todo de corrido". Confirmalo una
  vez y seguí.

En `generar-video-scrub` hay un paso más que **no es opcional**: bajar los dos frames y
mirarlos antes de lanzar el video. Un problema de composición cuesta 1,5 créditos
arreglarlo ahí y 28 después.

## Contrato con la MCP de Higgsfield

Parámetros verificados contra el catálogo (`models_explore action:"get"`). Si algo falla,
verificá de nuevo ahí antes de suponer que la skill está mal.

**Imagen** — `generate_image`, modelo `gpt_image_2`: `resolution` 1k/2k/4k, `quality`
low/medium/high, rol de referencia `image`. Para el frame final del scrub se pasa el
frame inicial como `medias: [{ role: "image", value: "<job_id>" }]` — la referencia no
suma costo.

**Video** — `generate_video`, modelo `kling3_0`: `duration` 3-15, `mode` std/pro/4k,
roles `start_image` / `end_image`. **10 créditos** los 8 s en `std`. Es el default para
scrub desde sept 2026, medido contra Veo 3.1 Lite y Seedance en `taller-carpinteria/`:
ganó en las siete métricas y entrega el movimiento ya lineal. No expone `resolution`.
`sound` viene en `"on"` → mandar `"off"`.

**Fallback** — `seedance_2_0`: `duration` 4-15, mismos roles, **28** créditos en 720p.
Más caro y peor en todo lo medido, pero es el que tiene historia en `cafe-londres/` y
`vivero/`, y expone `resolution`, así que 480p baja los 8 s a **12** para iterar barato.
Dos defaults suyos van en contra de lo que necesita un hero:

- `generate_audio` viene en `true` → un hero va mudo, mandar `false`.
- `mode` viene en `std` → mandar `"fast"`; y `fast` solo admite 480p/720p, así que pedir
  1080p con `fast` no funciona.

**Para iterar, bajá resolución, nunca duración.** 480p no cambia si el modelo corta,
alucina o deforma; un clip más corto tiene que cubrir la misma distancia en menos tiempo
y hace *más* probable el corte.

`medias[].value` lleva un `job_id` o `media_id`, **nunca una URL**.

**Preset fantasma:** el servidor recomienda `24bae836-2c4a-48e0-89b6-49fcc0b21612`
("IN THE DARK") ante casi cualquier prompt largo de cámara, sin relación con la escena.
Es un falso positivo conocido: rechazarlo con `declined_preset_id` y avisarle al usuario
en una línea.

**`use_unlim`:** si se omite y el usuario tiene allowance, la llamada **no genera** —
devuelve `unlim_choice`, que es una pregunta para trasladarle al usuario. No la contestes
por tu cuenta: encaja con la regla de aprobación previa.

## Post-proceso con ffmpeg

El MP4 que entrega Higgsfield tiene keyframes muy espaciados. Para scrub hay que
reencodear con un keyframe por cuadro o el video salta al arrastrar:

```bash
ffmpeg -i input.mp4 -c:v libx264 -g 1 -crf 20 -pix_fmt yuv420p -an \
  -movflags +faststart output.mp4
```

Esperá que el peso suba fuerte (un clip de 8s en 720p pasa de ~2 MB a 10-15 MB). Es el
precio del scrub fluido y no hay alternativa.

Para un loop **no** hace falta `-g 1`. Si el corte se nota igual, recortá unas décimas
del final o meté un crossfade corto entre final y principio.

### El ease-in-out depende del modelo — medir siempre, corregir sólo si hace falta

Pico de velocidad sobre el promedio, medido: **Seedance 2.0 → 6,6×** (hay que
relinealizar siempre), **Veo 3.1 Lite → 1,52×** (hay que hacerlo: los extremos caían a
0,34×), **Kling 3.0 → 1,11×** (no hace falta). El criterio: el tramo más rápido cerca de
2× o menos, el más lento no por debajo de ~0,5×. **Si ya cumple, no lo toques** — el
remapeo mezcla cuadros y cuesta nitidez.

Lo que sigue vale cuando el clip no cumple. Verificado midiendo cuadro por cuadro
(`cafe-londres/`, sept 2026): en un clip de 8 s, Seedance concentró el cambio entre los
segundos 2 y 5. El primer segundo y medio y los últimos dos y medio eran cuadros casi
idénticos, y el tramo más rápido avanzaba **6,6 veces** más que el promedio.

En reproducción normal se lee como un ease natural. **Con scrub es un defecto**: el
usuario scrollea sin que pase nada, después todo de golpe, después nada. Es el punto 6
de la lista de verificación de `generar-video-scrub`, y no se ve mirando el clip en el
widget.

Se arregla sin gastar créditos y sin regenerar. El método:

1. Extraer los cuadros y medir el avance real del cambio en cada uno. Para un rack focus
   sirve la nitidez media (gradiente) de la zona que entra en foco; para una
   transformación, la diferencia contra el primer cuadro.
2. Monotonizar la curva (el grano mete ruido) y normalizarla a 0-1.
3. Remuestrear a progresión pareja: para N cuadros de salida, interpolar la posición
   fuente de cada objetivo `j/(N-1)` con `np.interp`, y **mezclar los dos cuadros que lo
   bracketean** en vez de repetir el más cercano — repetir produce micro-tirones.
4. Reencodear la secuencia con `-g 1`.

En el caso medido, el tramo más rápido bajó de 6,6× a 2,0× el promedio. `hero-raw.mp4`
tenía **1 solo keyframe en 193 cuadros**, así que el `-g 1` no es opcional.

Guardá siempre el MP4 crudo antes de retocarlo: el retiming es destructivo y regenerar
cuesta 28 créditos.

## Entrega

```html
<!-- loop -->
<video src="hero.mp4" autoplay loop muted playsinline></video>

<!-- scrub: sin autoplay, o se reproduce solo en vez de scrubearse -->
<video src="hero.mp4" muted playsinline preload="auto"></video>
```

`muted` y `playsinline` no son opcionales: sin ellos iOS bloquea el autoplay.

**Geometría cortada por el borde = alucinación.** Cualquier arquitectura que el cuadro
corta —una ventana sin su alféizar, una puerta a medias— es donde el modelo inventa al
avanzar la cámara, porque tiene que completarla y no tiene de dónde. En
`taller-carpinteria/` una ventana pegada al borde izquierdo se convirtió en una puerta
vidriada abierta, justo encima de la zona del titular. Lo que está cortado por el borde,
o entra entero en cuadro, o no está: se arregla por 1,5 rehaciendo el frame.

Titulares, CTA, logos y etiquetas van **siempre** encima en HTML o SVG, nunca dentro de
la imagen generada — el modelo escribe texto mal de forma sistemática. Por eso los
prompts de las skills prohíben texto y carteles, y por eso las composiciones reservan
franja superior limpia (navbar) y un tercio lateral de aire (titular).

## Límite del entorno

Si el CDN de Higgsfield está bloqueado por la política de red, no vas a poder ver las
imágenes ni los videos que generás (`curl` devuelve un `x-deny-reason`). Decíselo al
usuario **de entrada** y pedile que suba los archivos al chat: adjuntos sí se ven.
Describí lo que pediste, no lo que "ves". Nunca afirmes que algo quedó bien sin haberlo
mirado.
