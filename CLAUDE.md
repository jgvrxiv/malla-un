# Malla Ingeniería Civil — UNAL

App web de una sola página para seguir el avance en el pensum de **Ingeniería Civil de la
Universidad Nacional de Colombia** (plan flexible 180 créditos, Acuerdo 26 de 2015). Hacés clic
en una materia para aprobarla; se van desbloqueando las que dependen de ella. El progreso se
guarda en `localStorage`. Inspirada en https://mallaiq.netlify.app.

## Cómo correrlo

```bash
node serve.mjs      # sirve la carpeta en http://localhost:4178
```

No hay build. `index.html` es autocontenido (HTML + CSS + JS vanilla, sin framework). `serve.mjs`
es solo un servidor estático para desarrollo. Para publicar (Netlify, GitHub Pages, itch.io) basta
subir `index.html`.

## Estructura

- `index.html` — toda la app. Los datos del pensum están en la constante `SEMS` (obligatorias por
  semestre) y `FLEX` (optativas/electivas). Cada materia: `{id, n, c, comp, r}` donde
  `comp` ∈ B/C/L y `r` es la lista de prerrequisitos.
- `PENSUM_BORRADOR.md` — extracción cruda del PDF + notas de qué se verificó con el usuario.
- `PLAN_ING (2).pdf` — la malla oficial (infografía de una página).
- `serve.mjs` — servidor estático dev.

## Modelo de prerrequisitos

`r` es un AND de:
- **id de otra materia** (string) → tiene que estar aprobada.
- **corte de créditos** `{comp:"C", min:67}` → suma de créditos aprobados de ese componente ≥ min.

Cortes reales del plan: Fundamentos de Construcción 67 C · Ing. Civil Sostenible 77 C · Trabajo
de Grado 77 C · Electiva Técnica 28 C. (Electivas de Profundización 108 y Prácticas 126 sobre
créditos totales — no modelados como materias sueltas todavía.)

No hay OR. Si alguna vez hace falta, cambiar `progress()` / `reqMet()`.

## Decisiones tomadas (no cambiar sin pensar)

- **Nivelación fuera**: Matemática Básica, Lecto-Escritura, Inglés I–IV no van en la app —
  las asigna el examen de admisión, no se pueden repetir y no cuentan en los 180.
- **`comp` (B/C/L)** es lo que mueve los cortes de crédito. "Disciplinar" = componente C
  (obligatorio + optativo). El header muestra el acumulado de cada componente.
- **Invalidación en cascada** (`cascade()`): al desmarcar una materia, se desmarca
  automáticamente todo lo que quede sin prerrequisitos válidos, en bucle hasta estabilizar.
  Es la mejora principal sobre mallaiq, que dejaba estados inconsistentes.
- **Solo las electivas (Libre Elección, componente L) van aparte**, en la sección "Electivas"
  bajo la malla — 12 cupos de 3 cr. Se cursan a menudo en otras facultades, no son fijas.
- **Las optativas van en su semestre** (a pedido del usuario), con `opt:true` y borde punteado:
  Optativa de Herramientas de Ingeniería en sem 2 y 5 (comp B), Electiva Técnica en sem 6
  (comp C, corte 28 C), Construcción de Obras Civiles en sem 10 (comp C, prereq Fundamentos de
  Construcción). No se elige la materia concreta salvo por el nombre del cupo.
- Con esto el total definido en la app da exactamente **180 cr**.
- **Barra de progreso** sobre 180 (meta oficial), `min(x/180, 100)`. Los cupos flexibles suman
  ~186 en total porque el reparto exacto de optativos/electivos del plan es aproximado.
- **Prerrequisitos**: verificados uno por uno con el usuario contra su experiencia en la carrera
  (ver historial). Un par quedaron distintos a lo que sugería el diagrama del PDF (p. ej.
  Mecánica de Sólidos depende de Estática + Cálculo en Varias Variables, no de Dinámica;
  Dinámica abre Análisis Estructural Aplicado).
- **Estética**: tema oscuro, cero emojis, fuente Inter. Colores por componente y por estado
  (bloqueada / falta 1 / disponible / aprobada).

## Pendiente

- Confirmar créditos exactos de un par de materias si el usuario nota algo raro.
- ¿Modelar Prácticas / Electivas de Profundización con su corte de 108 / 126 créditos totales?
- Publicar (Netlify / GitHub Pages).
