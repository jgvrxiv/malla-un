# Malla curricular — UNAL

App web de una sola página para seguir el avance en los pensa de la **Universidad Nacional de
Colombia (Sede Bogotá)**. Empezó con **Ingeniería Civil** (plan flexible 180 créditos, Acuerdo 26
de 2015) y hoy cubre **50 carreras**: las 9 ingenierías (Civil, Química, Agrícola, Industrial,
Sistemas, Eléctrica, Mecatrónica, Mecánica y Electrónica — verificadas con estudiantes) y 41 de las
facultades de Ciencias, Ciencias Económicas, Medicina, Enfermería, Odontología, Derecho/Ciencias Políticas, Ciencias Humanas, Ciencias Agrarias y Medicina Veterinaria y de Zootecnia (reconstruidas de las mallas oficiales, **sin
verificar**). Se hace clic en una materia para aprobarla; se van desbloqueando las que dependen de
ella. El progreso se guarda en `localStorage`. Inspirada en https://mallaiq.netlify.app.

> **Idioma:** el usuario es colombiano (Bogotá). Todo el texto de la app, docs, comentarios y
> mensajes de commit va en español neutro/colombiano — **sin voseo** ("haces", no "hacés";
> "revisa", no "revisá").

## Cómo correrlo

```bash
node serve.mjs      # sirve la carpeta en http://localhost:4179
```

No hay build. Cada `.html` es autocontenido (HTML + CSS + JS vanilla, sin framework). `serve.mjs`
es solo un servidor estático para desarrollo. Para publicar (Netlify, GitHub Pages, itch.io) se
sube la carpeta entera.

## Estructura

**`index.html` y `malla-artifact.html` son el MISMO archivo** (idénticos byte a byte; el segundo
existe sólo porque es el que se publica como Artifact). Un solo archivo autocontenido con **dos
vistas** y un router por hash:

- `#inicio` (por defecto) — **página de inicio**: hero, grilla de carreras (9 activas, sin
  "Próximamente"), "cómo funciona", "¿falta tu carrera?" y footer con el descargo de *proyecto
  estudiantil independiente*. Café y correo son **placeholders** (`<!-- TODO -->`).
- `#civil` / `#quimica` / `#agricola` / `#industrial` / `#sistemas` / `#electrica` /
  `#mecatronica` / `#mecanica` / `#electronica` — **una sola vista** `#view-malla` que muestra la
  carrera del hash.

**Registro `MALLAS = { civil, quimica, agricola, industrial, sistemas, electrica, mecatronica,
mecanica, electronica }`.** Cada
entrada trae `sems`, `agr`, `store`/`elecStore` (claves de `localStorage`), `meta` (denominadores
de las badges por componente), `sem9fix`, `compact`, `target`/`libreMeta` (opcionales, por
defecto 180/36) y `foot` (HTML del pie). `SEMS_<X>`/`AGR_<X>` son las constantes de datos.
`setMalla(key)` cambia los `let` activos (`TARGET`, `LIBRE_META`, `SEMS`, `AGR`, `ALL`, `META`,
`STORE`, `ELEC_STORE`, `SEM9FIX`) y relee `approved`/`elec` del navegador. **El motor no cambió**:
las funciones (`progress`, `cascade`, `toggle`, `card`, `render`…) son las mismas, solo leen esos
`let` en vez de constantes. `render()` aplica el apaño de altura del sem 9 **solo si `SEM9FIX`**
(true solo para civil).

- **`TARGET` / `LIBRE_META` por carrera**: eran `const 180/36`; ahora son `let` que `setMalla()`
  fija desde `M.target` / `M.libreMeta` (Civil/Química/Agrícola 180/36 · Industrial 168/34 ·
  Sistemas 165/33 · Eléctrica 167/33).
- **`.short` = solo los dos últimos semestres** (`i >= SEMS.length - 2`), no por conteo de
  materias. Antes era `sem.m.length <= 3`, lo que desjustificaba carreras con un semestre corto
  en medio (Eléctrica sem 2 y 8 tienen 3 materias). Los semestres 1–8 SIEMPRE se estiran a la
  misma altura.
- **Semestre 100% libre elección** (`sem.m` vacío — Sistemas e Industrial sem 9): `render()` pone
  ahí el contador de electivas (`elecWidget(true)` → clase `.elec.solo`, se estira para llenar la
  columna) y el Trabajo de Grado queda solo en el sem 10. `elecPlaced` evita duplicar el widget:
  si ningún semestre está vacío, va en el último (Civil/Química/Agrícola/Eléctrica). Cuando eso
  pasa, el `.tailgroup` también lleva la clase `.tg-libre`: como sem 9/10 tienen poco contenido,
  esas columnas quedan a su alto natural y la clave de colores (`.agrkey`) se lleva el espacio
  sobrante, en columna (una agrupación por renglón, `justify-content:space-between`).
- **`compact`** (`compact:true` en `MALLAS`): fichas más chicas, mismo diseño justificado.
  Lo usan Agrícola, Industrial, Sistemas y Eléctrica. La clase `#view-malla.compact` la pone el
  router. Además el router pone `#view-malla.malla-<key>` para reglas propias de una carrera
  (el ajuste fino de sem 9/10 de Agrícola — celdas a 96/118 px — está scopeado a
  `.malla-agricola`, no a `.compact`).

El router (`route()` + `hashchange`) alterna `#view-inicio` / `#view-malla` con `[hidden]`,
llama `setMalla` + `render`, rellena `#m-h1` / `#m-sub` / `#m-foot`, muestra/oculta "Reiniciar",
ajusta `document.title` y hace scroll arriba. El logo de la barra apunta a `#inicio`.

- **Química**: créditos y estructura de la malla oficial (Acuerdo 02 de 2013, PDF de
  ingenieria.bogota.unal.edu.co); **prerrequisitos y orden a partir de mallaiq.netlify.app**
  (código de datos de mallaiq extraído en su `<script>` inline), **verificados con estudiantes de
  la carrera**. Metas B 69 / C 75 / L 36 = 180 (cuadran con el PDF). Cortes: `{comp:"T",min:108}`
  (créditos totales → Optativa de Materiales), `{comp:"C",min:53}` (Taller de Proyectos
  Interdisciplinarios), `{comp:"C",min:60}` (Trabajo de Grado). Se agregó
  `if(comp==="T") return totalCredits()` a `creditsBy` para el corte por total.
- **Agrícola**: nombres, créditos, componentes y **agrupaciones** de la malla oficial (Acuerdo 30
  de 2016); los **prerrequisitos se leyeron del diagrama** del PDF (letras A–R = flechas) y fueron
  **verificados con estudiantes de la carrera**. Metas B 59 / C 85 / L 36 = 180 (cuadran con el
  PDF). Cortes solo `{comp:"C",min:40/65/68}` (Electrotecnia / Taller Interdisciplinario / Trabajo
  de Grado). Códigos reales del PDF. PDF descargado y rasterizado con `mupdf`+`sharp` en el
  scratchpad para leer el diagrama.
- **Industrial**: nombres, códigos, créditos, componentes y agrupaciones de la malla oficial
  (Acuerdo 024 de 2014); **prerrequisitos reconstruidos del diagrama** (nodos con letras) y
  **verificados con estudiantes de la carrera**. Metas B 42 / C 92 / L 34 = 168 (cuadran con los
  totales de componente del PDF: Fund. 22 oblig + 20 opt; Discipl. 52 oblig + 40 opt; Libre 34).
  Sem 9 = 100% libre. Corte `{comp:"C",min:70}` → Trabajo de Grado.
- **Sistemas y Computación**: plan 2A74, Acuerdo 033 de 2007. Metas B 51 / C 81 / L 33 = 165
  (cuadran con el PDF: Matemáticas 16, Prob/Est 3, Cs. Computación 18, Física 8, Cs. Econ. 6 = B;
  Métodos y Tec. Software 21, Infraestructura 30, Comp. Visual 3, Sist. Inteligentes 3, SMOS 12,
  Contexto 6, TG 6 = C). Sem 9 = 100% libre. `{comp:"C",min:55}` (Taller de Proyectos
  Interdisciplinarios) y `{comp:"C",min:60}` (Trabajo de Grado — id `TG_SIS`, el PDF no da el
  código) son **estimados** (el PDF los marca con `*` sin número). Prerrequisitos reconstruidos
  del diagrama de flechas numeradas y **verificados con estudiantes de la carrera**.
- **Eléctrica**: plan de estudios 2544. Metas B 63 / C 71 / L 33 = 167 (cuadran con el PDF:
  Fund. 51 oblig + 12 opt; Discipl. 63 oblig + 8 opt; Libre 33). La Nivelación y la Lengua
  Extranjera no se incluyen. Cortes `{comp:"T",min:121}` (70% del programa → Taller de Proyectos
  Interdisciplinarios) y `{comp:"C",min:63}` (80% disciplinar → Trabajo de Grado). Prerrequisitos
  reconstruidos del diagrama (nodos con letras) y **verificados con estudiantes de la carrera**.
- **Mecatrónica**: Acuerdo 18 de 2014. Metas B 56 / C 87 / L 36 = 179 (cuadran con el PDF:
  Fund. 47 oblig + 9 opt; Discipl. 81 oblig + 6 opt; Libre 36). La Lengua Extranjera no va.
  Cortes `{comp:"T",min:107}` (60% → Taller Interdisc.), `{comp:"T",min:134}` (75% → Proyecto
  Aplicado de Ingeniería), `{comp:"C",min:70}` (80% disciplinar → Trabajo de Grado).
  Agrupaciones B: Matemáticas, Física, Expresión Gráfica, Química, Cs. Económicas, Informática y
  Herramientas (clave `info`). Disciplinares: Inv./Innovación, Ing. Electrónica, Ing. Informática
  (clave `inf`), Ing. de Diseño, Automatización Control y Robótica, Materiales y Procesos.
  Prerrequisitos reconstruidos del diagrama y **verificados con estudiantes de la carrera**.
- **Mecánica**: Acuerdo 68 de 2018. Metas B 58 / C 86 / L 36 = 180 (cuadran con el PDF:
  Fund. 50 oblig + 8 opt; Discipl. 80 oblig + 6 opt; Libre 36). La Lengua Extranjera no va.
  Cortes `{comp:"T",min:99}` (55% → Taller Interdisc.), `{comp:"T",min:135}` (75% → Proyecto
  Aplicado), `{comp:"C",min:69}` (80% disciplinar → Trabajo de Grado). El diagrama tiene las
  materias muy escalonadas: la **asignación a semestres es aproximada** (reconstruida), pero las
  sumas de crédito por componente sí cuadran y los prerrequisitos ya fueron **verificados con
  estudiantes de la carrera**. "Herramientas para Ingeniería" quedó en Fundamentación (clave
  `her`). Créditos ajustados a ojo en 1 optativa de Física para cuadrar B=58.
- **Electrónica**: plan de estudios 2545 (hermano del 2544 de Eléctrica). Metas B 60 / C 78 /
  L 34 = 172 (cuadran con el PDF: Fund. 51 oblig + 9 opt; Discipl. 72 oblig + 6 opt; Libre 34).
  Nivelación y Lengua Extranjera no van. Cortes iguales a Eléctrica: `{comp:"T",min:121}` (Taller
  Inter-disciplinarios) y `{comp:"C",min:63}` (Trabajo de Grado). "Optativa de Telecomunicaciones"
  2019125 tiene código → se modela como obligatoria (la disc. optativa son las de Electrónica
  Análoga y Digital). Prerrequisitos reconstruidos del diagrama y **verificados con estudiantes de
  la carrera**.
- **Inicio generado desde `MALLAS`** (`renderInicio()`): ya no hay tarjetas escritas a mano. La
  constante `FACULTADES` (`ingenieria`, `ciencias`, `economicas`, `medicina`, `enfermeria`, `odontologia`, `derecho`, `humanas`, `agrarias`, `veterinaria`, `artes`) define las secciones y
  su orden; cada entrada de `MALLAS` trae `fac`, `color`, `cm` (renglón de la tarjeta) y `ver`
  (`true` = verificada con estudiantes → etiqueta "Verificada"; `false` → "Sin verificar", ámbar).
  El contador y el pie (`#foot-ver`) se calculan solos. Cada facultad lleva su grilla fija de 3
  columnas. Las 9 de Ingeniería son `ver:true`; las otras 41 son `ver:false`.
- **Facultades nuevas (Ciencias, Económicas, Medicina)** — datos con el helper `_m(id, nombre,
  créditos, comp, agr, prereqs, opt?)` y claves de `MALLAS` `quimica-ciencias`, `computacion`,
  `matematicas`, `fisica`, `geologia`, `farmacia`, `estadistica`, `economia`, `administracion`,
  `contaduria`, `nutricion`, `fonoaudiologia`, `medicina`, `enfermeria`, `odontologia`, `derecho`, `ciencia-politica`. Fuentes: mallas PDF de cada facultad + PEP
  (`diracad.bogota.unal.edu.co/recursos/pdf/unal-aspirante/PEP/PEP-<slug>.pdf`). Todas con sumas B/C/L
  iguales a las del PDF/PEP, salvo Química (Ciencias), cuya malla no marca el componente de cada
  materia (B/C aproximado, suma 158 vs 160 del PEP). Notas por carrera:
  - **Química (Ciencias)**: prerrequisitos de la malla; TG = 100% de B y de C obligatorio (cortes 40 B / 78 C, estimados).
  - **Computación, Matemáticas, Física**: prerrequisitos leídos de flechas (Matemáticas y Física
    decodificados con las letras del diagrama vía `pdf.toStructuredText`). Cortes del TG: Matemáticas
    112 T (80%), Física 60 T y 80 T (los "60/80 créditos del plan" de la malla).
  - **Geología**: sin códigos (ids `GEO_*`); componente leído del **color del texto** (muestreo de píxeles).
    Los semestres se llaman "matrículas". Corte TG 106 T (80% de 133 sin libre, estimado).
  - **Farmacia**: sin códigos; las flechas de colores no se pudieron leer → prerrequisitos aproximados.
  - **Estadística**: malla solo con nombres; créditos del PEP; muchos cupos ("Núcleo Estadístico").
  - **Economía / Administración / Contaduría**: sin flechas; prerrequisitos por secuencia. Sem 9 = libre
    (`m:[]`, activa `.tg-libre`).
  - **Nutrición y Dietética**: prerrequisitos del PEP (sí los trae); "Fisiología General" (4) pasa a cupo
    de 3 para cuadrar el PEP. **Fonoaudiología**: 2 optativas de fundamentación ajustadas a 3 cr porque
    los totales impresos por semestre no cuadran; sin flechas.
  - **Medicina** (12 semestres, 256 cr): malla por semestres en la **página 28 del PEP** (`pep_2_10.pdf`,
    la pág. 29 es el "rompecabezas" por agrupaciones). Fundamentación 72 = Biología del Desarrollo I+II +
    Salud Pública y CS I + Enlace Básico Clínico I (Farmacología es F aunque la malla la pinte de
    Disciplinar). La malla impresa tiene 1 cr de diferencia con el plan y typos en los totales de los sem
    1–2: Disciplinar suma 134 (PEP 133) y Libre 50 (PEP 51). Los 25 cr de "Internados Electivos" van
    dentro de la libre. Prerrequisitos por secuencia (las flechas A–I no se modelaron).
  - **Enfermería** (161 cr, Acuerdo 010 de 2012): malla del anexo 1 del PEP (`PEP-enfermeria.pdf`, págs. 24–25 del PDF); B 47 (32 Cs. Básicas + 6 Salud Pública + 5 Proceso Inv. + 4 Adm.) / C 82 / L 32, todo cuadra. Las 6 "Enfermerías" solo piden las Prácticas y Habilidades Avanzadas. Nivelación (20 cr) fuera. Los totales impresos por semestre 4 y 5 del PDF no cuadran con sus cajas (las sumas por componente sí).
  - **Odontología** (180 cr, plan 2557, Acuerdo 05 de 2023): malla vigente en `odontologia.unal.edu.co/wp-content/uploads/2026/06/Malla-Pregrado-2.pdf` (las URL viejas de `/docs/pregrado/` dan 404). B 53 / C 91 (incluye 27 Clínicas) / L 36 (24 libre + 12 profundización, todo en el contador). Prerrequisitos leídos de las flechas numeradas (circulitos); TG con corte `{comp:"C",min:73}` (leyenda "#"). Se omitió una flecha ambigua (15 → Salud Pública). Nivelación fuera.
  - **Derecho** (180 cr, Acuerdo 048 de 2008): malla del PEP (pág. 13 del PDF), sin códigos ni flechas (ids `DER_*`); B 36 / C 108 / L 36. Prerrequisitos **inventados por secuencia** (I/II, penal, civil, procesal). Preparatorio Básico (0 cr) e Inglés fuera. Sem 9 = solo Consultorio II; sem 10 = TG.
  - **Ciencia Política** (126 cr, Acuerdo 098 de 2008; clave `ciencia-politica`, misma facultad que Derecho): malla del PEP `pep_2_29.pdf` (pág. 33). B 30 / C 66 (con TG) / L 30. Según el PEP solo hay dos prerrequisitos: 24 cr de B (80 %) para todo lo disciplinar y 101 cr totales (80 %) para el TG. Sem 7 = 100 % libre (`m:[]`, `.tg-libre`). La malla es de 2008; puede haber un plan más nuevo.
  - **Ciencias Humanas** (13): `trabajo-social` (148), `antropologia` (122), `psicologia` (150), `historia` (121), `sociologia` (131), `linguistica` (123), `filologia-clasica` (125), `estudios-literarios` (119), `filologia-aleman`/`-frances`/`-ingles` (140, misma estructura generada con `semsFilologia()`), `filosofia` (122), `geografia` (128). Fuentes: PEP de `diracad…/PEP-<slug>.pdf` y `pregrado.unal.edu.co/docs/pep/pep_2_<n>.pdf` (Antropología 27, Sociología 26, Estudios Literarios 35, Filología e Idiomas 37, Historia 38, Español y Filología Clásica 44, Lingüística 46, Psicología 2); Filosofía del plan_de_estudios.pdf del Departamento; Geografía de la imagen `Semaforo.jpeg` del Departamento. Psicología, Estudios Literarios, Filología e Idiomas y Filosofía **no traen semestres**: la ubicación es propuesta. Antropología: créditos por materia repartidos de los totales por agrupación.
  - **Ciencias Agrarias**: `agronomica` (180, Acuerdo 032 de 2014, PEP `pep_2_40.pdf`); el plan cambió con el Acuerdo 04 de 2022, no consultado.
  - **Medicina Veterinaria y de Zootecnia**: `veterinaria` (209, 12 semestres; sem 11 = Práctica Integrada dentro de la libre, `m:[]`) y `zootecnia` (186, anexo 3 del PEP).
  - **Biología** (`biologia`, 163): códigos, créditos y prerrequisitos del **Acuerdo 133 de 2017** del Consejo de Facultad de Ciencias (texto pegado por el usuario desde legal.unal.edu.co, que tiene CAPTCHA) + semestres de la malla de la Facultad; las 55 cr de optativas disciplinares van como 14 cupos; TG pide 68 B y 44 C.
  - **Artes** (7): `arquitectura` (179, Res. 072 de 2010, PEP `pep_2_31`), `artes-plasticas` (166, `pep_2_47`; **créditos por materia estimados**: materias 3, talleres 6), `cine-tv` (158, `pep_2_36`), `diseno-grafico` (148, `pep_2_41`, con prerrequisitos reales), `diseno-industrial` (169, `pep_2_32`; reparto de los 42 cr optativos aproximado), `musica` (171, `pep_2_42` págs. 18–21) y `musica-instrumental` (183, `pep_2_43` págs. 21–25). Las dos de Música se generan con `semsDeSeries()`. Casi todas sin semestres oficiales: secuencia propuesta.
  - **Carreras con varios planes (`GRUPOS`)**: Música (4 énfasis: `musica` Dirección, `musica-jazz`, `musica-pedagogia`, `musica-composicion`) y Música Instrumental (5: `musica-instrumental` cuerdas/percusión/vientos, `musica-arpa`, `musica-canto`, `musica-guitarra`, `musica-piano`). Cada plan es una entrada de `MALLAS` con `grupo` y `planN` (hash y `store` propios); las derivadas se crean con `_plan()` después del registro, y los datos con `semsMusica()` / `semsInstrumental()`. El inicio muestra **una tarjeta por grupo** (`renderInicio` filtra al primer plan y enlaza a `planPreferido()`, el último usado: `localStorage["malla_plan_<grupo>"]`), y la malla muestra el selector `#m-planes` (píldoras `.planes`), oculto en carreras sin grupo. El contador "N disponibles" cuenta carreras, no planes. Filología e Idiomas (3 tarjetas) podría pasarse a este mecanismo.
  - **No agregadas todavía**: **Terapia Ocupacional** (el PEP solo trae las materias por agrupación, sin
    semestres), Fisioterapia (no está en la oferta 2027-01).
- **Alturas de sem 9/10**: en carreras sin `sem9fix` (todas menos Civil) el `.tailgroup` lleva la
  clase `.tg-fill` → `.tailcols` crece y `.agrkey` pasa a `flex:0 0 auto` (la clave de colores no
  queda con espacio sobrante). Civil conserva su ajuste fino. El apaño de Agrícola (celdas de
  sem 9/10 a 96/118 px exactos) está scopeado a `#view-malla.malla-agricola`.
- `PENSUM_BORRADOR.md` — extracción cruda del PDF de Civil + notas de verificación.
- `PLAN_ING (2).pdf` — malla oficial de Civil.
- `serve.mjs` — servidor estático dev (:4179).

Para una carrera nueva: agregar `SEMS_<X>`/`AGR_<X>` + una entrada en `MALLAS`, activar su
tarjeta en la grilla del inicio (`<a class="carrera" href="#<x>">`). Tras cualquier cambio,
`cp malla-artifact.html index.html` para no desincronizar.

## Modelo de prerrequisitos

`r` es un AND de:
- **id de otra materia** (string) → tiene que estar aprobada.
- **corte de créditos** `{comp:"C", min:67}` → créditos aprobados del componente C ≥ min.
  `comp:"T"` = créditos **totales** del plan (usado en Química); `comp:"L"` = Libre Elección.

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
  El recuadro (`.elec`) toma el borde verde (`.elec.done`) al llegar a 36, igual que una materia
  aprobada; `setElec()` y `elecWidget()` alternan la clase.
- **Las optativas van en su semestre** como materias con `opt:true` (solo se distinguen por el
  encabezado de categoría "OPTATIVA"): Optativa de Herramientas de Ingeniería en sem 2 y 5
  (comp B), Electiva Técnica en sem 6 (comp C, corte 28 C), Construcción de Obras Civiles en
  sem 10 (comp C, prereq Fundamentos de Construcción). No se elige la materia concreta salvo
  por el nombre del cupo.
- **Total definido = 180 cr exactos** (126 obligatorias + TG 6 + 6 opt B + 6 opt C + 36 L).
  Barra de progreso `min(x/180, 100)`.
- **Prerrequisitos**: verificados uno por uno con el usuario contra su experiencia en la carrera
  (ver historial). Un par quedaron distintos a lo que sugería el diagrama del PDF (p. ej.
  Mecánica de Sólidos depende de Estática + Cálculo en Varias Variables, no de Dinámica;
  Dinámica abre Análisis Estructural Aplicado).
- **Columnas de semestre justificadas**: los semestres 1–8 estiran sus materias (`.m` con
  `flex:1`) para quedar todos a la misma altura; los semestres 9 y 10 (≤3 materias) llevan la
  clase `.short` y quedan a su altura natural.
- **Barra superior (`.topbar`)**: fija arriba, con marca a la izquierda (glifo SVG + "Malla
  curricular" / "Ing. Civil · U. Nacional") y a la derecha: botón **"Invítame un café"**
  (`.coffee`, pill color café), **alternador de tema** (`#temaBtn`, iconos sol/luna en SVG) y
  **"Reiniciar"** (antes estaba en el `<header>`). El `href` del café es un **placeholder**
  (`buymeacoffee.com/`) — hay un `<!-- TODO -->` al lado; cambiarlo por la URL real (BMC / Ko-fi /
  Nequi). Sin librerías de iconos: todos los SVG van inline.
- **Tema claro / oscuro**: todo el color sale de tokens en `:root` (oscuro por defecto). El tema
  claro solo redefine esos tokens en `:root.theme-light`. Un `<script>` chico justo después del
  `</style>` (antes del contenido) fija la clase `theme-light` en `document.documentElement`
  **antes de pintar**, según `localStorage["malla_tema"]` o, si no hay, `prefers-color-scheme`.
  `aplicarTema()` alterna la clase y persiste. Tokens que se agregaron para poder tematizar:
  `--ready` (borde "disponible"), `--locked` (borde "bloqueada"), `--tint` (24% oscuro / **45%**
  claro: cuánto color de área lleva el fondo de la celda; en claro el usuario lo quiso más
  cargado que un pastel pero sin romper el texto oscuro), `--agr-filter` (realce del color de
  área en textos), `--coffee`, `--glow`, `--shadow`. Las `.badge` y varios bordes pasaron de
  `rgba()` fijos a `color-mix(... var(--token) ...)`. El **`.colh`** (encabezado de semestre) ya
  **no es la caja azul fuerte** (chocaba con el resto en los dos temas): ahora es una caja con
  `--surface2` + borde `--line`, "SEMESTRE N" en `--text` y los créditos en `--dim`.
  `index.html` usa nombres de token distintos (`--primary/--border/--compX/--success…`) pero la
  misma mecánica.
- **Estética**: oscuro por defecto (con alternador a claro), cero emojis (solo el favicon del
  Artifact y el ✓ de aprobada).
  - **Color de celda = agrupación** (el área de la malla oficial: Matemáticas azul, Hidráulica
    cian, Estructuras gris, Geotecnia café, Construcción rosa, Vías naranja, etc.). El mapa está
    en la constante `AGR` (`clave → [color, nombre, etiqueta corta]`) y cada materia trae `a:`.
    Fondo = `color-mix(agr 24%, surface)`, sin franja ni adornos (solo el color).
  - **Estado = solo el color del borde** (grueso, 3px): bloqueada → rojo tenue + atenuada;
    falta 1 → ámbar; disponible → **blanco** (no un verde que se confunde con aprobada); aprobada
    → verde + ✓, conservando el color del área.
  - En el header solo la leyenda de estados. La **clave de colores de agrupación** (`agrkeyEl()`)
    se mete en un `.tailgroup` junto a los semestres 9 y 10, llenando el espacio libre de esas
    columnas cortas. En el `.tailcols` los semestres 9 y 10 se estiran a la misma altura y sus
    materias llevan `flex:1` para repartirse el alto; el contador de electivas queda a la misma
    línea base que la última materia del sem 9.
  - **Alineación renglón a renglón de sem 9 y 10**: en render, solo para el sem 9 (`i ===
    SEMS.length - 2`), se le pone `min-height` a las dos casillas del bloque Disciplinar
    (`ms[0]` Fundamentos ≈124px artifact / 118px index, `ms[1]` Ingeniería Civil Sostenible
    105px) para que Sostenible quede enfrente de "Construcción de Obras Civiles" del sem 10.
    Al crecer Fundamentos, "Fundamentación" cae sola a la altura de "Libre Elección" — ya **no
    se usa el `.tspacer`** de antes. No se toca el ancho ni el sem 10. Valores a ojo, medidos en
    el navegador; si algún cambio de tipografía los descuadra, reajustar esos dos números.
  - Sin fondo cuadriculado (solo un glow suave arriba). Tipografía: **Montserrat** (una sola
    familia para todo; el usuario descartó Inter/Archivo por "olor a IA").

## Pendiente

- **Placeholders por reemplazar**: URL del botón "Invítame un café" y el `mailto:` de "¿falta
  tu carrera?" (los dos en `malla-artifact.html` / `index.html`).
- **Nombre/marca**: hoy la marca es sólo "Malla" — el usuario aún no eligió nombre definitivo.
- Las 9 ingenierías ya están **verificadas con estudiantes** (`ver:true`; así lo dice el pie de cada
  una); las otras 41 están **sin verificar** (`ver:false`) — cuando
  un estudiante confirme una, pasar su `ver` a `true` y reescribir su pie.
- **Faltan por agregar** (pedido del usuario: todas las carreras de la Sede Bogotá): Terapia Ocupacional y Fisioterapia. Ver arriba por qué las primeras tres
  quedaron pendientes.
- Confirmar créditos exactos de un par de materias si el usuario nota algo raro.
- ¿Modelar Prácticas / Electivas de Profundización con su corte de 108 / 126 créditos totales?
- Publicar (Netlify / GitHub Pages) — ya es la carpeta entera, no un archivo suelto.
- ¿Publicar también `index.html` como Artifact? (hoy sólo se publica `malla-artifact.html`).
