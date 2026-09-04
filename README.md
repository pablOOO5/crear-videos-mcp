# crear-videos-mcp

Taller de producción de video para secciones **hero** de landing pages.

El footage se genera con la MCP de Higgsfield (GPT Image 2 + Seedance 2.0), se
post-procesa con ffmpeg y se entrega como MP4 junto con el markup que lo reproduce.

## Estructura

- `.claude/skills/loop-hero-video/` — fondo animado que se repite sin corte (una sola imagen como start y end frame).
- `.claude/skills/generar-video-scrub/` — video controlado con el scroll (dos frames distintos, GSAP ScrollTrigger).
- `CLAUDE.md` — contrato con la MCP, reglas de costo, post-proceso y entrega.
- Una carpeta por cliente (`cafe-londres/`, `vivero/`) con `frames/`, `video/` y el `hero.html`.
