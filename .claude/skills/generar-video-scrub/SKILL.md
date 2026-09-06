---
name: generar-video-scrub
description: Genera videos de dos frames con Higgsfield (GPT Image 2 + Kling 3.0) para usar como hero de una landing page, normalmente controlados con scroll (scrub) vía GSAP. Cubre el flujo paso a paso con confirmación y preflight de costo, la elección del tipo de transición según su tasa de error, la estructura de los prompts de imagen y de video, el análisis medido de los frames generados antes de gastar en el video, y la entrega para scrub, incluido el reencodeo con keyframes densos, la elección medida del modelo de video, y la relinealización del ritmo del clip cuando el modelo la necesita. Usar siempre que el usuario pida un video para un hero, un video de frame inicial y final, un video con scroll o scrub, una transición entre dos imágenes, un explotado de producto, un recorrido interior, o pida ideas de videos para una landing — aunque no nombre Higgsfield ni la palabra "scrub".
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
intentos. Eso es una ventaja, no un defecto —falla del lado de 1 y no del de
10—, pero cambia la expectativa de costo. Ver "Calibrar el desenfoque".

**2. Transformación con cámara fija.** Mismo encuadre exacto en los dos frames,
lo que cambia es el estado de la escena: obra → edificio terminado, salón vacío →
montado, producto armado → explotado. La cámara no se mueve nada. **Técnicamente
sale muy bien** — sin cortes ni geometría inventada. Pero es el tipo con más
rechazos por motivos que no son técnicos: no viene con el piso de interés resuelto
(medile el delta de altas luces) y es donde el modelo mete lo nuevo en el aire del
titular. Ver las dos secciones correspondientes antes de elegirlo.

**3. Cambio de escena dentro de un mismo espacio.** Un desplazamiento corto —dos
o tres metros—, un giro sobre el eje, un retroceso o un acercamiento, dentro del
mismo ambiente. La luz y los materiales ya están definidos, el modelo solo
resuelve el punto de vista. Es la opción más natural para inmobiliaria,
gastronomía, hotelería y arquitectura.

**4. Dos mundos distintos.** Frame inicial y final en realidades diferentes,
unidas por sentido. Es lo que más sorprende y lo que más falla.

### Si el sujeto es una persona: el avance sobre el hombro

No es un tipo más de la lista, es la salida a una contradicción real entre dos
reglas de esta skill. *"Sin caras ni gente en primer plano"* y *"el frame final
tiene que estar anclado en el inicial"* **no se pueden cumplir las dos** cuando el
plano final es cerrado sobre una persona:

- Si la ponés de frente, tenés una cara en primer plano, que es lo que peor sale.
- Si la ponés de espaldas, para verla de frente al final la cámara tiene que
  bordearla, y un arco de 180° es **geometría pura** que el modelo no tiene de
  dónde sacar: le estás pidiendo que invente la cara, el torso y qué hay sobre la
  mesa delante de ella.

La salida es no elegir ninguna de las dos: **de espaldas en los dos frames**, y el
plano final sobre su hombro. Se ven las manos, la herramienta y la pieza; nunca la
cara. La cámara sólo avanza en línea recta. Cero arco, cero geometría inventada, y
encima es el plano clásico de oficio — sirve para carpintería, gastronomía,
cerámica, joyería, cualquier taller.

Verificado en `taller-carpinteria/`: la cara no apareció en ninguno de los 193
cuadros, y en el prompt del video el candado fue explícito —*"seen only from
behind, he never turns around, his face is never visible at any point"*—.

**Condición, y es la de siempre:** sobre el banco del frame inicial ya tiene que
verse —aunque sea desenfocado— lo que el plano final va a mostrar de cerca. Si al
final hay un cepillo y viruta, al principio tiene que haber un cepillo y viruta.

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
- **El modelo puede inventar detalle; no puede inventar geometría.** Es la forma
  corta de la regla anterior y la que conviene tener en la cabeza al elegir el
  plano. Que una mancha desenfocada se convierta en una remera con textura es
  invención de *detalle*: cualquier modelo lo hace bien. Que la cámara bordee a
  una persona para verla de frente le pide inventar *geometría* —la cara, el
  torso, qué hay sobre la mesa delante de ella—, y ahí es donde alucina.
- **En un avance recto, el sujeto se separa del centro.** Lo que está a la
  izquierda del centro se va **más** a la izquierda a medida que la cámara
  avanza; lo que está a la derecha se va más a la derecha. De ahí sale una regla
  que no es obvia y que cuesta cara:

  > **El sujeto del frame final tiene que estar, en el frame inicial, del mismo
  > lado del centro donde va a terminar.**

  Y como el aire define de qué lado termina el sujeto, la cadena queda: aire a la
  izquierda → el sujeto termina a la derecha → **el sujeto tiene que arrancar a
  la derecha del centro**. Con el aire a la derecha, todo espejado.

  Medido en `taller-carpinteria/`: con el sujeto en x≈17% y el aire a la
  izquierda, **no existía ningún avance de cámara capaz de relacionar los dos
  frames** — el sujeto tenía que cruzar media pantalla hacia la derecha mientras
  el resto de la escena pedía un avance frontal. Dos re-rolls y 3 créditos. Se
  detecta antes con la prueba del residual (ver "Medir la distancia"), y se evita
  del todo decidiendo esto al componer el frame inicial.
- **Lo que entra en cuadro tiene que entrar por el lado opuesto a tu aire.** Es la
  hermana de la regla anterior y vale también cuando la cámara no se mueve. Durante
  una transformación el modelo tiene que meter lo nuevo por algún lado, y **usa el
  espacio que le dejaste vacío — que es justamente el que reservaste para el
  titular.**

  Medido en `joyeria-alianzas/` (sept 2026): con el aire a la izquierda y un frame
  final donde la manga del novio venía **desde la izquierda**, su mano tenía que
  cruzar la zona del titular para llegar a destino. Lo hizo: el tercio izquierdo
  llegó a **p95 192 con el 18% de sus píxeles sobre 120 durante tres de los ocho
  segundos**, y el titular blanco desaparecía en la mitad del scroll. Se rehízo el
  frame final con las dos manos entrando desde la derecha y el mismo tercio quedó
  con **cero píxeles sobre 120 en los 193 cuadros**.

  Se ve en el frame final antes de gastar: preguntate por dónde tiene que entrar
  cada cosa nueva. Cuesta 1 arreglarlo ahí y 10 después.

  **Prohibirlo en el prompt no alcanza** — se lo pedí dos veces marcado como
  CRITICAL y lo ignoró las dos. Es el mismo caso que la geometría cortada por el
  borde: no es desobediencia, es que no tiene otro lugar de donde traerlo.

  **Y esperá que el problema se mude, no que desaparezca.** Al sacar las manos del
  tercio izquierdo, la que entra a tomar el objeto pasó a cruzar la franja de la
  navbar (p99 237). Eso se tapó gratis con un velo en CSS. La regla completa es
  *"componé para el aire, verificá las dos zonas cuadro por cuadro, y presupuestá
  un velo"*, no *"se arregla componiendo"*.

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
sólo que además de perder los créditos era aburrido. La lista de tipos de arriba
ordena por riesgo técnico, no por interés.

El piso se mide distinto según si la cámara viaja o no. **Las dos mediciones son
baratas y las dos predijeron un rechazo antes de gastar el video.**

### Si la cámara viaja: la escala aparente

Cuánto se agranda el sujeto del inicial al final (ver "Medir la distancia" más
abajo). Serie medida hasta hoy (sept 2026):

| Escala | Qué pasó | Veredicto |
|---|---|---|
| 1,35× | Vivero: avance de tres metros por un pasillo | **Rechazado** — "prácticamente no hay un cambio de escena" |
| **1,99×** | Taller de carpintería: tablero de herramientas → sobre el hombro del carpintero. Clip técnicamente impecable: sin corte, sólidos intactos, luz estable, ritmo parejo | **Rechazado** — "quedó corto" |
| 2,50× | Vivero: plano general → primer plano de un objeto de la mesada | **Aprobado** |

> **Apuntá a 2,5×. Por debajo de 2,5× avisá antes de gastar el video** y ofrecé un
> frame final más cerrado por 1.

El 1,99× es el caso que fijó el umbral, y vale la pena leer cómo pasó: la medición
lo marcó "justo en el filo", se avisó antes de gastar, el usuario eligió avanzar, y
**el clip salió perfecto en las seis verificaciones técnicas y se rechazó igual**.
Ese es exactamente el modo de falla que esta sección describe: la escala no predice
si el video se rompe, predice si vale la pena scrubearlo, y son cosas distintas.

Dos lecciones prácticas:

- **Un clip impecable que quedó corto se tira igual.** No te dejes tranquilizar por
  que las verificaciones den bien: son ortogonales a esto.
- **Cerrar el frame final cuesta 1 y tiene valor esperado positivo cuando la
  medición da borderline.** Si sale corto igual, perdiste 1; si no lo cerrás y
  sale corto, perdés 1 *más* el video. En el taller de carpintería el modelo
  además **entregó menos de lo pedido**: le pedí hombros a un cuarto del ancho
  (336 px) y devolvió 243 px, o sea 1,99× en vez de los 2,75× planificados. Pedí
  el plano **más cerrado de lo que querés**, porque `gpt_image_2` afloja.

Anotá la escala de cada tanda con su veredicto, para que el umbral se siga apoyando
en datos y no en intuición.

### Si la cámara NO viaja: el delta de altas luces

**La escala aparente no aplica en cámara fija**, y durante un tiempo esta skill
afirmó que no hacía falta ninguna medida porque el cambio de foco y la
transformación *"cambian mucho aunque la cámara no viaje"*. **Es falso, y costó 10
créditos comprobarlo** (`joyeria-alianzas/`, sept 2026): una transformación con
cámara fija puede cambiar la escena entera y ser perceptualmente casi nada.

Lo que sí predice el rechazo es **cuánto se mueven las altas luces entre los dos
frames**:

```python
from PIL import Image
import numpy as np
a = np.asarray(Image.open("frame1.png").convert("L")).astype(np.float32)
b = np.asarray(Image.open("frame2.png").convert("L")).astype(np.float32)
for n, g in (("inicial", a), ("final", b)):
    print("%-8s media %5.1f  p99 %5.1f  %%pix>120 %4.1f"
          % (n, g.mean(), np.percentile(g, 99), 100*(g > 120).mean()))
```

| p99 inicial → final | %pix>120 | Qué pasó | Veredicto |
|---|---|---|---|
| **113 → 114** | 0,8% → 0,8% | Joyería: estuche en la vitrina → el mismo estuche en una iglesia. Clip impecable en las seis verificaciones | **Rechazado** — "es como una imagen fija" |
| 113 → 209 | 0,8% → 13,8% | Joyería: alianzas en el estuche → puestas en las manos de los novios | Aprobado (se rehízo por otro motivo) |
| **113 → 213** | 0,8% → 8,9% | Ídem, con las manos entrando desde la derecha | **Aprobado** |

> **Si el p99 casi no se mueve entre los dos frames, el clip va a parecer una foto
> quieta.** Avisá antes de gastar el video y ofrecé rehacer un frame por 1.

**No uses el brillo medio global para esto**: en el par rechazado iba de 20,1 a 19,2
y en el aprobado de 20,1 a 31,4 — números parecidos para casos opuestos. Lo que
manda es la cola alta del histograma, no el promedio.

**El modo de falla es contraintuitivo: viene de cuidar demasiado el texto.** En esa
tanda las mediciones daban delta de navbar +1,2 contra un umbral de 5, y p95 del
titular en 28 con lugar hasta 100. Eso no era "pasa": era **estar usando la cuarta
parte del rango disponible**. Las zonas de texto están separadas espacialmente de
donde vive la luz, así que casi siempre podés subir mucho el contraste del resto sin
tocarlas.

### El corolario para elegir el tipo

El que corre el riesgo de quedar por debajo del piso es el desplazamiento corto: si
elegís ese, apuntá a que el frame final sea claramente **otro plano**, no el mismo
corrido unos metros. Y el cambio de foco y la transformación con cámara fija son los
de menor riesgo técnico, pero **no vienen con el piso resuelto de arriba** — medíles
el delta de altas luces igual.

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
`aspect_ratio: "16:9"`. Costo: **1 crédito** (bajó de 1,5 en sept 2026; verificá con `get_cost`).

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
problema de composición detectado acá se arregla por 1.

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
  rehaciendo ese frame por 1, no escribiendo más prohibiciones.
- **¿La distancia es razonable, y es suficiente?** No la estimes a ojo: medí la
  escala aparente (ver "Medir la distancia"). Si la cámara tiene que cruzar medio
  ambiente en 8 segundos, avisá antes de gastar; **si la escala no llega a 2,5×,
  avisá también**, porque el clip va a salir bien y no va a servir. Ya pasó: un
  1,99× impecable en las seis verificaciones se rechazó por corto.

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
3. **Qué cuesta arreglar cada cosa**: rehacer un frame 1, rehacer el video 10.

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
- **Geometría cortada por el borde**: cualquier elemento de arquitectura que el
  cuadro corta —una ventana sin su alféizar, una puerta a medias, una escalera que
  sale de plano— es **donde el modelo inventa**. Al avanzar la cámara tiene que
  completarlo y no tiene de dónde sacarlo.

  Medido en `taller-carpinteria/`: una ventana cuadrada pegada al borde izquierdo,
  sin pared visible debajo del alféizar, **se convirtió en una puerta vidriada
  abierta** durante los primeros cinco segundos del clip — justo encima de la zona
  del titular. Kling 3.0 no cayó con el mismo par, pero es una lotería que no hace
  falta jugar.

  La regla: **lo que está cortado por el borde, o entra entero en cuadro, o no
  está.** Se arregla por 1 rehaciendo el frame, y es más barato que rifar cuál
  modelo lo extrapola bien. Prohibirlo en el prompt del video ayuda pero no
  alcanza: no es desobediencia, es falta de información.

### Qué revisar entre los dos frames

- **Continuidad**: ¿mismos materiales, mismos colores, misma hora del día?
- **Dirección de la luz**: ¿viene del mismo lado en los dos?
- **Ancla**: ¿el sujeto del frame final ya existe en el inicial?
- **Distancia**: ¿cuánto tiene que viajar la cámara? Si es mucho, decilo ahora y
  ofrecé rehacer un frame por 1 en vez de tirar 10 a 28 en un video que va a cortar.
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

- **Corrimiento del sujeto**: por debajo del 0,5% del ancho no se nota. **Por
  encima, fijate primero si es una traslación pura antes de rehacer el frame**: el
  umbral está para detectar un *salto* —una reubicación que el modelo no puede
  interpolar—, no una deriva lineal, que interpola sin esfuerzo. Se distinguen
  buscando el `dx,dy` que mejor alinea la zona (ver abajo): si el residual cae
  fuerte al compensarlo, es traslación pura y el modelo la reparte a lo largo del
  clip. Medido en `joyeria-alianzas/`: 16 px (1,19% del ancho, residual −73% al
  compensar) salieron como una deriva perfectamente lineal de −2, −4, −6, −9, −11,
  −13, −14, −15 px a lo largo de los 8 s, o sea **0,08 px por cuadro**; y 68 px de
  bajada en otro par tampoco se notaron. Tomado al pie de la letra, el umbral hace
  gastar créditos al pedo. Si el residual **no** cae, ahí sí rehacé el frame.
  Y si el área cambia mucho pero el centroide no, es el halo del desenfoque, no un
  movimiento — no rehagas nada.
- **Cámara fija: overlay + búsqueda de `dx,dy`, no centroide.** El centroide sobre
  una máscara de brillos especulares se equivoca —daba 0,9% donde el real era
  1,19%— y toda la sección "Medir la distancia" está escrita para avance de cámara.
  Cuando la cámara no se mueve, lo que sirve es el overlay rojo/cian para *ver* el
  corrimiento y una búsqueda de desplazamiento para *cuantificarlo*, corrida por
  separado sobre el sujeto y sobre el fondo. En una transformación bien armada el
  sujeto da un `dx,dy` chico con caída fuerte del residual, y el fondo da `dx=0` con
  caída de 0%: ninguna traslación lo relaciona porque cambió de verdad.
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
- **La segmentación automática del sujeto no es confiable — el recorte visual es
  la fuente de verdad.** Aislar al sujeto con una máscara de color o de umbral
  parece lo rápido y falla feo: en `taller-carpinteria/` una máscara sobre la
  remera azul oscura devolvió **30 y 46 píxeles de ruido** en dos intentos, y con
  otro umbral se comió el marco de la ventana y dio un bbox de 159 px de ancho
  para un sujeto de 70. Lo que funciona siempre: recortar la zona, ampliarla,
  **mirarla**, y medir el ancho de hombros sobre esa vista.
- **Controlalo a mano.** Medí el ancho del sujeto en píxeles en los dos frames y
  dividí. En el vivero eso dio 170 → 380 px, o sea 2,24×, contra 2,50× del
  automático: mismo orden, y el manual es el que manda si difieren mucho.
- **No uses template matching del sujeto recortado.** Es lo primero que uno
  intenta y tiene un sesgo que arruina el resultado: cuanto más chico el template
  escalado, más posiciones donde encaja, así que la correlación premia las escalas
  bajas. En el vivero devolvió **1,20×** para un par que estaba en 2,2-2,5×, y sólo
  daba un valor razonable si se le ponía un piso arbitrario al rango — es decir, si
  ya sabías la respuesta.

### El residual es el semáforo: pasa o no pasa al video

El mismo barrido devuelve, gratis, la medición **más rentable de todo el flujo**:
si los dos frames son geométricamente compatibles. Leelo así:

| | Par que **no** pasa | Par que pasa |
|---|---|---|
| Caída del residual al compensar | **−11%** (51,5 → 45,9) | **−40%** (39,2 → 23,6) |
| Centro de fuga | **x=5%**, pegado al borde | **x=45% y=50%**, centrado |

**El centro de fuga pegado al borde de la grilla es la firma del rechazo.**
Significa que el optimizador no encontró ningún punto de expansión que funcione,
o sea que **no existe ningún avance de cámara que relacione los dos frames**. No
lo interpretes como "el número dio un poco peor": es categórico.

Los dos casos de la tabla son el mismo cliente el mismo día
(`taller-carpinteria/`), con el mismo frame final. Lo único que cambiaba era de
qué lado del centro estaba el sujeto en el frame inicial.

**Si el residual no baja al menos ~30% o el centro de fuga sale pegado a un
borde, no lances el video.** Rehacé un frame por 1. Es la diferencia entre
gastar 1 y gastar 10 a 28 en un clip que va a cortar.

El residual también dice si hubo parallax real: si compensar la escala casi no
baja el error (40,3 → 35,1 en el vivero), es un dolly con parallax y no un zoom.

**En cámara fija este semáforo no aplica y no significa que el par sea malo.** El
barrido busca un avance de cámara, y si no lo hay el residual no baja y el centro
de fuga sale donde caiga — exactamente la firma que arriba se lee como rechazo. Para
un par de cámara fija el semáforo equivalente es el de la búsqueda de `dx,dy` por
zonas: **el sujeto tiene que dar una caída fuerte del residual (traslación pura) y
el fondo una caída de ~0%** (cambió de verdad, ninguna traslación lo relaciona).
Medido en el par aprobado de `joyeria-alianzas/`: sujeto 14,76 → 4,00 (**−73%**),
fondo −0%, y el fondo cambió 3,5× más que el sujeto ya compensado. Y el piso lo mide
el delta de altas luces, no la escala (ver "Si la cámara NO viaja").

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

### Qué modelo: Kling 3.0 por default

```
model: "kling3_0"
duration: 8             // 8 es el equilibrio; 12 si hay que cubrir distancia
mode: "std"             // "pro" cuesta 12 en vez de 10; no medido todavía
sound: "off"            // un hero va siempre mudo
aspect_ratio: "16:9"
declined_preset_id: "24bae836-2c4a-48e0-89b6-49fcc0b21612"
medias: [
  { role: "start_image", value: "<job_id frame inicial>" },
  { role: "end_image",   value: "<job_id frame final>" }
]
```

Costo: **10 créditos** los 8 segundos. No expone `resolution`: ese precio ya es el
de calidad final, así que no hay escalón barato de screening — y a 10 no lo
necesita. Acepta `unlim`: si el allowance está activo, sale 0.

**Fallback documentado — Seedance 2.0** (`model: "seedance_2_0"`, `mode: "fast"`,
`resolution: "720p"`, `generate_audio: false`): **28** los 8 s, **42** los 12. Más
caro y peor en todo lo medido, pero es el que tiene historia en `cafe-londres/` y
`vivero/`, y **sí** expone resolución: 480p baja los 8 s a **12** sin cambiar si
alucina o no, así que sigue siendo la opción para iterar barato cuando Kling falla.

#### La comparación que puso a Kling arriba

Mismo par de frames, mismo prompt, mismo día (`taller-carpinteria/`, sept 2026):

| | Veo 3.1 Lite (8) | **Kling 3.0 (10)** |
|---|---|---|
| Alucinación | ventana → puerta vidriada | **ninguna** |
| Linealidad cruda | 0,46× a 1,52× | **0,78× a 1,11×** |
| Medio seg. inicial / final | 0,34× / 0,41× | **0,76× / 0,91×** |
| Fidelidad al `start_image` | 2,7 | **2,1** |
| Fidelidad al `end_image` | 5,5 | **3,5** |
| Detalle en la zona del titular | 2,24 → 2,74 | **2,00 → 1,35** |
| Rango de brillo | 7,4 | **5,5** |

Kling ganó las siete. **Pero es un solo par y un solo tipo de plano** (dolly
interior): no está probado en cambio de foco ni en transformación con cámara fija,
que es de donde vienen los datos de Seedance. Anotá cada tanda nueva acá antes de
tratar esto como ley.

#### Otros modelos con `start_image` + `end_image`

Precios verificados con `get_cost` (8 s, 16:9, sin audio, sept 2026). El costo es
lineal por segundo:

| Modelo | 480p | 720p |
|---|---|---|
| Veo 3.1 Lite | — | **8** *(sin param de resolución)* |
| **Kling 3.0** std / pro | — | **10 / 12** |
| MiniMax H3 | — | 16 *(2K nativo)* |
| Wan 3.0 | 10 | 20 *(tiene `enable_thinking`)* |
| Seedance 2.0 Mini | 8 | 20 |
| MiniMax H3 Max | 12 | 20 |
| Gemini Omni Flash 1.1 | 8 *(360p)* | 24 |
| **Seedance 2.0 fast** | **12** | **28** |
| Wan 3.0 Prime | — | 28 |
| Cinema Studio 3.0 | — | 40 |
| FLUX 3 Video | — | 44 |
| Seedance 2.5 | — | 52 |

**Bajá resolución, nunca duración, para iterar.** 480p no cambia si el modelo
corta, alucina o deforma — sólo esconde la textura fina. Un clip de 4 s en cambio
tiene que cubrir la misma distancia en la mitad del tiempo, así que hace *más*
probable el corte: ibas a descartar un modelo bueno por un test injusto.

### El preset "IN THE DARK"

El servidor recomienda el preset `24bae836-2c4a-48e0-89b6-49fcc0b21612` ("IN THE
DARK") ante casi cualquier prompt largo de cámara, sin importar la escena —
aparece igual en un amanecer en el Caribe que en un fondo blanco de estudio. Es un
falso positivo conocido. **Rechazalo siempre** pasando
`declined_preset_id: "24bae836-2c4a-48e0-89b6-49fcc0b21612"` y comentáselo al
usuario en una línea, en lugar de aplicarlo por tu cuenta.

**Mandalo también en el `get_cost`**, no sólo en la generación: la recomendación
del preset **bloquea el preflight igual que la llamada real** —devuelve el aviso
en lugar del costo— así que sin el `declined_preset_id` perdés un ida y vuelta
antes de poder decirle el número al usuario.

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

7. **Las zonas de texto, cuadro por cuadro.** No alcanza con que estén limpias en
   los dos frames extremos: hay que medirlas en los 193 cuadros. Ver abajo.

Si falla, ofrecé rehacer el video con el prompt corregido (10 con Kling) o rehacer
un frame si el problema viene de la composición (1). **Preguntá antes de rehacer.**

### El punto 7: las zonas de texto se rompen en el medio del clip

Los puntos 1 a 6 miran el clip como película. **Ninguno mira si el titular y la
navbar se siguen leyendo**, y ahí es donde se fueron dos videos en
`joyeria-alianzas/` (sept 2026). Los dos tenían **las dos zonas perfectamente
limpias en el `start_image` y en el `end_image`** y las dos rotas en el medio: un
p95 de 192 en el tercio del titular a los 2,9 s, y un adorno dorado alucinado en la
franja de la navbar con p99 233 entre los 3,5 y los 6,5 s.

No se ve mirando el clip en el widget, y no se puede deducir de los dos frames.
Corre en segundos y hay que hacerlo siempre:

```python
from PIL import Image
import numpy as np, glob
G = np.stack([np.asarray(Image.open(f).convert("L")).astype(np.float32)
              for f in sorted(glob.glob("png/f*.png"))])
N, H, W = G.shape
for n, z in (("titular tercio izq", G[:, int(.28*H):int(.80*H), :int(.33*W)]),
             ("titular mitad izq",  G[:, int(.28*H):int(.80*H), :int(.50*W)]),
             ("navbar",             G[:, :int(.12*H), :]),
             ("navbar tercio izq",  G[:, :int(.12*H), :int(.33*W)])):
    m = z.mean(axis=(1, 2)); p = np.percentile(z, 95, axis=(1, 2))
    print("%-20s peor cuadro: media %5.1f  p95 %5.1f  %%>120 %4.1f  (s %.1f)"
          % (n, m.max(), p.max(), 100*(z > 120).mean(axis=(1, 2)).max(), m.argmax()/24))
```

Cómo leerlo:

- **El criterio es el peor cuadro, no el promedio ni los extremos.** El texto es fijo
  y el fondo se le mueve debajo todo el scroll: basta un tramo malo.
- **Referencia buena:** el clip aprobado dio, en el tercio del titular, media máxima
  11,8 · p95 28 · **0,0% de píxeles sobre 120 en los 193 cuadros**. El rechazado, en
  la misma zona, 56,9 · 192 · 18,2%.
- **Medí también las sub-zonas.** En el clip aprobado la franja de la navbar entera
  llegaba a p99 237, pero **su tercio izquierdo se mantenía en p99 19**: alcanzó con
  un velo que arranca transparente a la izquierda, y el logo quedó sin tapar.
- **Antes de rehacer, probá bajar la franja.** En ese caso midiendo seis bandas de
  y=0% a y=32% todas llegaban a p99 ~235, así que no había salida y el velo era la
  única. Pero cuando la hay, es gratis.

Un tramo malo **no obliga a rehacer el video**: un velo en CSS sobre la franja
afectada cuesta cero y es práctica normal en cualquier landing. Rehacer sale 10 a 28
y, como se midió, tiende a mudar el problema de zona en vez de eliminarlo. Reservá
el re-roll para cuando la zona rota sea la del titular, que ocupa media pantalla y
no se puede velar sin ensuciar el clip.

### El punto 6 depende del modelo — medilo siempre, corregilo sólo si hace falta

**Cuánto ease-in-out mete cada modelo, medido como pico sobre el promedio:**

| Modelo | Tipo de plano | Pico | ¿Hay que relinealizar? |
|---|---|---|---|
| Seedance 2.0 | dolly interior | **6,6×** | Sí |
| Veo 3.1 Lite | dolly interior | 1,52× | Sí — los extremos caían a 0,34× y 0,41× |
| Kling 3.0 | dolly interior | **1,11×** | No — 0,78× a 1,11× de fábrica |
| **Kling 3.0** | **transformación, cámara fija** | **2,75× y 3,41×** | **Sí — la cola caía a 0,17× y 0,02×** |

> **La linealidad depende del plano, no sólo del modelo. Medila siempre.** Durante un
> tiempo esta skill decía "con Kling se saltea el paso entero", apoyada en un único
> dolly. En transformación con cámara fija el mismo modelo se porta **peor que Veo en
> el dolly**: dos clips medidos en `joyeria-alianzas/` (sept 2026) dieron picos de
> 2,75× y 3,41× con los últimos dos segundos prácticamente congelados. No hay atajo
> por modelo.

El criterio es el de siempre: el tramo más rápido cerca de 2× o menos y el más
lento no por debajo de ~0,5×. **Si el clip ya lo cumple, no lo toques**: el
remapeo mezcla cuadros y cuesta nitidez.

Lo que sigue vale cuando el clip no cumple.

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
2. Medir la **velocidad instantánea** de cada cuadro **sobre cuadros filtrados**:
   achicar a ~320 px y desenfocar antes de restar,
   `Image.open(f).convert("L").resize((320,179)).filter(GaussianBlur(1.5))`, y
   recién ahí `np.abs(G[i]-G[i-1]).mean()`. Sirve igual para un cambio de foco, una
   transformación o un movimiento de cámara, sin caso especial. **El filtrado no es
   opcional: ver la advertencia de abajo.**
3. **Restar el piso de grano sólo si hace falta.** El piso —`np.percentile(d, 4)`,
   recortando con `np.clip(d - piso, 0, None)`— es un parche contra el grano, y si
   filtraste en el paso 2 casi siempre sobra. **En un clip oscuro directamente
   empeora**: barriendo el piso del percentil 0 al 6 en `joyeria-alianzas/`, no
   restar nada dio un pico de **1,37×** y restar el percentil 4 lo llevó a **6,4×**,
   porque el piso se come el avance real de la cola y el remuestreo sobrecorrige.
   Probá con percentil 0 primero.
4. Integrar la velocidad limpia y normalizar: `p = np.concatenate([[0],
   np.cumsum(dc)]); p /= p[-1]`, monotonizando con `np.maximum.accumulate`. Esa es
   la curva de avance.
5. **Truncar la fuente donde el avance se termina**: `END = np.searchsorted(p, 0.990)`
   y renormalizar `pt = p[:END+1] / p[END]`. Si la cola quedó congelada, `p` se queda
   plana ahí y `np.interp` le asigna todos los objetivos al primer índice de la
   meseta, y después pega un salto en el último cuadro. Medido: un clip tenía **8
   cuadros con avance exactamente cero** y el último objetivo saltaba 9 cuadros de
   golpe. Sin este paso la receta produce un tirón al final.
6. Remuestrear a progresión pareja: `pos = np.interp(np.linspace(0,1,N), pt,
   np.arange(END+1))`, y para cada objetivo **mezclar los dos cuadros que lo
   bracketean** con su peso fraccional. Repetir el cuadro más cercano en vez de
   mezclar produce micro-tirones.
7. Reencodear la secuencia con `-g 1` (ver "Entrega para scrub").

**Barré los dos parámetros en vez de adivinarlos.** El piso (percentil 0 a 6) y el
corte (0,985 a 1,0) se evalúan sobre los cuadros chicos en segundos, sin renderizar
nada en tamaño real. Elegí el par que minimiza el pico y mantiene todos los tramos
entre ~0,6× y ~1,5×.

> **Medí siempre sobre cuadros filtrados, también al verificar.** Con grano crudo,
> mezclar cuadros descorrelaciona el grano e **infla la medición justo donde los
> pasos del remuestreo son grandes**, o sea en la cola. En `joyeria-alianzas/` el
> mismo clip retimeado medía pico 3,15× y cola 2,17× con cuadros crudos, y 2,18× y
> 1,67× con cuadros filtrados. **El primer par de números era un artefacto de la
> medición**, y por poco lleva a "arreglar" dos veces un clip que ya estaba bien.
> Cuanto más oscura la escena, peor el efecto.

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
| Imagen 1k / medium / 16:9 | **1** *(bajó de 1,5; sept 2026)* |
| **Video 8s — Kling 3.0 std** | **10** |
| Video 12s — Kling 3.0 std | 15 |
| Video 8s — Seedance 2.0 fast 720p | 28 |
| Video 8s — Seedance 2.0 fast 480p *(iterar)* | 12 |
| Video 12s — Seedance 2.0 fast 720p | 42 |
| **Piso teórico (2 imágenes + Kling)** | **12** |
| **Tanda realista (3 a 5 imágenes + Kling)** | **13 a 15** |
| **Tanda con un video descartado** | **23 a 25** |

Medido de punta a punta en `taller-carpinteria/`: **25,5 créditos** para 5
imágenes y **dos** videos (uno descartado). Con un solo video habrían sido 17,5.

Y en `joyeria-alianzas/`: **36 créditos** para 6 imágenes y **tres** videos, con dos
descartados. Con uno solo habrían sido 16.

El piso asume que las dos imágenes salen a la primera, y casi nunca pasa: entre
calibrar el desenfoque, corregir el aire para el texto o sacar un objeto colado,
lo normal son tres a cinco. **Decile el rango al usuario cuando presentes el
presupuesto de la tanda**, no el piso, así no parece que te fuiste de precio a
mitad de camino.

**Y no prometas que el video sale a la primera.** Esta skill decía que "suele salir
a la primera si los frames se analizaron bien"; en `joyeria-alianzas/` se
analizaron bien y se descartaron dos, uno por rango dinámico y otro por invasión de
las zonas de texto — ninguno de los dos motivos era visible en los frames. Sigue
siendo cierto que **rinde gastar imágenes de más**, porque a 1 crédito son la parte
barata; pero cuando el plano es una transformación, presupuestá **dos videos**.

Los precios cambian: **verificá siempre con `get_cost: true`** antes de lanzar.
