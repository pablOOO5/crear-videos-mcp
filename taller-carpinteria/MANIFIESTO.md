# taller-carpinteria — tanda de sept 2026

Recorrido interior para hero con scrub: tablero de herramientas → sobre el hombro
del carpintero. **25,5 créditos** (5 imágenes + 2 videos, uno descartado).

> **Este archivo es el único registro de la tanda.** Ni las imágenes ni los videos
> están versionados: se recuperan con el `job_id`. Si perdés esta tabla, perdés el
> material.

## Imágenes — `gpt_image_2`, 1k / medium / 16:9, 1,5 c/u

| Archivo | `job_id` | Estado |
|---|---|---|
| `frames/tablero-v1.png` | `60457d8c-8d12-4420-99dd-5fe969306b89` | Descartada: carpintero a 6,8× de distancia |
| `frames/tablero-v2.png` | `9c6c86a8-e55f-423e-82fa-82ead9332fef` | Descartada: sujeto a la izquierda del centro |
| **`frames/tablero-v3.png`** | **`09725e5b-b442-4a94-85e5-4d2f32cc69c3`** | **`start_image`** |
| `frames/final-v1.png` | `728ae6bb-fcb8-4463-92ee-66c1b5c5c546` | Descartada: arrastraba el error de v2 |
| **`frames/final-v2.png`** | **`68677536-3b7e-40ff-9463-f28f6bbc1e4b`** | **`end_image`** |

Se bajan con `job_status` / `job_display` sobre el `job_id` y un `curl` al `rawUrl`.

**Sin `job_id` y por lo tanto irrecuperables:** `frames/1.png`, `frames/2.png`,
`frames/frame1.png`, `frames/frame2.png`. Son material propio del usuario, hecho
fuera de Higgsfield, y no forman parte del entregable — el par definitivo se
generó de cero. Sólo existen en el disco local.

## Videos — no versionados

| `job_id` | Modelo | Créditos | Estado |
|---|---|---|---|
| `e8625f61-05f5-4962-9a5f-4804d3c2277e` | `veo3_1_lite`, 8 s | 8 | Descartado: inventó una puerta vidriada |
| **`866b8859-a0a7-450d-a2e4-28eb6ff53b9c`** | **`kling3_0` std, 8 s, `sound:"off"`** | **10** | **El bueno** |

## Cómo reconstruir `video/hero.mp4`

```bash
# 1. URL del crudo
#    job_display / job_status sobre 866b8859-a0a7-450d-a2e4-28eb6ff53b9c
# 2. bajarlo
curl -sL -o video/kling-raw.mp4 "<rawUrl>"
# 3. reencodear con keyframe por cuadro — SIN ESTO EL SCRUB SALTA
ffmpeg -i video/kling-raw.mp4 -c:v libx264 -g 1 -crf 20 -pix_fmt yuv420p -an \
  -movflags +faststart video/hero.mp4
```

El clip pasó las seis verificaciones técnicas y **se rechazó igual, por escala
insuficiente**. Es el caso que subió el umbral de la skill de 2× a 2,5×.

**No se relinealizó**: Kling entregó la curva ya pareja (0,78× a 1,11×), y el
remapeo mezcla cuadros y cuesta nitidez. Ver el punto 6 de la skill.

## Mediciones del par

| | |
|---|---|
| Residual del warp | 39,2 → 23,6 (**−40%**), centro de fuga x=45% y=50% |
| Escala aparente | **1,99×** (automática 1,90×, manual 122 → 243 px) — **RECHAZADO por el usuario: "quedó corto"** |
| Zona del titular | detalle 2,00 → 1,35, brillo 121 → 109 a lo largo del clip |
| Linealidad del clip | 0,78× a 1,11×, sin corregir |
