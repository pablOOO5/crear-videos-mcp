# joyeria-alianzas — tanda de sept 2026

Casa de alianzas de casamiento. Transformación con cámara fija para hero con
scrub: **dos alianzas en el estuche de la joyería → las mismas dos alianzas
puestas en las manos de los novios.** Los anillos no se mueven; las manos crecen
alrededor.

**36 créditos** — 6 imágenes y 3 videos; se descartaron 3 imágenes y 2 videos.

> **Este archivo es el único registro de la tanda.** Ni las imágenes ni los videos
> están versionados: se recuperan con el `job_id`. Si perdés esta tabla, perdés el
> material.

Todo se generó con Higgsfield. **No entró material propio del usuario a la
generación**: el usuario aportó una foto de referencia de la pose de manos, pero se
usó sólo para escribir el prompt, no como `medias`. La tanda se reconstruye entera
desde los `job_id`.

## Cómo llegó a ser lo que es

El pedido original fue **"dos mundos distintos, sin ancla, sin nada en común"** —
el tipo de mayor tasa de error de la skill. Se advirtió una vez; el usuario
reformuló solo. El concepto pasó por tres estados y **los dos primeros se
rechazaron por aburridos, no por fallas técnicas**:

1. **Estuche en la joyería → el mismo estuche en una iglesia** (fondo reemplazado,
   novios en silueta al fondo). Video generado. Pasó las seis verificaciones
   técnicas y se rechazó: *"nunca se revela la pareja, queda todo muy lineal, es
   como una imagen fija"*.
2. **Los anillos se vuelven los novios** (siluetas a contraluz de vitral, con el
   destello de la alianza como ancla de luz). Frame generado, video nunca lanzado.
   El usuario mostró una foto de referencia y pidió otra cosa.
3. **Los anillos se quedan siendo anillos y las manos crecen alrededor.** Es el
   estado final y el más sólido de los tres: mismo objeto, misma escala, misma
   posición en los dos frames.

### La lección cara de esta tanda

El clip descartado falló por una razón que **ninguna de las verificaciones de la
skill mide**: rango dinámico. Los dos frames tenían p99 = 113 y 114 — la misma
distribución de altas luces. El clip entero vivió entre 15,4 y 18,4 de brillo
medio: **3,0 sobre 255, el 1,2% del rango disponible.** La zona de la pareja
terminó con un contraste local de 43 sobre un fondo de brillo 19, y durante los
primeros tres segundos *se oscurecía* en vez de revelarse.

La causa fue proteger de más las zonas de texto. Las mediciones decían delta de
navbar +1,2 contra un umbral de 5 y p95 del titular en 28 con lugar hasta 100: eso
no era "pasa", era **estar usando la cuarta parte del presupuesto disponible**.

> **Medí el delta de p99 y de %pix>120 entre los dos frames antes de lanzar el
> video.** Un par con la misma distribución de altas luces produce un clip que se
> lee como una foto quieta, por más que pase todas las demás verificaciones.

El par final: p99 **113 → 213**, %pix>120 **0,8% → 8,9%**. Once veces más superficie
brillante.

## Imágenes — `gpt_image_2`, 1k / medium / 16:9

**1 crédito cada una, no 1,5.** El precio bajó respecto de la tabla de la skill;
verificado con `get_cost` en las seis llamadas.

| Archivo | `job_id` | Estado |
|---|---|---|
| `frames/estuche-v1.png` | `af724a44-c84b-4edf-b49b-f0fc6aeca293` | Descartada: vitrina iluminada encima de la zona del titular |
| **`frames/estuche-v2.png`** | **`ee9c7190-90a0-44ac-9d0a-7cd2bb8ff83b`** | **`start_image`** |
| `frames/iglesia.png` | `c1903b5d-22cd-45ed-a9de-b4dcb9ba1f81` | Descartada: concepto 1, sin rango dinámico |
| `frames/novios.png` | `4bb00e3b-9438-4470-b7d4-efa856bb1246` | Descartada: concepto 2, nunca se filmó |
| `frames/manos-v1.png` | `448459b5-dd5a-4417-b137-e714f2024f8d` | Descartada: su manga a la izquierda hacía cruzar la mano por la zona del titular |
| **`frames/manos-v2.png`** | **`8a247b13-db37-47de-8d86-aa667b2fdb6a`** | **`end_image`** |

*(La sexta imagen es `estuche-v1`; el conteo de 6 incluye las dos descartadas por
cambio de concepto.)*

Todas las posteriores a la primera se generaron **pasando otra como referencia**
(`medias: [{ role: "image", value: "<job_id>" }]`, sin costo extra) y nombrando el
único cambio. Es lo que mantuvo la composición entre intentos.

El CDN respondió `http=200` en las seis: **no estaba bloqueado en esta tanda.**

### Por qué se descartó `estuche-v1`

El tercio izquierdo, reservado para titular y CTA, quedó ocupado por la vitrina
iluminada — el objeto más brillante del cuadro.

| | v1 | v2 |
|---|---|---|
| media / std | 30,0 / 32,5 | **13,8 / 7,8** |
| p95 / máx | 114 / 176 | **29,7 / 54,7** |
| % de píxeles > 120 | 3,8% | **0,0%** |

El re-roll además dejó **dos masas verticales oscuras** en x≈19% y x≈28%, que se
usaron como ancla en los conceptos 1 y 2.

## Videos — no versionados

| `job_id` | Modelo | Créditos | Estado |
|---|---|---|---|
| `1ea5fa4f-c97a-470c-a604-a470b670adc1` | `kling3_0` std, 8 s, `sound:"off"` | 10 | Descartado: **impecable en lo técnico, rechazado por aburrido** |
| `d45916bd-f737-45a0-8593-082bb11b8e29` | `kling3_0` std, 8 s | 10 | Descartado: invadió el tercio del titular (p95 192) y alucinó un adorno dorado en la navbar |
| **`71936d16-46bc-4575-895e-28f08f6793ce`** | **`kling3_0` std, 8 s, `sound:"off"`** | **10** | **El bueno** |

`declined_preset_id: "24bae836-2c4a-48e0-89b6-49fcc0b21612"` en el `get_cost` y en
la generación. El preflight pasó limpio las dos veces.

12 s cuestan **15**. Se eligió 8 porque el clip descartado mostró que Kling se
queda sin material antes de terminar (último segundo a 0,23× del promedio): más
duración da más cola muerta, no más transformación.

## Mediciones del par final (`estuche-v2` → `manos-v2`)

| | |
|---|---|
| Posición de los anillos | x 66,6% → 68,6%, y **59,9% → 69,0%** (unos 68 px hacia abajo) |
| p99 | **113 → 213** |
| % de píxeles > 120 | **0,8% → 8,9%** |
| Brillo medio | 20,1 → 31,4 |
| Tercio del titular | 13,7 → **4,9** (std 1,6, p95 8) |
| **Mitad** izquierda | 20,0 → **6,1** (std 2,4, p95 10) |
| Navbar, global | **+0,1** |
| Navbar, esquina 85-100% | 0,9 → 7,3 (p95 9) |

**Los 68 px de bajada de los anillos.** Es 4× el corrimiento del par anterior. Se
avanzó igual porque son 0,35 px por cuadro y el par anterior había interpolado su
corrimiento de forma perfectamente lineal. **Y en vez de pedirle al modelo que "los
anillos no se muevan nunca" —que los frames desmentían— se le pidió que se
asentaran hacia abajo de a poco.** Contradecir lo que muestran los frames es lo que
hace que el modelo resuelva mal.

## Verificación del clip bueno (`71936d16`)

| | Crudo | Tras relinealizar |
|---|---|---|
| Linealidad por segundo | 0,36 · 1,89 · 1,99 · 1,54 · 1,42 · 0,61 · 0,17 · **0,02** | **0,75 · 0,93 · 1,04 · 1,14 · 1,16 · 1,09 · 1,14 · 0,76** |
| Pico | 2,75× | **2,37×** |
| Corte | mayor salto 2,5× la mediana — **no hay** | — |
| Titular tercio izq, peor cuadro | media 11,8 · p95 28 · **0,0% >120** | igual |
| Navbar tercio izq, peor cuadro | media 4,8 · p99 19 | igual |
| Navbar del 33% a la derecha | media 76,3 · **p99 237** | igual |
| Fidelidad | 1,98 al inicial · 2,25 al final | 1,98 · 3,78 |
| Keyframes | 1 de 193 | **193 de 193** |

**El defecto que quedó, y por qué se aceptó.** Entre s0,75 y s4,3 **entra una mano
desde arriba a la derecha a tomar el estuche** y ocupa la franja de la navbar
(pico p99 237 a s1,12). No es una alucinación: es un gesto coherente y bien hecho.
Se aceptó porque:

- Vive entero en el 12% superior, y **el tercio izquierdo de esa franja se mantiene
  en p99 19 durante los 193 cuadros** — el logo no necesita nada.
- Se resuelve gratis en CSS con un velo que arranca transparente a la izquierda.
  Ver `ENTREGA.html`.
- Ninguna banda horizontal del cuadro se salva (medidas seis, de y=0 a y=32%: todas
  llegan a p99 ~235), así que bajar la navbar no era una salida.
- El intento anterior de arreglar esto por composición costó 11 créditos y movió el
  problema del titular a la navbar. El siguiente lo movería a otro lado.

### Mediciones del par descartado (`estuche-v2` → `iglesia`)

Se conservan porque son el contraejemplo útil: **todo daba bien y el clip no
servía.**

| | |
|---|---|
| Ancla (estuche + anillos) | dx = +16 px (1,19% del ancho), dy = 0. Traslación pura |
| Residual del ancla al compensarla | 8,94 → **3,13 (−65%)**; sólo anillos, 14,76 → **4,00 (−73%)** |
| Fondo | dx = 0, residual **−0%**: ninguna traslación los relaciona |
| Navbar / titular / exposición | +1,2 / −0,8 / −2,5 — todos muy por debajo del umbral |
| **p99** | **113 → 114** ← el número que explica el rechazo |

**Los 16 px, y por qué no se rehizo el frame.** El umbral de la skill es 0,5% del
ancho y esto daba 1,19%. Se avanzó igual, y el clip lo confirmó: la deriva salió
perfectamente lineal (−2, −4, −6, −9, −11, −13, −14, −15 px a lo largo de los 8 s),
o sea 0,08 px por cuadro. **El umbral del 0,5% detecta un salto, no una deriva
lineal.** El centroide no lo detectó (daba 0,9%, sobre una máscara de brillos
especulares); lo detectó el **overlay rojo/cian** y lo cuantificó una búsqueda de
`dx,dy`. Para cámara fija: overlay + búsqueda de desplazamiento, no centroide.

## Kling 3.0 en transformación con cámara fija — dato nuevo

La comparación que puso a Kling como default de la skill se midió sobre un **dolly
de interior**, donde dio una linealidad de 0,78× a 1,11× y la skill concluyó que
**no hace falta relinealizar**. Medido acá sobre una **transformación con cámara
fija**, con el grano filtrado:

| | pico | s0 | s1 | s2 | s3 | s4 | s5 | s6 | s7 | último 0,5 s |
|---|---|---|---|---|---|---|---|---|---|---|
| Crudo | **3,41×** | 0,95 | 0,99 | 2,02 | 1,83 | 1,19 | 0,47 | 0,32 | **0,23** | **0,24** |
| Retimeado | **1,84×** | — | — | — | — | — | — | — | — | 0,52 |

> **La linealidad de Kling no generaliza del dolly a la transformación.** Acá se
> comporta peor que Veo 3.1 Lite en el dolly (1,52×) y **sí hay que relinealizar.**
> Medí siempre; no des por buena la fila de la tabla de la skill.

### Dos correcciones al método de relinealización de la skill

1. **Medí la velocidad sobre cuadros filtrados, no crudos.** En una escena oscura
   la diferencia entre cuadros consecutivos está dominada por el grano, y al
   mezclar cuadros el grano se descorrelaciona e **infla la medición justo donde
   los pasos son grandes**. Con cuadros crudos el retimeado medía un pico de 3,15×
   y una cola de 2,17×; los mismos archivos, medidos con
   `resize((320,179))` + `GaussianBlur(1.5)`, daban 2,18× y 1,67×. El primer par de
   números era un artefacto de medición.

2. **No restes el piso de grano en un clip oscuro.** El paso 3 de la skill dice que
   restar `np.percentile(d, 4)` es indispensable. Barriendo el piso entre el
   percentil 0 y el 6 y el corte de la fuente entre p≥0,990 y 1,0, **el mejor
   resultado por lejos fue no restar nada** (percentil 0, corte en p≥0,990): pico
   **1,84×**, por segundo entre **0,52× y 1,36×**. Con percentil 4 el pico se iba a
   5,6× y la cola a 3,4×. En un clip oscuro el piso se come el avance real de la
   cola y el remuestreo sobrecorrige.

**Y truncá la fuente donde el avance se termina.** Este clip tenía **8 cuadros con
avance exactamente cero** al final; `p` se quedaba plano en 0,9993 desde el cuadro
185. `np.interp` sobre esa meseta produce un salto en el último cuadro. Cortar en
`np.searchsorted(p, 0.990)` lo elimina.

## Cómo reconstruir `video/hero.mp4`

```bash
# 1. URL del crudo
#    job_display / job_status sobre 71936d16-46bc-4575-895e-28f08f6793ce
# 2. bajarlo
curl -sL -o video/kling-raw.mp4 "<rawUrl>"
# 3. relinealizar: curva sobre cuadros filtrados (320x179 + blur 1.5),
#    SIN restar piso de grano, truncando la fuente en np.searchsorted(p, 0.990)
#    -> corta en el cuadro 169 de 193
# 4. reencodear con keyframe por cuadro — SIN ESTO EL SCRUB SALTA
ffmpeg -i png-lin/f%04d.png -r 24 -c:v libx264 -g 1 -crf 20 -pix_fmt yuv420p -an \
  -movflags +faststart video/hero.mp4
```

El crudo de Kling trae **1 solo keyframe en 193 cuadros** (verificado con
`ffprobe`), igual que en `taller-carpinteria/`. El `-g 1` no es opcional.

Sale a 1280x716, no 1280x720.

## Entrega

```html
<video src="hero.mp4" muted playsinline preload="auto"></video>
```

Ver **`ENTREGA.html`**, que ya lleva los dos límites medidos escritos en el CSS:

- **El titular no pasa del 33% del ancho** (`left:4vw; width:29%`). Medida la mitad
  izquierda, el clip llega a p95 180 en el medio; el tercio izquierdo nunca supera
  28. Sin velo: esa zona se oscurece a lo largo del clip, no se aclara.
- **La navbar lleva un velo que arranca transparente a la izquierda** y llega a
  `rgba(0,0,0,.62)` desde el 42% del ancho. Es lo que tapa la mano que entra a
  tomar el estuche entre s0,75 y s4,3.

`scrub: 1` deja un segundo de suavizado. Si el recorrido se siente corto, subí el
`end` del ScrollTrigger de `+=200%` a `+=300%`: no cuesta créditos.
