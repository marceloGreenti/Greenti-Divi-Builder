# Mapeo HTML → módulos Divi

Este documento es la tabla de decisión que usa el `divi-json-builder` para decidir qué módulo Divi emitir según cada patrón HTML de entrada.

## Principios generales

1. **Preferencia por módulos nativos.** Solo usar Code Module cuando ningún módulo nativo cubre el patrón (ver `code-module-triggers.md`).
2. **Semántica sobre presentación.** Si el HTML tiene `<button>`, emitir módulo `button` aunque visualmente parezca un link.
3. **Un módulo Divi puede reemplazar varios elementos HTML.** Ejemplo: un blurb (`<div class="feature-card">` con ícono + título + descripción) va en un solo módulo `blurb`, no en 3 módulos separados.
4. **Preservar jerarquía semántica.** `<h1>` no se convierte en `<h2>` en el JSON. El `headingLevel` se preserva.

## Tabla de mapeo — Elementos HTML → módulos Divi

### Estructura

| Patrón HTML | Módulo Divi | Notas |
|---|---|---|
| `<body>` (raíz) | wrapper `placeholder` | Obligatorio en Divi 5. |
| `<section>` | `section` | Uno por cada sección semántica del contenido. |
| `<div class="container">`, `.wrapper`, `.row` | `row` | Contenedor de columnas dentro de section. |
| `<div class="col">`, `.column`, `[flex: 1]` | `column` | Con `type` fraccional + `flexType` N_24. |
| `<div class="grid">` con hijos flex | `row` con columnas por hijo | Detectar cantidad de columnas por el layout. |
| `<div>` con `display: flex; flex-direction: column` que agrupa elementos | `group` | Nuevo en Divi 5, wrapper flex/grid. |
| `<div>` con `display: flex` que agrupa elementos horizontalmente | `group` con `flexDirection: row` | Idem. |
| `<article>` | `column` o `group` según contenido | Depende del contexto. |
| `<aside>` | Specialty section o `column` lateral | Solo si el layout lo justifica. |
| `<header>` | `divi-import-header.json` separado | Ver "Separación header/footer". |
| `<footer>` | `divi-import-footer.json` separado | Ver "Separación header/footer". |
| `<nav>` (top level) | Va dentro de header, no en la página. |

### Encabezados

| Patrón HTML | Módulo Divi | Notas |
|---|---|---|
| `<h1>` standalone | `heading` con `headingLevel: h1` | Solo uno por página. |
| `<h2>...<h6>` standalone | `heading` con nivel correspondiente | |
| `<h1>` + `<p>` juntos con estilo unificado | `text` con innerContent HTML | Cuando hay párrafo asociado directamente. |
| `<h2>` + `<p>` + `<button>` (bloque CTA) | `cta` | Cuando semánticamente es un llamado a la acción completo. |

### Texto

| Patrón HTML | Módulo Divi | Notas |
|---|---|---|
| `<p>` standalone | `text` | Emite el `<p>` en innerContent. |
| Múltiples `<p>` en un mismo bloque | `text` único con todo el HTML | No fragmentar en múltiples módulos text. |
| `<ul>`, `<ol>` | `text` con la lista en innerContent | O `icon-list` si tiene íconos por item. |
| `<blockquote>` | `text` con blockquote en innerContent | O `testimonial` si hay autor + cargo + empresa. |

### Botones y CTAs

| Patrón HTML | Módulo Divi | Notas |
|---|---|---|
| `<button>` standalone | `button` | |
| `<a class="btn">` | `button` | El estilo de botón lo hace el módulo. |
| Bloque con título + descripción + botón | `cta` | Composición típica de hero o feature. |

### Imágenes y media

| Patrón HTML | Módulo Divi | Notas |
|---|---|---|
| `<img>` standalone | `image` | Con alt obligatorio. |
| `<img>` dentro de `<figure>` con `<figcaption>` | `image` con caption en attributes o Code Module | Divi image no tiene caption nativo. Depende del caso. |
| `<picture>` con múltiples `<source>` | `image` (Divi genera responsive automático) | La skill usa el `<source>` desktop como principal. |
| `<video>` con URL YouTube/Vimeo | `video` | |
| `<video>` con MP4 hosted | `video` con URL a MP4. | |
| `<audio>` | `audio` | |
| Galería de imágenes (`<div class="gallery">` con múltiples `<img>`) | `gallery` | |
| Comparador antes/después | `before-after-image` | Muy usado en portfolios. |
| Ícono individual (SVG inline o Font Awesome) | `icon` | Nuevo módulo en Divi 5. |
| Ícono + texto + descripción (feature card) | `blurb` | |

### Formularios

| Patrón HTML | Módulo Divi | Notas |
|---|---|---|
| `<form>` (cualquiera) | `code` (Code Module) con placeholder CF7 | Ver `code-module-triggers.md`. Greenti no usa formularios Divi. |
| Input search con label "buscar" | `search` | Solo si es la búsqueda WP nativa. |
| Formulario de login | `login` | Solo si es login WP nativo. |

### Componentes interactivos

| Patrón HTML | Módulo Divi | Notas |
|---|---|---|
| `<details><summary>` (acordeón nativo HTML) | `accordion` con `accordion-item`s | O `toggle` si es uno solo. |
| Tabs (`<div role="tablist">` + panels) | `tabs` con `tab`s | |
| Slider/carousel de contenido | `slider` con `slide`s | |
| Slider/carousel de imágenes | `gallery` en modo slider | O `slider` con imágenes como fondo. |
| Slider/carousel de videos | `video-slider` con `video-slider-item`s | |
| Dropdown/modal genérico | `dropdown` con `group` adentro | Nuevo en Divi 5. |
| Timeline/línea de tiempo | `timeline` con `timeline-item`s | Nuevo en Divi 5. |
| Countdown a fecha | `countdown-timer` | |
| Contador con número grande + label | `number-counter` | |
| Contador con progreso circular | `circle-counter` | |
| Barras de progreso animadas | `counters` con `counter`s | |
| Toggle simple abrir/cerrar | `toggle` | |

### Contenido especializado

| Patrón HTML | Módulo Divi | Notas |
|---|---|---|
| Testimonial con cita + autor + cargo + empresa | `testimonial` | |
| Card de miembro del equipo (foto + nombre + cargo + bio + redes) | `team-member` | |
| Tabla de precios (planes con nombre + precio + features + botón) | `pricing-tables` con `pricing-table`s | |
| Lista con íconos por item | `icon-list` con `icon-list-item`s | |
| Grid de posts del blog | `blog` | |
| Grid de portfolio items | `portfolio` o `filterable-portfolio` | Filterable si tiene filtros. |
| Slider de posts | `post-slider` | |
| Mapa con pines | `map` con `map-pin`s | |
| Feed de Instagram embebido | `instagram-feed` | Requiere configuración de API. |
| Tabla de contenidos automática de post | `table-of-contents` | Nuevo en Divi 5. |
| Comentarios de post | `comments` | Solo en single post. |
| Título dinámico de post | `post-title` | Solo en Theme Builder templates. |
| Contenido dinámico de post | `post-content` | Solo en Theme Builder templates. |

### Redes sociales

| Patrón HTML | Módulo Divi | Notas |
|---|---|---|
| Grupo de iconos de redes sociales | `social-media-follow` con `social-media-follow-network`s | Cada red social es un item. |

### Separadores y utilidad

| Patrón HTML | Módulo Divi | Notas |
|---|---|---|
| `<hr>` | `divider` | |
| Sidebar de WordPress | `sidebar` | Solo si es la sidebar WP nativa. |
| Menú de navegación WP | `menu` | Requiere que el menú esté registrado en WP. |

### Hero fullwidth

| Patrón HTML | Módulo Divi | Notas |
|---|---|---|
| Hero con imagen de fondo + título grande + subtítulo + hasta 2 botones | `fullwidth-header` | Módulo dedicado. |
| Hero simple con título + botón | `section` normal con configuración de fullwidth | Ver `divi5-reference.md` sección 4.7. |

## Separación de header, footer y body

### Header

Detectar por:
- Etiqueta `<header>` semántica.
- Clase `.header`, `.site-header`, `.main-header`.
- Bloque que contiene `<nav>` principal y logo, usualmente en la parte superior.

El contenido del header se emite en `divi-import-header.json` con la misma estructura (placeholder → section → row → column → módulos).

### Footer

Detectar por:
- Etiqueta `<footer>` semántica.
- Clase `.footer`, `.site-footer`, `.main-footer`.
- Bloque en la parte inferior que contiene información de contacto, enlaces secundarios, redes sociales, copyright.

Se emite en `divi-import-footer.json`.

### Body (contenido principal)

Todo lo que no es header ni footer va en `divi-import-page.json`.

## Reglas transversales de mapeo

### Reglas transversales de mapeo

#### Módulos con fondo transparente obligatorio por defecto

Ciertos módulos de Divi aplican un fondo por defecto (blanco u otro) cuando no se declara `background.color`. Esto rompe el diseño cuando la sección/row ya define el color de fondo (patrón habitual). Para evitarlo, la Skill emite estos módulos **siempre** con `background.color: "transparent"` en el grupo `module.decoration`, salvo que el HTML declare un color de fondo específico para el módulo:

- **`menu`** — el fondo por defecto es blanco. Casi siempre está dentro de un header con fondo propio. **Regla: siempre emitir con `background.color: "transparent"`** salvo instrucción explícita en contrario.
- **`login`** — mismo patrón.
- **`search`** — mismo patrón cuando está inline en un header o footer con fondo propio.
- **`sidebar`** — cuando el fondo de la columna/row ya está definido.

**Ejemplo de menu correcto:**

```json
{
  "module": {
    "decoration": {
      "background": {
        "desktop": { "value": { "color": "transparent" } }
      }
    }
  },
  "logo": {
    "innerContent": { "desktop": { "value": { "src": "...", "alt": "..." } } }
  },
  "menu": {
    "advanced": { "menuId": { "desktop": { "value": "none" } } }
  },
  "builderVersion": "5.8.1"
}
```

Esto vale también cuando el módulo se anide dentro de un `group` con fondo propio.

### CSS externo o `<style>`

Los estilos se leen del `<style>` interno o inline styles. La Skill no procesa archivos CSS externos.

Si un HTML tiene `<link rel="stylesheet" href="styles.css">`, avisar al usuario: "El HTML referencia un CSS externo. Para que los estilos se apliquen, pega el contenido del CSS en un `<style>` dentro del HTML."

### Clases utilitarias (Tailwind, Bootstrap)

Detectar clases utilitarias comunes y mapearlas a propiedades Divi:

- `flex`, `grid` → `layout.display`.
- `p-4`, `px-8`, `mt-16` → `spacing.padding/margin`.
- `text-2xl`, `text-lg` → `font.size` según escala.
- `bg-*`, `text-*`, `border-*` → colores (requiere tener el mapeo de la paleta configurada).
- `rounded-*` → `border.radius`.
- `shadow-*` → `boxShadow`.

Si el HTML usa Tailwind y no hay `design-tokens.md` con la paleta, la Skill lo detecta y pide al usuario que confirme los colores base antes de continuar.

### Elementos custom (`<mi-widget>`, Web Components)

Si el HTML tiene elementos personalizados (nombre con guión), la Skill los detecta y decide:
1. Si tienen contenido HTML interno estándar, procesar el contenido.
2. Si son componentes con JS interno (custom elements), emitir como Code Module con el HTML tal cual y avisar en `notes.md`.

### Contenido dinámico (ACF, campos custom)

Si el HTML tiene marcadores tipo `{{campo}}`, `[campo]`, `{{ACF:nombre}}`, la Skill detecta y avisa que no soporta contenido dinámico en v1. El usuario debe reemplazar por texto estático o configurar dinámico manualmente en Divi tras importar.

## Cuándo NO mapear directamente

En casos ambiguos, el `divi-json-builder` **pregunta al usuario** antes de decidir. Ejemplos:

- HTML que parece ser un carousel pero no está claro si es slider genérico, gallery, post-slider o video-slider.
- HTML con estructura de columnas irregular (mezcla de 2 y 3 columnas en la misma sección).
- Componente interactivo custom que podría mapearse a varios módulos nativos.

## Log de mapeo

Toda decisión de mapeo se registra en el log del builder, que la Skill copia a `notes.md`:

```
=== HTML → Divi Mapping Log ===

- <section class="hero"> → section "Hero Section"
- <div class="row"> con 2 hijos flex → row con columnStructure "1_2,1_2"
- <h1>Impulsa tu presencia</h1> → cta.title (parte de bloque CTA compuesto)
- <p>Ayudamos a marcas...</p> → cta.content
- <button>Empezar ahora</button> → cta.button
- <img src="hero-bg.jpg"> → section.decoration.background.image (usada como fondo)
- <div class="feature-card"> → blurb "UX/UI a medida"
- <form id="contacto"> → code module "Formulario CF7 - reemplazar shortcode"
```
