# Malla Ingeniería Civil — UNAL

App web de una sola página para seguir el avance en el pensum de **Ingeniería Civil de la
Universidad Nacional de Colombia (Sede Bogotá)** — plan flexible 180 créditos, Acuerdo 26 de
2015. Se hace clic en una materia para aprobarla; se van desbloqueando las que dependen de ella.
El progreso se guarda en `localStorage`. Inspirada en https://mallaiq.netlify.app.

> **Idioma:** el usuario es colombiano (Bogotá). Todo el texto de la app, docs, comentarios y
> mensajes de commit va en español neutro/colombiano — **sin voseo** ("haces", no "hacés";
> "revisa", no "revisá").

## Cómo correrlo

```bash
node serve.mjs      # sirve la carpeta en http://localhost:4178
```

No hay build. `index.html` es autocontenido (HTML + CSS + JS vanilla, sin framework). `serve.mjs`
es solo un servidor estático para desarrollo. Para publicar (Netlify, GitHub Pages, itch.io) basta
subir `index.html`.

## Estructura

- `index.html` — versión local de la app. Los datos del pensum están en la constante `SEMS`
  (materias obligatorias + optativas, por semestre). Cada materia: `{id, n, c, comp, r, opt?}`
  donde `comp` ∈ B/C/L y `r` es la lista de prerrequisitos.
- `malla-artifact.html` — misma app con diseño más trabajado (fuentes Archivo + IBM Plex Mono,
  fondo tipo plano). Es la que se publica como Artifact. Mantener las dos en sync.
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
- **Las electivas (Libre Elección, componente L) son un contador global**, no materias. Un
  control `− / número / +` en el encabezado, sobre 36 créditos. No se registra en qué semestre
  se cursaron (no tienen prerrequisito ni cupo por semestre). Se guarda como un número en
  `localStorage` (`malla_civil_elec`); migra automáticamente sumando el formato viejo (array por
  semestre). Decisión del usuario tras descartar: campo por semestre, y tarjetas de 3 cr fijas.
- **Las optativas van en su semestre** como materias con `opt:true` y borde punteado: Optativa
  de Herramientas de Ingeniería en sem 2 y 5 (comp B), Electiva Técnica en sem 6 (comp C, corte
  28 C), Construcción de Obras Civiles en sem 10 (comp C, prereq Fundamentos de Construcción).
  No se elige la materia concreta salvo por el nombre del cupo.
- **Total definido = 180 cr exactos** (126 obligatorias + TG 6 + 6 opt B + 6 opt C + 36 L).
  Barra de progreso `min(x/180, 100)`.
- **Prerrequisitos**: verificados uno por uno con el usuario contra su experiencia en la carrera
  (ver historial). Un par quedaron distintos a lo que sugería el diagrama del PDF (p. ej.
  Mecánica de Sólidos depende de Estática + Cálculo en Varias Variables, no de Dinámica;
  Dinámica abre Análisis Estructural Aplicado).
- **Columnas de semestre justificadas**: los semestres 1–8 estiran sus materias (`.m` con
  `flex:1`) para quedar todos a la misma altura; los semestres 9 y 10 (≤3 materias) llevan la
  clase `.short` y quedan a su altura natural.
- **Estética**: tema oscuro, cero emojis (solo el favicon del Artifact y el ✓ de aprobada).
  - **Color de celda = agrupación** (el área de la malla oficial: Matemáticas azul, Hidráulica
    cian, Estructuras gris, Geotecnia café, Construcción rosa, Vías naranja, etc.). El mapa está
    en la constante `AGR` (`clave → [color, nombre, etiqueta corta]`) y cada materia trae `a:`.
    Fondo = `color-mix(agr 20%, surface)` + franja izquierda `inset box-shadow` del color puro.
  - **Estado = solo el borde**: bloqueada → borde rojo tenue + atenuada; falta 1 → borde ámbar;
    disponible → borde claro (agr+blanco) con glow; aprobada → borde verde 2px + ✓, conserva el
    color del área. Hay una leyenda de estados + una clave de colores de agrupación bajo el header.

## Pendiente

- Confirmar créditos exactos de un par de materias si el usuario nota algo raro.
- ¿Modelar Prácticas / Electivas de Profundización con su corte de 108 / 126 créditos totales?
- Publicar (Netlify / GitHub Pages).
