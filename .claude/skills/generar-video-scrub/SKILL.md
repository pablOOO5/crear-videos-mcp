---
name: generar-video-scrub
description: Genera videos de dos frames con Higgsfield (GPT Image 2 + Seedance 2.0) para usar como hero de una landing page, normalmente controlados con scroll (scrub) vía GSAP. Cubre el flujo paso a paso con confirmación y preflight de costo, la elección del tipo de transición según su tasa de error, la estructura de los prompts de imagen y de video, el análisis medido de los frames generados antes de gastar en el video, y la entrega para scrub, incluido el reencodeo con keyframes densos y la relinealización del ritmo del clip. Usar siempre que el usuario pida un video para un hero, un video de frame inicial y final, un video con scroll o scrub, una transición entre dos imágenes, un explotado de producto, un recorrido interior, o pida ideas de videos para una landing — aunque no nombre Higgsfield ni la palabra "scrub".
---

# Video de dos frames para hero con scrub

Se generan dos imágenes —frame inicial y frame final— y un video que va de una a
la otra. En la landing, el video se controla con el scroll: el usuario avanza y
retrocede el clip con la rueda.

## REGLA NÚMERO UNO: una tarea por vez

**Nunca gastes créditos sin aprobación explícita.** Cada generación cuesta dinero
real. El flujo es una conversación entre el usuario y Higgsfield, y vos sos el
intermediario, no el que decide.

El ciclo es siempre el mismo, y nunca se saltea:

1. Proponer el prompt en el chat, para que el usuario lo lea antes de gastar
2. Correr `get_cost: true` y decir el costo exacto
3. Preguntar si se lanza
4. Generar
5. Analizar el resultado y esperar el veredicto antes de seguir

Nunca encadenes dos generaciones en un mismo turno. Nunca "avances para ahorrar
tiempo". La única excepción es que el usuario diga explícitamente "hacé todo de
corrido".

Consultá `balance` al principio y al final, e informá cuánto se gastó en la tanda.

## Elegir el tipo de video

Ordenados de menor a mayor tasa de error. **Ante la duda, elegí el de más
arriba.**

**1. Cambio de foco.** La cámara casi no se mueve: arranca cerrada en un detalle
y abre, o al revés. Riesgo mínimo en el video. Pero el error no desaparece: se
corre a las imágenes, donde calibrar el desenfoque suele llevar dos o tres
intentos. Eso es una ventaja, no un defecto —falla del lado de 1,5 y no del de
28—, pero cambia la expectativa de costo. Ver "Calibrar el desenfoque".

**2. Transformación con cámara fija.** Mismo encuadre exacto en los dos frames,
lo que cambia es el estado de la escena: obra → edificio terminado, salón vacío →
montado, producto armado → explotado. La cámara no se mueve nada. Sale muy bien.

**3. Cambio de escena dentro de un mismo espacio.** Un desplazamiento corto —dos
o tres metros—, un giro sobre el eje, un retroceso o un acercamiento, dentro del
mismo ambiente. La luz y los materiales ya están definidos, el modelo solo
resuelve el punto de vista. Es la opción más natural para inmobiliaria,
gastronomía, hotelería y arquitectura.

**4. Dos mundos distintos.** Frame inicial y final en realidades diferentes,
unidas por sentido. Es lo que más sorprende y lo que más falla.

### El criterio que decide si sale o no

> **Cuanto menos tenga que viajar la cámara entre los dos frames, mejor sale.**

Si el recorrido no le entra al modelo en la duración pedida, no viaja: **corta y
aparece** en el frame final. No se arregla prohibiendo cortes en el prompt, porque
no es un problema de obediencia sino de distancia.

Reglas que se desprenden:

- **El frame final tiene que estar contenido o anclado en el inicial.** El
  elemento que se muestra al final ya tiene que existir en la primera imagen. Si
  no está, el modelo lo alucina y la transición se rompe.

  **Esto vale también para el foco, no solo para la cámara.** En un cambio de
  foco el ancla es lo que se adivina detrás del desenfoque. Si el fondo quedó
  fundido en manchas sin forma, el sujeto del frame final no está anclado en
  ningún lado y el modelo lo inventa entero, aunque la cámara no se haya movido
  un milímetro. El desenfoque del frame inicial es entonces un límite del ancla,
  no una decisión estética: podés desenfocar hasta donde las formas se sigan
  adivinando, y ni un paso más.
- **Nada de aéreo alto → nivel del piso.** Es el error clásico. Si querés llegar
  al piso, arrancá de un aéreo bajo.
- **Compartí el eje.** Aunque los dos frames sean lugares distintos, que la
  cámara esté a la misma altura y mirando en la misma dirección.

Si el salto es inevitable y el corte aparece igual, las salidas son: alargar a 12
segundos, o partirlo en dos clips con un frame intermedio y pegarlos con ffmpeg.

### Pero también hay un piso, y es el error que más caro sale

Todo lo anterior empuja a mover poco. Eso baja la probabilidad de que el clip se
rompa, que **no es lo mismo** que la probabilidad de que valga la pena scrubearlo.
Un desplazamiento prolijo en el que no pasa nada se rechaza igual que uno cortado,
sólo que además de perder los 28 créditos era aburrido. La lista de tipos de arriba
ordena por riesgo técnico, no por interés.

La medida práctica del piso es **la escala aparente entre los dos frames**: cuánto
se agranda el sujeto del inicial al final (ver "Medir la distancia" más abajo).
Medido hasta hoy, todo de la tanda del vivero (sept 2026):

| Escala | Qué pasó |
|---|---|
| 1,35× | Avance de tres metros por un pasillo. **Rechazado por el usuario:** "prácticamente no hay un cambio de escena". |
| 2,50× | Plano general → primer plano de un objeto de la mesada. Aprobado. |

Es un caso de cada lado, así que no es una ley sino el comienzo de una serie: **por
debajo de 2× avisá antes de gastar los 28** y ofrecé un frame final más cerrado por
1,5. Anotá la escala de cada tanda con su veredicto, para que el umbral se apoye en
datos y no en intuición.

El corolario para elegir el tipo: el cambio de foco y la transformación con cámara
fija cambian mucho *aunque* la cámara no viaje — por eso son seguros y además
sirven. El que corre el riesgo de quedar por debajo del piso es el desplazamiento
corto. Si elegís ese, apuntá a que el frame final sea claramente **otro plano**, no
el mismo corrido unos metros.

## Composición para hero

- **Mitad o tercio lateral libre** en los DOS frames, para titular y CTA. Si la
  cámara se desplaza, que el aire esté del lado hacia el que se mueve.
- **Franja superior limpia** y de tono parejo, para la navbar.
- **Sin texto, logos, carteles ni marcas.** El modelo los escribe mal siempre. Se
  agregan después en HTML o SVG.
- **Sin caras ni gente en primer plano.** Las manos son lo que más se deforma: si
  hacen falta, que estén de costado y en penumbra, nunca de frente e iluminadas.
- **Cuidado con los espejos.** Al mover la cámara el reflejo tiene que cambiar de
  forma coherente y es donde el modelo inventa. Ponelos en ángulo oblicuo, con
  reflejo parcial.

Preguntale al usuario de qué lado quiere el aire antes de generar: es una decisión
de composición y va antes de la primera imagen.

**El movimiento, en cambio, no se decide antes de ver el frame inicial.** Es
tentador cerrarlo en la misma conversación que el aire —"aire a la izquierda,
entonces un lateral a la izquierda"— y queda una decisión tomada a ciegas que
después arrastra. En la tanda del vivero pasó exactamente eso: el frame inicial
salió con la mesada en fuga hacia el fondo del pasillo, y un lateral habría sacado
la mesada por la derecha revelando una pared izquierda que no existía en la imagen,
o sea sin ancla. El movimiento se deduce del frame inicial ya generado.

**Para scrub, el frame final es el que más se ve** — es donde el usuario se queda.
Que sea el más lindo de los dos y el que tenga el espacio para el texto, aunque el
primero sea más impactante.

## Generar el frame inicial

Estructura del prompt (en inglés, GPT Image 2 responde mejor):

```
Photorealistic [tipo de foto] of [SUJETO], shot [ángulo y altura] with a [lente],
[profundidad de campo].

[DESCRIPCIÓN DE LA ESCENA: materiales, colores, objetos concretos]

[LUZ: hora, dirección, calidad, sombras]

[SUJETO] occupies the right two thirds of the frame; the left third is
[qué hay ahí] as negative space. The upper strip of the frame is clean
[cielo/techo/pared], uniform in tone.

No text, no logos, no signage, no faces, no people.
Cinematic [grade], natural film grain.
```

Parámetros: `model: "gpt_image_2"`, `resolution: "1k"`, `quality: "medium"`,
`aspect_ratio: "16:9"`. Costo: **1,5 créditos**.

### Calibrar el desenfoque

En un clip de cambio de foco, la profundidad de campo no es un adorno del
prompt: es la técnica. Y el rango útil es angosto, así que no lo dejes librado a
una frase suelta. Rango medido con GPT Image 2:

| Lo que pedís | Lo que sale |
|---|---|
| *very shallow depth of field, creamy bokeh* | **Poco.** El fondo sale casi nítido y al foco no le queda distancia que recorrer: el video se ve casi estático. |
| *clearly defocused but the shapes remain readable* | **El punto.** Se adivinan las siluetas, se pierde la textura. |
| *extreme bokeh, unrecognisable, no discernible detail* | **Demasiado.** Se rompe el ancla y el frame final se inventa la escena. |

Describí el resultado, no la óptica. "f/1.4" y "shallow DOF" son ambiguos para el
modelo; enumerar qué se tiene que seguir reconociendo y qué se tiene que perder,
no:

```
Everything behind [SUJETO] is softly out of focus — a moderate, gentle defocus,
not an extreme melt. The silhouettes and shapes of [LA ESCENA] remain clearly
readable: you can still recognise [LISTAR TRES O CUATRO OBJETOS]. But all fine
detail is gone: no [TEXTURA], no [TEXTURA], no crisp edges anywhere in the
background.
```

Cuando dudes, quedate corto de desenfoque. Un rack focus tibio se ve pobre; uno
sin ancla directamente no se puede armar.

## Generar el frame final

**Siempre pasando el frame inicial como referencia.** Es lo que hace que sea la
misma escena y no una nueva:

```
medias: [{ role: "image", value: "<job_id del frame inicial>" }]
```

La referencia no suma costo. En el prompt, repetí explícitamente qué tiene que
mantenerse: *same architecture, same colours, same materials, same light
direction, identical colour grade as the reference image*.

Si es un desplazamiento, pedí que **algo del primer frame siga visible** en el
borde del cuadro. Es el ancla que le dice al modelo que es un movimiento y no un
corte a otro lugar.

Y describí la posición de los objetos tal como se ven en el frame inicial: si en
la primera imagen la mesada se ve desde arriba en ángulo, no pidas el detalle
"at counter level" o el objeto va a aparecer con otra orientación.

### La referencia sirve para cualquier re-roll, no solo para el frame final

Si rehacés un frame para corregir **un** atributo —más desenfoque, otra luz, sacar
un objeto— y reescribís el prompt desde cero, el modelo te cambia todo lo demás.
Es el error más caro de esta etapa: pedís más bokeh y te devuelve otro encuadre,
así que ahora tenés dos problemas en vez de uno.

Pasá el intento anterior como referencia y nombrá el único cambio:

```
Match the camera position, framing and composition of the reference image
exactly: [REPETIR LOS ELEMENTOS QUE SE MANTIENEN].

The ONE difference from the reference image is [EL ATRIBUTO]. [DESCRIBIRLO].
```

Funciona igual de bien para re-rollear el frame inicial que para generar el
final. La referencia no suma costo, así que no hay motivo para no usarla.

## Analizar los dos frames antes de gastar en el video

**Este paso es obligatorio y va antes del video.** El video cuesta 28 créditos: un
problema de composición detectado acá se arregla por 1,5.

No alcanza con leer el prompt que escribiste: **hay que bajar las dos imágenes y
mirarlas**. El prompt es lo que pediste; la imagen es lo que salió, y casi nunca
coinciden del todo. El prompt del video se escribe desde la imagen, no desde la
intención.

### Descargar las imágenes

1. Obtené las URL con `job_display` sobre cada `job_id`.
2. Bajalas al disco local:

```bash
mkdir -p /tmp/frames
curl -sL -o /tmp/frames/frame1.png "<url del frame inicial>"
curl -sL -o /tmp/frames/frame2.png "<url del frame final>"
```

3. Abrí cada una con `view`. Miralas de verdad, una al lado de la otra.

**Si la descarga falla**, es la política de red del entorno: el CDN de Higgsfield
puede estar bloqueado y `curl` devuelve un `x-deny-reason`. En ese caso **pedile
al usuario que suba las dos imágenes al chat** — adjuntas sí las podés ver. No
sigas a ciegas ni digas que algo "quedó bien" sin haberlo mirado: si no las
tenés, decilo y apoyate en el criterio visual del usuario.

### Deducir el movimiento mirando las dos imágenes

Este es el punto del paso. Comparando los dos frames tenés que poder responder:

- **¿Qué movimiento las une?** ¿Es un acercamiento, un lateral, un retroceso, un
  descenso, un cambio de foco, o la cámara está quieta y lo que cambia es la
  escena? Nombralo con precisión: eso es la primera línea del prompt del video.
- **¿En qué dirección y cuánto?** "Lateral a la izquierda, dos o tres metros" es
  utilizable; "se mueve un poco" no.
- **¿Qué objetos aparecen en los dos frames?** Esos son los que van en la lista
  de *stay rigid and physically solid*. Nombralos como se ven, no como los
  imaginabas.
- **¿Qué elementos de la escena pueden aportar movimiento ambiental?** Cortinas,
  vapor, plantas, sombras de ventana, follaje afuera, polvo en un haz de luz.
  Solo poné en el prompt los que estén realmente en la imagen.
- **¿Cómo está orientado y ubicado el sujeto?** Si en el frame final un objeto
  quedó girado o corrido respecto del inicial, el video va a saltar. Se corrige
  rehaciendo ese frame por 1,5, no escribiendo más prohibiciones.
- **¿La distancia es razonable, y es suficiente?** No la estimes a ojo: medí la
  escala aparente (ver "Medir la distancia"). Si la cámara tiene que cruzar medio
  ambiente en 8 segundos, avisá antes de gastar; si la escala no llega a 2×, avisá
  también, porque el clip va a salir bien y no va a servir.

Escribí el prompt del video recién después de esto, usando los sustantivos que
viste en las imágenes.

### Contale al usuario qué video va a salir

Mirar las imágenes sirve para **anticipar el error antes de que ocurra**. El
usuario no tiene por qué saber qué le cuesta a un modelo de video: eso es lo que
aportás vos. No le devuelvas la decisión sin la información.

Antes de gastar, escribile tres o cuatro líneas con:

1. **El clip que va a salir**, contado como una escena: de dónde arranca la
   cámara, qué hace durante los ocho segundos, dónde termina y qué se ve al
   final. Si al describirlo te suena forzado o inverosímil, ese es el aviso.
2. **Qué puede fallar en este caso concreto**, no en general: "hay una mano en el
   primer frame y es lo que más se deforma", "el espejo del fondo va a tener que
   recalcular el reflejo", "la tijera quedó girada respecto del otro frame".
3. **Qué cuesta arreglar cada cosa**: rehacer un frame 1,5, rehacer el video 28.

Y recién ahí preguntá si se avanza. Un problema nombrado antes de gastar vale más
que un diagnóstico perfecto después.

### Qué revisar en cada frame

- **Aire para el texto**: ¿la mitad o el tercio reservado quedó realmente limpio,
  sin objetos ni manchas de luz que compitan?
- **Franja superior**: ¿tono parejo para la navbar?
- **Texto colado**: carteles, números de casa, etiquetas de productos, grabados en
  metal. Es lo más frecuente.
- **Elementos frágiles**: manos con dedos raros, caras, espejos con reflejos
  imposibles, tipografía inventada.

### Qué revisar entre los dos frames

- **Continuidad**: ¿mismos materiales, mismos colores, misma hora del día?
- **Dirección de la luz**: ¿viene del mismo lado en los dos?
- **Ancla**: ¿el sujeto del frame final ya existe en el inicial?
- **Distancia**: ¿cuánto tiene que viajar la cámara? Si es mucho, decilo ahora y
  ofrecé rehacer un frame por 1,5 en vez de tirar 28 en un video que va a cortar.
- **Posición del sujeto**: si se mueve de lugar entre un frame y otro sin que la
  cámara lo justifique, el video va a saltar.
- **La franja de la navbar, comparada**: no alcanza con que esté limpia en cada
  frame por separado. El texto de la navbar es fijo y el fondo se le mueve debajo
  durante todo el scrub, así que lo que importa es el delta. En el vivero esa
  franja cayó **62 puntos** en el centro entre un frame y el otro: ningún color
  fijo de texto se lee en todo el recorrido. No se arregla rehaciendo un frame —se
  resuelve con un velo propio en CSS— pero hay que detectarlo acá para escribirlo
  en la entrega.

### Medir, no estimar: el ojo se equivoca justo en estas dos cosas

Las dos revisiones de arriba —corrimiento del sujeto y cambio de luz— son
precisamente las que peor juzga el ojo cuando además cambió el foco o el estado
de la escena. Un sujeto desenfocado *parece* más grande porque el halo se
expande, y una zona que ganó nitidez *parece* más clara. Las dos ilusiones te
empujan a rehacer un frame que estaba bien.

Con dos frames en disco esto se mide en treinta segundos y el resultado es
concluyente. Hacelo **siempre** antes de dar el diagnóstico:

```python
from PIL import Image
import numpy as np

a = np.asarray(Image.open("frame1.png").convert("RGB")).astype(np.float32)
b = np.asarray(Image.open("frame2.png").convert("RGB")).astype(np.float32)
ga, gb = a.mean(axis=2), b.mean(axis=2)

# 1. superposicion rojo/cian: gris = alineado, franjas de color = corrimiento
ov = np.stack([ga, gb, gb], axis=2)
Image.fromarray(ov.astype(np.uint8)).save("overlay.png")   # miralo con view

# 2. centroide y area del sujeto (ajusta umbral y recorte a tu imagen)
def obj(g, x0,y0,x1,y1, umbral=150):
    m = g[y0:y1, x0:x1] > umbral
    ys, xs = np.nonzero(m)
    return xs.mean()+x0, ys.mean()+y0, m.sum()

# 3. brillo medio por zona: pared del texto, franja de la navbar, y DOS zonas de
#    contenido invariante (ver abajo). El cuadro entero no sirve si hubo movimiento.
```

Cómo leerlo:

- **Corrimiento del sujeto**: por debajo del 0,5% del ancho no se nota. Si el
  área cambia mucho pero el centroide no, es el halo del desenfoque, no un
  movimiento — no rehagas nada.
- **Brillo**: diferencias por debajo de ~5 sobre 255 (2%) son imperceptibles. Lo
  que delata un empalme son los saltos grandes, no estos. **Y medilo sobre zonas
  de contenido invariante, nunca sobre el cuadro entero si la cámara se movió.**
  Elegí dos zonas que muestren lo mismo en los dos frames —un techo, un piso, una
  pared que no entra ni sale de cuadro— y comparalas sólo a ellas. En el vivero el
  cuadro entero cayó 18,8 puntos, que se lee como un cambio de exposición que va a
  producir un fade; era sólo que había entrado más follaje verde oscuro en cuadro,
  y el techo y el piso daban **0,3 de delta**. El número global te empuja a rehacer
  un frame que estaba bien.
- **Alineación**: si la superposición sale gris parejo en las zonas que no
  cambian, el encuadre está clavado.

### Medir la distancia: la escala aparente entre los dos frames

La distancia es la variable que decide **las dos** cosas que pueden salir mal —que
el clip corte por exceso, o que no valga la pena por defecto (ver "también hay un
piso")— y es la única que la lista de arriba deja librada al ojo. No hace falta:
se barre escala y centro de expansión sobre la imagen entera y se busca el par que
minimiza el error contra el frame final.

```python
from PIL import Image, ImageFilter
import numpy as np

a = Image.open("frame1.png").convert("L")
b = Image.open("frame2.png").convert("L")
W, H = a.size
sc = 4                                   # en chico alcanza, y es rapido
aw, ah = W//sc, H//sc
A = np.asarray(a.resize((aw, ah)).filter(ImageFilter.GaussianBlur(1))).astype(np.float32)
B = np.asarray(b.resize((aw, ah)).filter(ImageFilter.GaussianBlur(1))).astype(np.float32)
ys, xs = np.mgrid[0:ah, 0:aw].astype(np.float32)

best = None
for s in np.arange(1.00, 3.61, 0.05):                  # arranca en 1.0, sin piso
    for cx in np.arange(aw*0.10, aw*0.95, aw*0.05):    # centro de expansion
        for cy in np.arange(ah*0.10, ah*0.95, ah*0.05):
            sx = np.clip((xs-cx)/s + cx, 0, aw-1)
            sy = np.clip((ys-cy)/s + cy, 0, ah-1)
            d = np.abs(A[sy.astype(int), sx.astype(int)] - B).mean()
            if best is None or d < best[0]: best = (d, s, cx/aw, cy/ah)

print("escala %.2fx  residual %.1f (sin warp %.1f)  fuga x=%.0f%% y=%.0f%%"
      % (best[1], best[0], np.abs(A-B).mean(), 100*best[2], 100*best[3]))
```

Tres advertencias, las tres aprendidas rompiéndose la cabeza contra esto:

- **El número es aproximado y sensible a la grilla de búsqueda.** Achicar el rango
  de escalas o el paso de los centros lo mueve fácil un 20-30% —el mismo par de
  frames dio 1,80× con una grilla acotada y 1,35× con una amplia—. Lo robusto es el
  orden de magnitud y la comparación contra tandas anteriores, no el segundo
  decimal. No lo reportes con más precisión de la que tiene.
- **Controlalo a mano.** Medí el ancho del sujeto en píxeles en los dos frames y
  dividí. En el vivero eso dio 170 → 380 px, o sea 2,24×, contra 2,50× del
  automático: mismo orden, y el manual es el que manda si difieren mucho.
- **No uses template matching del sujeto recortado.** Es lo primero que uno
  intenta y tiene un sesgo que arruina el resultado: cuanto más chico el template
  escalado, más posiciones donde encaja, así que la correlación premia las escalas
  bajas. En el vivero devolvió **1,20×** para un par que estaba en 2,2-2,5×, y sólo
  daba un valor razonable si se le ponía un piso arbitrario al rango — es decir, si
  ya sabías la respuesta.

El residual, además, dice si hubo parallax real: si compensar la escala casi no
baja el error (40,3 → 35,1 en el vivero), es un dolly con parallax y no un zoom.

Entregá el diagnóstico como una lista corta de hallazgos concretos, con los
números medidos, y para cada problema decí qué cuesta arreglarlo. Después
preguntá si se avanza al video o se rehace un frame.

## Generar el video

Escribilo con lo que viste en las dos imágenes en el paso anterior: el movimiento
que deduciste, los objetos que aparecen en los dos frames y los focos de
movimiento que existen en la escena. Un prompt escrito desde la intención y no
desde las imágenes es la causa más común de que el video no empalme.

Estructura del prompt. La mayor parte tiene que describir **lo que sí pasa**: si
más de la mitad son prohibiciones, se diluye.

```
One single continuous uninterrupted camera move, no cuts, no transitions, no fades.

[EL MOVIMIENTO, concreto: dolly-in, lateral tracking to the left, locked-off,
descending drone, focus rack. De dónde arranca y dónde termina.]

Constant slow velocity throughout, perfectly steady, no handheld shake, no whip
pans, no speed ramps.

[LISTAR LOS OBJETOS SÓLIDOS] stay rigid and physically solid, moving naturally
with parallax as the camera passes, never warping, stretching or morphing.

Gentle ambient life throughout: [CUATRO A SEIS FOCOS DE MOVIMIENTO CÍCLICO, cada
uno con su ubicación: vapor, cortinas, sombras de hojas, polvo en los haces de
luz, follaje, gente desenfocada al fondo].

The light direction, the exposure and the colour grade stay identical from the
first frame to the last.

Nothing new enters the frame: no people, no faces, no text.
Photorealistic live action, cinematic [lente] look, [profundidad de campo],
natural film grain.
```

Los focos de movimiento ambiental no son decorativos: sin ellos el clip parece una
foto moviéndose, y con scrub eso se nota más.

**Para transformaciones**, agregá que el cambio es progresivo y a ritmo constante,
*one single flowing change with no jump and no sudden replacement*, y prohibí
explícitamente los fundidos: el atajo del modelo es resolverlo con un cross-fade.

Parámetros:

```
model: "seedance_2_0"
duration: 8             // 8 es el equilibrio precio/resultado; 12 si hay que cubrir distancia
resolution: "720p"
mode: "fast"
aspect_ratio: "16:9"
generate_audio: false   // un hero va siempre mudo
medias: [
  { role: "start_image", value: "<job_id frame inicial>" },
  { role: "end_image",   value: "<job_id frame final>" }
]
```

Costo: **28 créditos** los 8 segundos, **42** los 12.

### El preset "IN THE DARK"

El servidor recomienda el preset `24bae836-2c4a-48e0-89b6-49fcc0b21612` ("IN THE
DARK") ante casi cualquier prompt largo de cámara, sin importar la escena —
aparece igual en un amanecer en el Caribe que en un fondo blanco de estudio. Es un
falso positivo conocido. **Rechazalo siempre** pasando
`declined_preset_id: "24bae836-2c4a-48e0-89b6-49fcc0b21612"` y comentáselo al
usuario en una línea, en lugar de aplicarlo por tu cuenta.

## Verificar el video

Pedile al usuario que lo mire y pasale esta lista:

1. ¿El movimiento es continuo y a velocidad pareja, o hay un corte y aparece
   directamente en el frame final?
2. ¿Los objetos sólidos se mantienen sólidos, o algo se deforma en el medio?
3. ¿El parallax es correcto —lo cercano se mueve más rápido que lo lejano— o todo
   se desplaza en bloque como una foto?
4. ¿La luz y la exposición se mantienen?
5. ¿Apareció gente, texto o carteles que no se pidieron?
6. Para scrub: ¿hay algún tramo que va más rápido que el resto, o alguno donde no
   pasa nada? Con scroll, tanto una aceleración como un tramo muerto se notan más
   que en reproducción normal, y el tramo muerto es el más fácil de pasar por alto
   mirando el clip.

Si falla, ofrecé rehacer el video con el prompt corregido (28) o rehacer un frame
si el problema viene de la composición (1,5). **Preguntá antes de rehacer.**

### El punto 6 casi siempre falla, y se arregla gratis

Seedance no entrega el cambio a ritmo parejo: lo entrega con ease-in-out. En un
clip medido de 8 segundos, todo el cambio ocurrió entre el segundo 2 y el 5; el
primer segundo y medio y los últimos dos y medio eran cuadros casi idénticos, y
el tramo más rápido avanzaba **6,6 veces** más que el promedio.

En reproducción normal eso se lee como un ease natural y no molesta. **Con scrub
es un defecto**: el usuario scrollea sin que pase nada, después todo de golpe,
después nada. Y no se ve en el widget, porque ahí el clip corre solo.

**No siempre es simétrico.** En el clip del vivero el arranque estaba bien (0,89×
el promedio en el primer medio segundo) y el pico era tolerable (2,4×), pero el
movimiento se terminaba en el segundo 5,3 y el último tercio quedaba congelado:
**0,07× el promedio en el último segundo**, que era casi todo grano. Un pico
aceptable no te exime de medir; mirá los dos extremos de la curva.

No hace falta regenerar. Se remapean los cuadros para que el cambio avance
parejo:

1. Extraer los cuadros: `ffmpeg -i raw.mp4 png/f%04d.png`
2. Medir la **velocidad instantánea** de cada cuadro: la diferencia media absoluta
   contra el anterior, `np.abs(G[i]-G[i-1]).mean()`. Sirve igual para un cambio de
   foco, una transformación o un movimiento de cámara, sin caso especial.
3. **Restarle el piso de grano** —`np.percentile(d, 4)`, que en un clip medido dio
   0,163— y recortar a cero con `np.clip(d - piso, 0, None)`. Sin este paso el
   tramo congelado sigue sumando, porque el grano nunca es cero, y el remuestreo
   termina asignándole cuadros a un tramo donde no pasa nada.
4. Integrar la velocidad limpia y normalizar: `p = np.concatenate([[0],
   np.cumsum(dc)]); p /= p[-1]`, monotonizando con `np.maximum.accumulate`. Esa es
   la curva de avance.
5. Remuestrear a progresión pareja: `pos = np.interp(np.linspace(0,1,N), p,
   np.arange(len(p)))`, y para cada objetivo **mezclar los dos cuadros que lo
   bracketean** con su peso fraccional. Repetir el cuadro más cercano en vez de
   mezclar produce micro-tirones.
6. Reencodear la secuencia con `-g 1` (ver "Entrega para scrub").

**No uses la diferencia contra el primer cuadro como curva de avance**, aunque sea
lo que parece natural: satura. En el vivero llegó a 1,00 en el cuadro 128 cuando
todavía había movimiento real hasta el 178, y retimear con ella habría comprimido
el tramo final creyendo que lo emparejaba. La velocidad integrada no tiene ese
problema porque mide cuánto cambia cada cuadro, no cuánto se alejó del principio.

Verificá el resultado midiendo de nuevo: el tramo más rápido tiene que quedar
cerca de 2× el promedio o menos, y el más lento no debería bajar de ~0,5×. En el
vivero quedó entre 0,94× y 1,1×, partiendo de 0,06× y 2,4×.

**Guardá siempre el MP4 crudo antes de tocarlo.** El retiming es destructivo y
regenerar cuesta 28.

## Entrega para scrub

El MP4 de Higgsfield tiene **keyframes muy espaciados**. Al scrubear, el navegador
decodifica desde el keyframe anterior y el video salta. Hay que reencodear con un
keyframe por cuadro:

```bash
ffmpeg -i input.mp4 -c:v libx264 -g 1 -crf 20 -pix_fmt yuv420p -an \
  -movflags +faststart output.mp4
```

Pesa bastante más —un clip de 8s en 720p puede pasar de ~2 MB a 10-15— pero es la
única forma de que el scrub sea fluido.

Del lado del código:

```html
<video src="hero.mp4" muted playsinline preload="auto"></video>
```

```js
gsap.to(video, {
  currentTime: video.duration,
  ease: "none",
  scrollTrigger: { trigger: hero, start: "top top", end: "+=200%", scrub: 1 }
});
```

`scrub: 1` mete un segundo de suavizado y disimula cualquier salto que quede.
`muted` y `playsinline` son obligatorios para iOS. Si el video se reproduce solo en
vez de scrubearse, no pongas `autoplay`.

Las etiquetas, líneas guía y textos van encima en SVG o HTML, animados con el
mismo ScrollTrigger. Nunca dentro de la imagen.

## Costos de referencia

| Ítem | Créditos |
|---|---|
| Imagen 1k / medium / 16:9 | 1,5 |
| Video 8s / 720p / fast | 28 |
| Video 12s / 720p / fast | 42 |
| **Piso teórico (2 imágenes + video)** | **31** |
| **Tanda realista (3 a 5 imágenes + video)** | **32 a 36** |

Los 31 asumen que las dos imágenes salen a la primera, y casi nunca pasa: entre
calibrar el desenfoque, corregir el aire para el texto o sacar un objeto colado,
lo normal son tres a cinco. **Decile el rango al usuario cuando presentes el
presupuesto de la tanda**, no el piso, así no parece que te fuiste de precio a
mitad de camino.

El video, en cambio, suele salir a la primera si los frames se analizaron bien.
Ahí es donde rinde gastar imágenes de más.

Los precios cambian: **verificá siempre con `get_cost: true`** antes de lanzar.
