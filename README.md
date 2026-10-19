# Malla — seguimiento del avance curricular

Herramienta web para visualizar el avance dentro de un plan de estudios: qué asignaturas están
aprobadas, cuáles están disponibles, cuáles siguen bloqueadas y por qué (prerrequisitos y cortes
de créditos), con el porcentaje de avance total y por componente (Fundamentación, Disciplinar y
Libre elección).

Cubre las 50 carreras de pregrado de la Sede Bogotá de la Universidad Nacional de Colombia
(60 planes de estudio contando énfasis e instrumentos).

> Proyecto estudiantil independiente. No es una herramienta oficial de la Universidad Nacional de
> Colombia. Ante cualquier duda sobre un plan de estudios, la fuente válida es el SIA.

## Funciones

- Malla por semestres con el color de cada agrupación de la malla oficial.
- Clic para aprobar una materia; se desbloquean las que dependen de ella.
- Prerrequisitos por materia y por cortes de créditos (por componente o totales).
- Invalidación en cascada: al desmarcar una materia se desmarca todo lo que dependía de ella,
  con opción de deshacer.
- Al tocar una materia bloqueada se indica qué falta y se señalan esos prerrequisitos.
- Al señalar una materia se marcan las que abre.
- Contador de créditos de libre elección.
- Selector de plan para carreras con varios énfasis o instrumentos (Música, Música Instrumental,
  Filología e Idiomas).
- Inicio con buscador de carreras y acceso directo a la última malla abierta.
- Tema claro y oscuro, impresión o PDF de la malla y enlace para compartir.
- El avance se guarda en el navegador (`localStorage`): no hay cuentas ni servidor.

## Tecnología

- Un solo archivo HTML con CSS y JavaScript sin frameworks ni dependencias.
- Datos de cada plan reconstruidos de los documentos oficiales (mallas, PEP, acuerdos y
  resoluciones). Para leer los PDF se usaron scripts de Node.js con `mupdf` y `sharp`.

## Cómo correrlo

```bash
node serve.mjs      # http://localhost:4179
```

O abrir `index.html` directamente en el navegador.

`index.html` y `malla-artifact.html` son el mismo archivo. `CLAUDE.md` documenta en detalle la
estructura del código, las fuentes de cada carrera y las decisiones tomadas.

## Estado de los datos

- Las 9 ingenierías están verificadas con estudiantes de cada carrera.
- Las otras 41 carreras están reconstruidas de los documentos oficiales y marcadas en la app como
  «Sin verificar». El pie de cada malla explica la fuente y las aproximaciones usadas.

## Autoría

Concepción, diseño funcional y de interfaz, definición de reglas académicas, recopilación y
validación de datos, pruebas y dirección del desarrollo: Julián García (2026).
Implementado con asistencia de IA (Claude, de Anthropic).
