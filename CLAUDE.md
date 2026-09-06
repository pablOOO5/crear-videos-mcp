# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Qué es este proyecto

Taller de producción de video para secciones **hero** de landing pages que el usuario
construye y vende a clientes. No es una aplicación: no hay build, ni tests, ni
dependencias. El trabajo es generar footage con la **MCP de Higgsfield**, post-procesarlo
con ffmpeg y entregar el MP4 junto con el markup que lo reproduce.

Cada tanda es footage para el sitio de un cliente distinto. Guardá el material por
cliente/rubro, no suelto en la raíz.

**El repo guarda el procedimiento, no los archivos.** Los binarios de media están en
`.gitignore`: el negocio consume los videos por URL, y todo lo generado con Higgsfield
se recupera con su `job_id`. Cada carpeta de cliente lleva un **`MANIFIESTO.md`** con
los `job_id` de cada imagen y cada video, cuáles se descartaron y por qué, el comando
para reconstruir el MP4 final, y las mediciones del par. Ver
`taller-carpinteria/MANIFIESTO.md` como plantilla.

Tres cosas que no se pueden olvidar al hacerlo:

- **El archivo que sirve el CDN no se puede scrubear**: es el crudo, con un solo
  keyframe. El que va a la landing es el reencodeado con `-g 1`, que es un artefacto de
  build y se sube al hosting del cliente, no acá.
- **Si entra material propio del usuario a una tanda, no tiene `job_id` y no se
  recupera.** Decíselo y pedile que lo respalde aparte antes de untrackearlo.
- **`cafe-londres/` y `vivero/` siguen con la media versionada a propósito**: son
  anteriores a esta convención y no tienen manifiesto, así que sacarles los archivos
  los volvería irrecuperables. Si querés aplicarles la misma regla, primero hay que
  reconstruirles el manifiesto con los `job_id`.

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
mirarlos antes de lanzar el video. Un problema de composición cuesta 1 crédito
arreglarlo ahí y 10 después.

## Contrato con la MCP de Higgsfield

Parámetros verificados contra el catálogo (`models_explore action:"get"`). Si algo falla,
verificá de nuevo ahí antes de suponer que la skill está mal.

**Imagen** — `generate_image`, modelo `gpt_image_2`: `resolution` 1k/2k/4k, `quality`
low/medium/high, rol de referencia `image`. **1 crédito** en 1k/medium/16:9 — bajó de
1,5 (sept 2026). Para el frame final del scrub se pasa el frame inicial como
`medias: [{ role: "image", value: "<job_id>" }]` — la referencia no suma costo.

**Video** — `generate_video`, modelo `kling3_0`: `duration` 3-15, `mode` std/pro/4k,
roles `start_image` / `end_image`. **10 créditos** los 8 s en `std`. Es el default para
scrub desde sept 2026, medido contra Veo 3.1 Lite y Seedance en `taller-carpinteria/`:
ganó en las siete métricas. **Entrega el movimiento ya lineal sólo en dolly** — en
transformación con cámara fija hay que relinealizar igual, ver abajo. No expone
`resolution`.
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

Pico de velocidad sobre el promedio, medido: **Seedance 2.0 → 6,6×** y **Veo 3.1 Lite
→ 1,52×**, los dos en dolly interior, los dos hay que relinealizarlos. **Kling 3.0 →
1,11× en dolly, pero 2,75× y 3,41× en transformación con cámara fija**, con la cola
cayendo a 0,02× (`joyeria-alianzas/`). **Depende del plano, no sólo del modelo: medí
siempre, no hay atajo por modelo.** El criterio: el tramo más rápido cerca de 2× o
menos, el más lento no por debajo de ~0,5×. **Si ya cumple, no lo toques** — el
remapeo mezcla cuadros y cuesta nitidez.

**Medí siempre sobre cuadros filtrados** (achicar a ~320 px + `GaussianBlur(1.5)`),
también al verificar. Con grano crudo la mezcla de cuadros descorrelaciona el grano e
infla la medición justo en la cola: el mismo clip daba pico 3,15× crudo y 2,18×
filtrado. Y en clips oscuros **no restes el piso de grano** — el paso está en la skill
como indispensable y ahí sobrecorregía de 1,37× a 6,4×.

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
o entra entero en cuadro, o no está: se arregla por 1 rehaciendo el frame.

Titulares, CTA, logos y etiquetas van **siempre** encima en HTML o SVG, nunca dentro de
la imagen generada — el modelo escribe texto mal de forma sistemática. Por eso los
prompts de las skills prohíben texto y carteles, y por eso las composiciones reservan
franja superior limpia (navbar) y un tercio lateral de aire (titular).

**El aire que reservás para el texto es donde el modelo va a meter lo nuevo.** Es el
error más caro de `joyeria-alianzas/` y no estaba en ninguna lista. Durante una
transformación el modelo tiene que traer lo que aparece por algún lado, y usa el
espacio vacío: con el aire a la izquierda y un frame final donde la manga del novio
entraba desde la izquierda, su mano cruzó la zona del titular y la dejó en **p95 192
durante tres de los ocho segundos**. Prohibirlo en el prompt no alcanza —se lo pidió
dos veces como CRITICAL— porque no es desobediencia: no tiene otro lugar de donde
traerlo.

- **Lo que entra en cuadro tiene que entrar por el lado opuesto al aire.** Se ve en el
  frame final antes de gastar: preguntate por dónde tiene que entrar cada cosa nueva.
  Cuesta 1 arreglarlo ahí y 10 después.
- **Verificá las dos zonas cuadro por cuadro, no en los dos frames extremos.** Los dos
  videos descartados tenían las zonas limpias en el `start_image` y en el `end_image` y
  rotas en el medio. No se ve en el widget. Es el punto 7 de la lista de la skill.
- **Y esperá que el problema se mude, no que desaparezca.** Al sacar las manos del
  tercio izquierdo pasaron a cruzar la franja de la navbar. Eso se tapa gratis con un
  velo en CSS; el re-roll se guarda para cuando la zona rota es la del titular.

**El piso de "vale la pena scrubearlo" no lo cubre la escala aparente si la cámara no
se mueve.** Ahí lo mide el **delta de p99 y de %pix>120 entre los dos frames**: un par
con 113 → 114 produjo un clip impecable que se rechazó por parecer una foto quieta; el
aprobado fue 113 → 213. No uses el brillo medio, que da parecido en los dos casos.

## Límite del entorno

**Probá el `curl` antes de suponer nada.** El CDN de Higgsfield *puede* estar bloqueado
por la política de red, pero **no siempre lo está**: en la tanda de `taller-carpinteria/`
(sept 2026) bajaron sin problema las 5 imágenes y los 2 videos, todos `http=200`. Avisar
"no puedo ver lo que genero" de entrada le hace hacer al usuario un trabajo que
probablemente no haga falta.

El orden correcto es: bajar el archivo, y **sólo si `curl` falla con `x-deny-reason`**,
decírselo al usuario y pedirle que lo suba al chat — los adjuntos sí se ven.

Lo que no cambia: describí lo que pediste, no lo que "ves", y **nunca afirmes que algo
quedó bien sin haberlo mirado**.
