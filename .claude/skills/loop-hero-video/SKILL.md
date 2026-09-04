---
name: loop-hero-video
description: Crea videos de loop infinito con Higgsfield (Seedance 2.0 + GPT Image 2) para usar como hero animado en landing pages. Usar cuando el usuario pida un video loop, loop infinito, video en bucle, hero animado, video de fondo para una landing, o pida convertir una imagen en un video que se repita sin corte. Cubre la técnica de start/end frame idéntico, la composición para navbar y titulares, y el markup HTML para reproducirlo en bucle.
---

# Video de loop infinito para hero de landing page

Genera un video corto que se reproduce en bucle sin que se note el corte, pensado
para usarse como fondo animado de la sección hero de una landing page.

## REGLA NÚMERO UNO: confirmar cada paso

**Nunca gastes créditos sin aprobación explícita del usuario.** Cada generación
cuesta dinero real. El flujo es una conversación entre el usuario y Higgsfield,
y vos sos el intermediario, no el que decide.

Antes de CADA llamada a `generate_image` o `generate_video`, usá `AskUserQuestion`
para que el usuario confirme. Después de cada generación, mostrá el resultado con
`job_display` y esperá su veredicto antes de seguir.

Los puntos de control obligatorios son:

1. **Antes de arrancar** — confirmar rubro/escena, duración y costo total estimado
2. **Después de generar la imagen** — mostrarla y esperar aprobación o pedido de cambios
3. **Antes de generar el video** — confirmar que se lanza y con qué duración
4. **Después del video** — mostrarlo y ofrecer rehacerlo si algo falló

Si el usuario pide cambios en la imagen, regenerá y volvé a preguntar. Repetí el
ciclo las veces que haga falta. No avances "para ahorrar tiempo": el usuario
prefiere una pregunta de más antes que 28 créditos tirados.

La única excepción es que el usuario diga explícitamente "hacé todo de corrido"
o "no me preguntes más". Ahí confirmá esa decisión una vez y seguí.

## Por qué funciona la técnica

Un loop se ve perfecto cuando **el último cuadro es idéntico al primero**. Eso se
consigue pasando **la misma imagen como `start_image` y como `end_image`**:

```
medias: [
  { role: "start_image", value: "<job_id de la imagen>" },
  { role: "end_image",   value: "<el MISMO job_id>" }
]
```

El resto del trabajo es elegir qué se mueve. Y ahí hay una sola regla que decide
si el loop sale o no:

> **Solo se mueve lo que no tiene estado final.**

Agua corriendo, llamas, humo, vapor, olas, hojas con viento, partículas flotando,
destellos, reflejos ondulando. Todos vuelven solos a donde empezaron. Nada de eso
"termina" en ningún lado.

## Los tres errores que arruinan un loop

**1. El sujeto se desplaza dentro del cuadro.**
Si la cámara acompaña a algo que avanza (un auto, un camión, una persona
caminando), cualquier diferencia mínima de velocidad se acumula y el sujeto
deriva. Al reiniciar el video, el salto es evidente.
→ Cámara clavada en trípode y sujeto quieto. El mundo se mueve, el sujeto no.

**2. El tiempo avanza.**
Un atardecer que oscurece, carne que se dora, brasas que se consumen, una vela
que se acorta, una estela que se acumula. El último cuadro ya no puede coincidir
con el primero.
→ Prohibí explícitamente la progresión temporal en el prompt.

**3. Cambia la luz.**
Cualquier variación global de exposición o de dirección de sombras delata el
empalme, aunque todo lo demás esté quieto.
→ Pedí que la luz, la exposición y el grade sean idénticos del primer al último
cuadro.

## Varios focos de movimiento

Un loop con un solo elemento animado se siente pobre y el ojo encuentra el corte
enseguida. Repartí **cuatro a seis movimientos independientes** por distintas
zonas del cuadro. El ojo salta entre ellos y nunca se fija en el punto de
reinicio.

Ejemplos por rubro:

| Rubro | Focos de movimiento |
|---|---|
| Spa | agua cayendo, ondas, vapor, velas, sahumerio, hojas, espejo de agua |
| Parrilla | brasas latiendo, llamas, humo, grasa goteando, chispas, ceniza |
| Marina | olas, destellos, reflejos, barcos meciéndose, banderines, nubes |
| Inmobiliaria | ondas en pileta, agua del borde infinity, fogonero, pastos, nubes |
| Agro | grano cayendo, polvo, hojas con brisa |
| Joyería | destellos en las facetas, partículas en el haz, llama, seda, reflejos |

Si el rubro no tiene movimiento natural (joyería, retail, consultorio), hay que
construirlo: partículas flotando en un haz de luz, una vela, una tela con brisa,
reflejos sobre una superficie pulida.

## Composición para hero

El video va detrás de la navbar y del titular, así que la imagen tiene que dejar
lugar:

- **Franja superior** limpia y de tono parejo, para la navbar
- **Un tercio lateral** (normalmente el izquierdo) como espacio negativo calmo,
  para titular y CTA
- **El sujeto** en los dos tercios restantes
- **Sin gente en primer plano** salvo que el usuario lo pida: caras y manos son
  lo que más se deforma
- **Sin texto, logos ni marcas** en la imagen — se agregan después en HTML

Pedile al usuario que confirme de qué lado quiere el aire antes de generar.

## Parámetros

**Imagen** — `generate_image`:
```
model: "gpt_image_2"
resolution: "1k"        // 2 créditos con quality medium
quality: "medium"       // "high" cuesta 4 y no rinde para 720p de video
aspect_ratio: "16:9"
```

**Video** — `generate_video`:
```
model: "seedance_2_0"
duration: 8             // 4 a 15; 8 es el equilibrio precio/resultado
resolution: "720p"
mode: "fast"            // "std" cuesta bastante más
aspect_ratio: "16:9"
generate_audio: false   // un hero de landing va siempre mudo
```

**Costo de referencia** (verificar siempre con `get_cost: true` antes de lanzar,
los precios cambian): imagen 1k/medium ≈ 2 créditos; video 8s/720p/fast ≈ 28;
12s ≈ 42. Total de una tanda de loop ≈ 30 créditos.

Consultá `balance` al principio y al final, e informá al usuario cuánto gastó.

Si el servidor devuelve un `preset_recommendation` que no tiene nada que ver con
la escena, rechazalo pasando `declined_preset_id` y comentáselo al usuario en
lugar de aplicarlo por tu cuenta.

## Esqueleto del prompt de video

Adaptá los elementos, pero mantené la estructura y la contundencia de las
prohibiciones. Escribilo en inglés: Seedance responde bastante mejor.

```
A perfectly seamless looping shot filmed with a completely locked-off camera on
a heavy tripod.

THE CAMERA NEVER MOVES: absolutely no dolly, no tracking, no push-in, no
pull-back, no pan, no tilt, no zoom, no orbit, no parallax, no handheld drift,
no camera shake whatsoever. The framing is frozen and identical in every single
frame from first to last.

[LISTAR LOS OBJETOS SÓLIDOS] stay completely static and locked in exactly the
same position and size in the frame, and never move, shift, warp, stretch or
morph.

[SI APLICA] NOTHING PROGRESSES FORWARD: this is a frozen moment in time, not a
time lapse. [prohibir la progresión concreta: que no se cocine, que no se
consuma, que no anochezca, que no se acumule].

The ONLY motion comes from small cyclical elements that continuously renew
themselves with no beginning and no end: [DESCRIBIR CADA FOCO DE MOVIMIENTO,
uno por uno, con su ubicación en el cuadro].

The light levels, the light direction, the exposure and the colour grade stay
absolutely identical and unchanging from the first frame to the last.

Nothing new ever enters or leaves the frame: no people, no hands, no new
objects.

The final frame must match the opening frame exactly so the clip loops
invisibly.

Photorealistic live action, locked tripod shot, [lente], [profundidad de campo],
natural film grain, [grade].
```

## Verificación

Cuando el video esté listo, mostralo con `job_display` y decile al usuario que
lo mire **en bucle**, no en reproducción normal — el reproductor del widget no
loopea, así que el defecto no se ve ahí.

Lista de control para pasarle:

1. ¿El sujeto quedó en el mismo lugar de principio a fin?
2. ¿Se nota el salto en el punto de reinicio?
3. ¿Cambió la luz o avanzó el tiempo durante el clip?
4. ¿Algo se deformó? (líneas finas, cristal, cromo, manos, caras)

Si falla algo, ofrecé regenerar el video con el prompt corregido (28 créditos) o
rehacer la imagen si el problema viene de la composición (2 créditos). **Preguntá
antes de rehacer.**

## Entrega

El markup mínimo para que loopee de verdad:

```html
<video src="hero.mp4" autoplay loop muted playsinline></video>
```

Los cuatro atributos importan: sin `muted` y `playsinline`, iOS bloquea el
autoplay.

Si el corte se sigue notando aunque el contenido esté bien, hay dos arreglos:
recortar unas décimas del final con ffmpeg hasta que empalme, o hacer un
crossfade corto entre el final y el principio. El crossfade funciona siempre y
cuesta unos KB más.

Si el usuario además va a controlar el video con scroll (GSAP ScrollTrigger),
avisale que conviene reencodear con keyframes densos (`ffmpeg -g 1`), porque el
MP4 que entrega Higgsfield tiene keyframes espaciados y el scrub salta.

## Límite conocido

Si tu entorno no puede descargar desde el CDN de Higgsfield, no vas a poder ver
las imágenes ni los videos que generás. **Decíselo al usuario de entrada** y
apoyate en su criterio visual: describí lo que pediste, no lo que "ves". No
afirmes que algo quedó bien si no lo verificaste.
