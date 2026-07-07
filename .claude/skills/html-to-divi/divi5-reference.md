# Divi 5.0 — Reference Documentation

**Versión Divi objetivo:** 5.8.1
**Versión del documento:** v1.1
**Última actualización:** consolidación completa tras análisis de 9 exports reales (2 iniciales + 7 lotes de catálogo).
**Uso:** Este documento es la fuente de verdad técnica sobre la estructura interna de los archivos de exportación de Divi 5.0. Lo consulta el subagente `divi-json-builder` durante la fase de emisión del JSON. También es referencia obligatoria para `divi-qa-validator` durante la fase de validación.

---

## 1. Contexto y objetivos

Este documento describe la arquitectura del archivo JSON que Divi 5.0 usa para exportar e importar layouts. El objetivo del pipeline `html-to-divi` es generar archivos que:

1. Sean **importables sin errores** en Divi 5.8.1 vía Divi Library.
2. Queden **completamente editables desde el constructor visual** de Divi (todos los módulos deben ser nativos, salvo excepciones documentadas).
3. **Respeten los estilos definidos en el HTML de entrada** sin interpolar ni "adivinar" valores.
4. **Avisen explícitamente** cuando un patrón de diseño requiera Code Module o desarrollo custom.

### 1.1 Cambios importantes en v1.1

Esta versión consolida todo lo aprendido después de analizar exports reales de todos los módulos disponibles en Divi 5.8.1. Cambios clave respecto a v1.0:

- **Catálogo pasa de 24 a 48 módulos con schema real y verificado.**
- **Sistema de 5 breakpoints** (`desktop`, `tabletWide`, `tablet`, `phoneWide`, `phone`) en vez de 3.
- **Sistema `columnStructure` + `flexColumnStructure`** documentado (nueva sintaxis de Divi 5).
- **Sistema `flexType` de 24 columnas** paralelo a `advanced.type` (fracciones N_M).
- **Variables globales** (`$variable(...)$`) documentadas para colores/tipografías/spacing globales.
- **Patrón "contenedor + item"** generalizado y catalogado.
- **Los módulos `fullwidth-*` no existen** en Divi 5; se logra fullwidth ajustando `sizing.maxWidth` y `sizing.width` de section/row.
- **Módulos hijos nuevos capturados:** `map-pin`, `accordion-item`, `tab`, `slide`, `counter`, `timeline-item`, `icon-list-item`.
- **Módulos nuevos de Divi 5** que no existían en Divi 4: `heading`, `group`, `group-carousel`, `icon-list`, `timeline`, `dropdown`, `before-after-image`, `instagram-feed`, `table-of-contents`.

---

## 2. Arquitectura del archivo de exportación

Un archivo exportado desde Divi 5 es un objeto JSON con **8 llaves top-level**:

```json
{
  "context": "et_builder",
  "data": { "<page_id>": "<HTML string con bloques Gutenberg de Divi>" },
  "presets": null,
  "global_colors": [],
  "global_variables": [],
  "page_settings_meta": null,
  "canvases": { "local": [], "global": [] },
  "images": {},
  "thumbnails": []
}
```

### 2.1 Descripción de cada llave

| Llave | Tipo | Descripción | Uso en v1 |
|---|---|---|---|
| `context` | string | Siempre `"et_builder"`. Identifica el archivo como export de Divi Builder. | Valor fijo. |
| `data` | objeto | Contiene el contenido de la página como un string HTML con bloques Gutenberg de Divi. La llave interna es el ID de la página en WordPress (ej. `"161"`); para importaciones nuevas usamos un ID genérico como `"1"`. | Aquí va el 99% del trabajo. |
| `presets` | null / objeto | Estilos guardados reutilizables por tipo de módulo. | Se emite como `null` (fuera de scope v1). |
| `global_colors` | array | Colores globales referenciables desde módulos vía tokens. | Se emite como `[]` (v1 usa colores hardcoded por módulo). Ver sección 5.6. |
| `global_variables` | array | Variables globales (fuentes, espaciados, etc.). | Se emite como `[]` (v1 usa valores hardcoded). |
| `page_settings_meta` | null / objeto | Configuración de la página (custom CSS de página, etc.). | Se emite como `null` salvo casos específicos. |
| `canvases` | objeto | Canvases (Theme Builder templates aplicables). | Se emite como `{"local": [], "global": []}`. |
| `images` | objeto o array | Imágenes embebidas en base64 (usado en exports "completos"). | Se emite como `{}` — imágenes se referencian por URL, deben subirse al Media Library antes de importar. |
| `thumbnails` | array | Miniaturas del layout para el Divi Library. | Se emite como `[]`. |

### 2.2 Ejemplo de esqueleto vacío importable

```json
{
  "context": "et_builder",
  "data": {
    "1": "<!-- wp:divi/placeholder --><!-- wp:divi/section {\"builderVersion\":\"5.8.1\"} /--><!-- /wp:divi/placeholder -->"
  },
  "presets": null,
  "global_colors": [],
  "global_variables": [],
  "page_settings_meta": null,
  "canvases": { "local": [], "global": [] },
  "images": {},
  "thumbnails": []
}
```

Este es el archivo mínimo importable en Divi Library: una página con una sección vacía como slot inicial.

### 2.3 Sobre `global_colors` reales encontrados

En exports reales de Divi 5.8.1 aparece este patrón cuando el usuario tiene colores globales configurados:

```json
"global_colors": [
  ["gcid-body-color", { "color": "#666666", "status": "active", "label": "Color del Texto Principal" }],
  ["gcid-link-color", { "color": "#2ea3f2", "status": "active", "label": "Color Del Enlace" }]
]
```

Formato: array de arrays `[id, { color, status, label }]`. El `id` comienza con prefijo `gcid-` (Global Color ID). Los módulos los referencian mediante la sintaxis `$variable(...)$` (ver 5.6). **En v1 la Skill no emite global_colors** (los colores van hardcoded), pero debe **respetar los global_colors si vienen en el HTML** de entrada con notación de variable, y avisarlo.

---

## 3. Sintaxis de bloques Gutenberg de Divi 5

Divi 5 abandonó los shortcodes de Divi 4 y ahora usa la sintaxis de bloques nativa de WordPress (Gutenberg). Cada módulo es un bloque HTML-comentario con este formato:

### 3.1 Bloque con contenido anidado (para contenedores)

```
<!-- wp:divi/NOMBRE_MODULO {JSON_DE_CONFIGURACION} -->
  ...bloques hijos aquí...
<!-- /wp:divi/NOMBRE_MODULO -->
```

### 3.2 Bloque self-closing (para módulos hoja)

```
<!-- wp:divi/NOMBRE_MODULO {JSON_DE_CONFIGURACION} /-->
```

Nota la barra `/` **antes** del `-->` final. Esta es la diferencia sintáctica clave.

### 3.3 Wrapper obligatorio `placeholder`

**Todo el contenido de la página debe ir envuelto** en un bloque `placeholder`:

```
<!-- wp:divi/placeholder -->
  ...todo el contenido de la página...
<!-- /wp:divi/placeholder -->
```

El `placeholder` no lleva JSON de configuración. Es un contenedor técnico que Divi 5 usa para marcar el árbol raíz de contenido gestionado por el builder. Verificado en el 100% de los exports analizados.

### 3.4 Reglas de escapado del JSON

El JSON de configuración va **inline** dentro del comentario HTML. Reglas críticas:

- Las **comillas dobles** del JSON van tal cual (no se escapan) cuando se emiten al string HTML.
- Los caracteres `<`, `>`, `&` **no deben aparecer sin escape** dentro del JSON. Si contenido HTML entra en un `innerContent`, se codifica: `<p>` → `\u003cp\u003e`, `&` → `\u0026`, `"` → `\"`, `'` → `\u0027`, `#` → `\u0023`.
- **Saltos de línea** entre bloques hermanos se preservan (Divi usa `\r\n` en los exports).
- El JSON debe ser **serializable de forma estricta**: sin comas colgantes, sin comentarios, sin funciones.
- Los **iconos** se codifican con formato Unicode escape: `&#xe0fd;` se emite como `\u0026#xe0fd;` (donde el `&` inicial se codifica en Unicode escape también).

### 3.5 Ejemplo de codificación real

Este fragmento de contenido:
```html
<p>Hola mundo</p>
```

Se emite dentro del JSON como:
```
"value":"\u003cp\u003eHola mundo\u003c\/p\u003e"
```

El `/` también se escapa a `\/` en los exports reales.

---

## 4. Jerarquía de contenido

Divi 5 impone una jerarquía estricta:

```
placeholder                          (wrapper obligatorio)
└── section                          (nivel 1: bloque semántico principal)
    └── row                          (nivel 2: fila de columnas)
        └── column                   (nivel 3: columna)
            └── módulos de contenido (nivel 4: text, image, cta, etc.)
                └── (opcional)
                    └── group        (nivel 5: contenedor de agrupación)
                        └── módulos hoja
```

### 4.1 Reglas de anidamiento

- Un **section** contiene 1 o más **row**s.
- Un **row** contiene 1 o más **column**s.
- Un **column** contiene módulos de contenido, o un **group** que agrupa varios módulos.
- Un **group** contiene módulos hoja (nuevo en Divi 5, actúa como wrapper flex/grid dentro de una columna).
- Los tamaños de columna se declaran en `advanced.type.desktop.value` con formato **N_M** (fracciones tradicionales) o en `decoration.sizing.flexType.desktop.value` con formato **N_24** (sistema flex de 24).
- La **suma de fracciones** de columnas dentro de un row debe totalizar 1.

### 4.2 Sistema `columnStructure` + `flexColumnStructure` (nuevo en Divi 5)

En Divi 5, el `row` declara la estructura completa de sus columnas en `advanced.columnStructure` (fracciones separadas por coma) y opcionalmente el layout tipo grid en `advanced.flexColumnStructure`.

Ejemplos observados:

```json
{
  "advanced": {
    "columnStructure": { "desktop": { "value": "4_4" } },
    "flexColumnStructure": { "desktop": { "value": "equal-columns_1" } }
  }
}
```

**Valores de `columnStructure`** (patrones observados en exports reales):

- `4_4` — 1 columna completa
- `1_2,1_2` — 2 columnas iguales
- `1_3,1_3,1_3` — 3 columnas iguales
- `1_4,1_4,1_4,1_4` — 4 columnas iguales
- `1_5,1_5,1_5,1_5,1_5` — 5 columnas iguales
- `1_6,1_6,1_6,1_6,1_6,1_6` — 6 columnas iguales
- `1_12,1_12,...(x12)` — 12 columnas iguales (útil para grids finos)
- `1_3,2_3` / `2_3,1_3` — 2 columnas asimétricas
- `1_4,3_4` / `3_4,1_4` — 2 columnas asimétricas
- `1_4,1_2,1_4` — 3 columnas asimétricas
- `1_5,3_5,1_5` — 3 columnas asimétricas

**Valores de `flexColumnStructure`** (patrones observados):

- `equal-columns_1` — 1 columna
- `equal-columns_2` — 2 columnas iguales
- `equal-columns_3` — 3 columnas iguales
- `equal-columns_4` — 4 columnas iguales
- `offset-columns_1` — columnas offset (asimétricas)
- `multi-column-grids_1` — grid multi-columna
- `multi-row-grids_10` — grid multi-fila (12 items en grid)
- `masonry-grids_1` — layout masonry

### 4.3 Sistema paralelo `flexType` de 24 columnas

Además del sistema de fracciones (`advanced.type`), Divi 5 introduce un **sistema flex de 24 columnas** en `column.decoration.sizing.flexType.desktop.value`:

```json
{
  "decoration": {
    "sizing": {
      "desktop": { "value": { "flexType": "12_24" } },
      "tablet":  { "value": { "flexType": "24_24" } }
    }
  }
}
```

Valores comunes:

- `24_24` — 100% (columna completa)
- `12_24` — 50%
- `8_24` — 33.33%
- `6_24` — 25%
- `4_24` — 16.67%
- `2_24` — 8.33%
- Cualquier `N_24` donde N entre 1 y 24.

**Ambos sistemas coexisten:**
- `advanced.type` con fracciones tradicionales define el ancho "conceptual" de la columna.
- `sizing.flexType` con N_24 define el ancho real en el sistema de 24 columnas.
- Cuando se usan responsive breakpoints, `flexType` permite mucha más granularidad (ej: en móvil una columna que era `1_3` (33%) puede pasar a `24_24` (100%) para stack completo).

**Regla operativa:** cuando emitas una columna, siempre incluye ambos:
```json
{
  "advanced": { "type": { "desktop": { "value": "1_3" } } },
  "decoration": { "sizing": { "desktop": { "value": { "flexType": "8_24" } } } }
}
```

### 4.4 Cuáles módulos son contenedores y cuáles son hoja

**Contenedores estructurales (obligatorios en la jerarquía):**
- `placeholder`
- `section`
- `row`, `row-inner`
- `column`, `column-inner`

**Contenedores de agrupación (opcionales, dentro de columnas):**
- `group` — wrapper genérico flex/grid
- `group-carousel` — wrapper que muestra hijos en carrusel

**Contenedores tipo "colección" (contienen items específicos):**
- `accordion` (contiene `accordion-item`)
- `tabs` (contiene `tab`)
- `slider` (contiene `slide`)
- `video-slider` (contiene `video-slider-item`)
- `pricing-tables` (contiene `pricing-table`)
- `counters` (contiene `counter`)
- `timeline` (contiene `timeline-item`)
- `icon-list` (contiene `icon-list-item`)
- `social-media-follow` (contiene `social-media-follow-network`)
- `contact-form` (contiene `contact-field`)
- `map` (contiene `map-pin`)
- `dropdown` (contiene un `group` con módulos hoja adentro)

**Hoja (self-closing):**
- Todos los demás: `text`, `cta`, `image`, `blurb`, `button`, `heading`, `icon`, `divider`, `code`, `testimonial`, `blog`, `signup`, `contact-field`, `number-counter`, `pricing-table`, `video-slider-item`, `accordion-item`, `tab`, `slide`, `timeline-item`, `icon-list-item`, `counter`, `map-pin`, `search`, `login`, `sidebar`, `comments`, `menu`, `video`, `audio`, `gallery`, `countdown-timer`, `circle-counter`, `toggle`, `team-member`, `portfolio`, `filterable-portfolio`, `post-slider`, `post-title`, `post-content`, `fullwidth-header`, `fullwidth-portfolio`, `before-after-image`, `instagram-feed`, `table-of-contents`, `social-media-follow-network`.

### 4.5 Patrón "contenedor + item" — resumen general

Divi 5 usa un patrón consistente para colecciones. El **contenedor** define el estilo compartido (fuentes, colores base, layout, animaciones globales de la colección) y los **items** definen su contenido individual (título, descripción, media, config específica).

Tabla completa de pares contenedor + item en Divi 5.8.1:

| Contenedor | Item hijo | Uso |
|---|---|---|
| `accordion` | `accordion-item` | FAQs, colapsables |
| `tabs` | `tab` | Pestañas de contenido |
| `slider` | `slide` | Carousel de slides con contenido rico |
| `video-slider` | `video-slider-item` | Carousel de videos |
| `pricing-tables` | `pricing-table` | Planes de precios |
| `counters` | `counter` | Barras de progreso animadas |
| `timeline` | `timeline-item` | Línea de tiempo con eventos |
| `icon-list` | `icon-list-item` | Lista con íconos |
| `social-media-follow` | `social-media-follow-network` | Íconos de redes sociales |
| `contact-form` | `contact-field` | Campos de formulario |
| `map` | `map-pin` | Marcadores geolocalizados en mapa |
| `dropdown` | `group` (que contiene hoja) | Dropdown/modal con contenido |

**Regla operativa para la Skill:** cuando emitas uno de estos módulos, siempre emite el contenedor y al menos 1 item. Un contenedor sin items produce un módulo vacío que puede fallar al renderizar.

### 4.6 Specialty section (layout con sidebar)

Para layouts asimétricos tipo "sidebar + contenido principal", Divi 5 usa **specialty sections**. Se marcan así:

```json
{
  "module": {
    "advanced": {
      "type": { "desktop": { "value": "specialty" } }
    }
  },
  "column1": { "decoration": {...} },
  "column2": { "decoration": {...} },
  "column3": { "decoration": {...} }
}
```

Dentro de una specialty section, las columnas pueden contener **row-inner** (que a su vez contienen **column-inner** con módulos). Esto permite anidar filas dentro de una columna.

Estructura:

```
section (type: specialty)
└── column
    └── row-inner
        └── column-inner
            └── módulos de contenido
```

Las specialty sections son un caso avanzado. La mayoría de layouts se resuelven con sections normales. Solo emitir specialty cuando el diseño lo exige claramente (ej: un aside con múltiples widgets apilados junto a un artículo principal).

### 4.7 Cómo lograr fullwidth en Divi 5 (sin módulos `fullwidth-*`)

En Divi 5 **no existen los módulos `fullwidth-*`** que existían en Divi 4 (fullwidth-header, fullwidth-image, fullwidth-slider, fullwidth-menu, fullwidth-portfolio, fullwidth-post-slider, fullwidth-post-title). El comportamiento fullwidth se logra ajustando la sección y la row:

```json
{
  "module": {
    "decoration": {
      "sizing": {
        "desktop": {
          "value": { "width": "100%", "maxWidth": "100%" }
        }
      },
      "spacing": {
        "desktop": {
          "value": {
            "padding": { "top": "0px", "right": "0px", "bottom": "0px", "left": "0px", "syncVertical": "off", "syncHorizontal": "off" }
          }
        }
      }
    }
  }
}
```

Se aplica en el **row** (y opcionalmente en la **section** también) para lograr el edge-to-edge del contenido.

**Excepción:** `fullwidth-header` y `fullwidth-portfolio` sí existen en Divi 5 como módulos independientes (verificados en los exports). Son módulos "wide" preparados con configuraciones específicas y se documentan en la sección A. Los demás `fullwidth-*` de Divi 4 no existen.

---

## 5. Sistema unificado de propiedades

Divi 5 aplica un sistema muy consistente en todos los módulos, con **4 dimensiones**:

```
[GRUPO] → [CATEGORÍA] → [BREAKPOINT] → [ESTADO]
```

### 5.1 Grupos

Un módulo se compone de "grupos". El grupo `module` siempre existe (representa el módulo entero). Otros grupos son subelementos internos del módulo:

- **Todos los módulos:** `module`
- **CTA:** `module`, `title`, `content`, `button`
- **Text:** `module`, `content`
- **Blurb:** `module`, `imageIcon`, `title`, `content`
- **Image:** `module`, `image`
- **Heading:** `module`, `title`
- **Icon:** `module`, `icon`
- **Testimonial:** `module`, `title`, `content`, `author`, `jobTitle`, `company`, `portrait`, `quoteIcon`
- **Contact Form:** `module`, `title`, `field`, `checkbox`, `radio`, `button`
- **Signup:** `module`, `content`, `button`, `field`, `checkbox`, `radio`, `resultMessage`
- **Number Counter:** `module`, `title`, `number`
- **Circle Counter:** `module`, `title`, `number`
- **Counter (bar counter item):** `module`, `title`, `barProgress`
- **Counters (bar counters container):** `module`, `barProgress`
- **Pricing Tables:** `module`, `title`, `price`, `currencyFrequency`, `content`, `button`
- **Pricing Table:** `module`, `title`, `price`, `currencyFrequency`, `content`, `button`
- **Blog:** `module`, `post`, `title`, `content`, `meta`, `pagination`, `blogGrid`
- **Social Media Follow Network:** `module`, `socialNetwork`, `button`
- **Accordion:** `module`, `title`, `content`, `openToggle`, `closedToggle`, `openToggleIcon`, `closedToggleIcon`
- **Accordion Item:** `module`, `title`, `content`
- **Tabs:** `module`, `tab`, `activeTab`
- **Tab:** `module`, `title`, `content`
- **Slider:** `module`, `navigationArrows`, `navigationDots`
- **Slide:** `module`, `title`, `content`, `image`, `button`
- **Timeline:** `module`, `track`, `item`, `itemEven`, `connector`, `marker`, `children`
- **Timeline Item:** `module`, `date`, `title`, `content`
- **Icon List:** `module`
- **Icon List Item:** `module`, `content`, `icon`
- **Video:** `module`, `video`
- **Video Slider Item:** `module`, `video`, `thumbnail`, `overlay`
- **Audio:** `module`, `title`, `artistName`
- **Gallery:** `module`, `gallery`, `title`, `caption`, `pagination`, `overlay`
- **Countdown Timer:** `module`, `title`, `numbers`, `separator`
- **Menu:** `module`, `logo`, `menu`
- **Search:** `module`, `searchPlaceholder`, `button`
- **Login:** `module`, `title`, `content`, `button`
- **Sidebar:** `module`
- **Comments:** `module`, `image`, `button`, `commentCount`, `meta`, `content`
- **Team Member:** `module`, `image`, `title`, `position`, `content`, `socialNetwork`
- **Portfolio / Filterable Portfolio:** `module`, `image`, `title`, `content`, `meta`, `filter`, `overlay`, `pagination`
- **Post Slider:** `module`, `image`, `title`, `content`, `meta`, `button`, `overlay`
- **Post Title:** `module`, `title`, `meta`, `featuredImage`
- **Post Content:** `module`
- **Fullwidth Header:** `module`, `title`, `subtitle`, `content`, `button1`, `button2`, `image`, `logo`
- **Fullwidth Portfolio:** `module`, `image`, `title`, `overlay`
- **Map:** `module`
- **Map Pin:** `module`, `pin`, `title`, `content`
- **Before/After Image:** `module`, `beforeImage`, `afterImage`, `beforeLabel`, `afterLabel`
- **Instagram Feed:** `module`, `feed`, `followButton`
- **Table of Contents:** `module`
- **Dropdown:** `module`
- **Group:** `module`
- **Group Carousel:** `module`
- **Divider:** `module`, `divider`

Cada grupo se configura independientemente. El grupo `module` afecta al contenedor externo; los grupos internos afectan al elemento correspondiente.

### 5.2 Categorías

Dentro de cada grupo, las propiedades se organizan en 4 categorías:

- **`meta`** — Metadatos del builder (`adminLabel`, `forceVisible`, etc.).
- **`advanced`** — Configuración lógica/funcional específica del grupo (tipo, orientación, atributos HTML, id/class custom, valores como número/fecha, etc.).
- **`decoration`** — Estilo visual (background, spacing, font, border, boxShadow, animation, layout, sizing, dividers, filters, transform, etc.).
- **`innerContent`** — Contenido interno editable (texto, HTML, valores). Solo presente en grupos que llevan contenido.

Además, cada módulo tiene siempre:

- **`builderVersion`** (string) — versión de Divi con la que se generó. Fijo en `"5.8.1"` para la Skill.
- **`locked`** (opcional) — estado de bloqueo de edición.
- **`css`** (opcional) — CSS custom por elemento del módulo.

### 5.3 Breakpoints (5 en total)

Divi 5.8.1 soporta 5 breakpoints (actualización desde v1.0 del reference doc):

| Breakpoint | Ancho típico | Notas |
|---|---|---|
| `desktop` | >= 1281px | Obligatorio en propiedades declaradas. Es la línea base. |
| `tabletWide` | 981px a 1280px | Tabletas grandes / laptops pequeñas. Opcional. |
| `tablet` | 768px a 980px | Tablet estándar. Opcional. |
| `phoneWide` | 480px a 767px | Móviles grandes / phablets. Opcional. |
| `phone` | < 480px | Móvil estándar. Opcional. |

**Regla de herencia** (Divi resuelve automáticamente si no se declara):

1. `tabletWide` hereda de `desktop`.
2. `tablet` hereda de `tabletWide` (o desktop si tabletWide no existe).
3. `phoneWide` hereda de `tablet` (o del anterior disponible).
4. `phone` hereda de `phoneWide` (o del anterior disponible).

**Regla de emisión de la Skill:**

- Siempre declarar `desktop`.
- Declarar breakpoints intermedios **solo cuando el valor difiera** de lo que se heredaría.
- Si el HTML de entrada solo trae `desktop` y `phone`, la Skill infiere los intermedios según `rules/responsive-inference.md`.

**Ejemplo real de export con múltiples breakpoints:**

```json
{
  "decoration": {
    "sizing": {
      "desktop":    { "value": { "flexType": "6_24" } },
      "tabletWide": { "value": { "flexType": "12_24" } },
      "tablet":     { "value": { "flexType": "12_24" } },
      "phoneWide":  { "value": { "flexType": "24_24" } },
      "phone":      { "value": { "flexType": "24_24" } }
    }
  }
}
```

### 5.4 Estados

Cada breakpoint puede tener hasta 3 estados:

- **`value`** — estado normal (obligatorio).
- **`hover`** (opcional) — estado al pasar el mouse.
- **`sticky`** (opcional) — estado cuando la sección está sticky.

**Regla:** hover y sticky son overrides parciales. Solo se declaran las propiedades que cambian. Ej: si en hover solo cambia el color de fondo, se emite solo `{ "hover": { "color": "#..." } }`, no todo el objeto de background.

### 5.5 Ejemplo completo del sistema unificado

Un texto con:
- Familia Poppins tamaño 48px en desktop
- Tamaño 40px en tabletWide
- Tamaño 32px en tablet
- Tamaño 28px en phoneWide
- Tamaño 24px en phone
- Color cambia a rojo al hover

Se emite así:

```json
{
  "content": {
    "decoration": {
      "headingFont": {
        "h1": {
          "font": {
            "desktop":    { "value": { "family": "Poppins", "size": "48px", "color": "#000000" }, "hover": { "color": "#ff0000" } },
            "tabletWide": { "value": { "size": "40px" } },
            "tablet":     { "value": { "size": "32px" } },
            "phoneWide":  { "value": { "size": "28px" } },
            "phone":      { "value": { "size": "24px" } }
          }
        }
      }
    },
    "innerContent": {
      "desktop": { "value": "<h1>Título ejemplo</h1>" }
    }
  }
}
```

Observaciones:
- `desktop.value` declara la línea base completa.
- `desktop.hover` solo declara el override (color).
- Los breakpoints intermedios solo declaran los overrides de tamaño; el resto (family, color) se hereda.
- `innerContent` no requiere overrides por breakpoint salvo que el texto cambie por dispositivo.

### 5.6 Variables globales (`$variable(...)$`)

Divi 5 soporta variables globales para referenciar colores y otros valores desde múltiples módulos. La sintaxis es:

```
$variable({"type":"color","value":{"name":"gcid-body-color","settings":{}}})$
```

En un JSON emitido dentro de un bloque, esa referencia (que es un string) aparece codificada:

```json
{
  "font": {
    "font": {
      "desktop": {
        "value": {
          "color": "$variable({\u0022type\u0022:\u0022color\u0022,\u0022value\u0022:{\u0022name\u0022:\u0022gcid-body-color\u0022,\u0022settings\u0022:{}}})$"
        }
      }
    }
  }
}
```

**Uso en v1:**
- La Skill no emite variables globales por defecto (valores hardcoded).
- **Si el HTML de entrada las trae**, la Skill debe respetarlas y avisar en `notes.md`.
- Los `global_colors` correspondientes deben venir declarados en el top-level del export (llave `global_colors`).

Tipos de `$variable(...)$`:
- `type: "color"` — referencia a un global color (por `name` = gcid-*).
- `type: "font"` — referencia a una variable de fuente global.
- `type: "spacing"` — referencia a una variable de spacing global.

---

## 6. Sub-propiedades comunes de `decoration`

Estas son las sub-propiedades más usadas dentro de `decoration`. Todas siguen el patrón `[breakpoint].[estado].{objeto de valores}`.

### 6.1 `background`

```json
{
  "background": {
    "desktop": {
      "value": {
        "color": "#3E0D61",
        "gradient": {
          "enabled": "on",
          "direction": "100deg",
          "stops": [
            { "position": 0, "color": "#3E0D61" },
            { "position": 100, "color": "#541690" }
          ],
          "length": "100%"
        },
        "image": {
          "url": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/hero-bg.jpg",
          "size": "cover",
          "position": "center center",
          "repeat": "no-repeat",
          "parallax": { "enabled": "off", "method": "on" }
        }
      }
    }
  }
}
```

Propiedades: `color`, `gradient`, `image`, `video`, `pattern`, `mask`. Todas opcionales, se declaran solo las que aplican.

### 6.2 `spacing` (padding y margin)

```json
{
  "spacing": {
    "desktop": {
      "value": {
        "padding": {
          "top": "80px", "right": "40px", "bottom": "80px", "left": "40px",
          "syncVertical": "off", "syncHorizontal": "off"
        },
        "margin": {
          "top": "0px", "right": "auto", "bottom": "0px", "left": "auto",
          "syncVertical": "off", "syncHorizontal": "off"
        }
      }
    }
  }
}
```

Unidades aceptadas: `px`, `em`, `rem`, `%`, `vw`, `vh`. Los valores vacíos (`""`) significan "sin override".
`syncVertical: "on"` sincroniza top y bottom, `syncHorizontal: "on"` sincroniza left y right.

### 6.3 `font` (para tipografía general del grupo)

```json
{
  "font": {
    "font": {
      "desktop": {
        "value": {
          "family": "Poppins",
          "weight": "600",
          "weightFineTune": "600",
          "variationSettings": { "WGHT": "" },
          "style": ["uppercase", "italic"],
          "size": "16px",
          "lineHeight": "1.4em",
          "letterSpacing": "2px",
          "color": "#2a2a2a",
          "textAlign": "left",
          "textWrap": "wrap",
          "capitalization": "none"
        }
      }
    },
    "textShadow": {
      "desktop": {
        "value": {
          "style": "preset3",
          "horizontal": "0em",
          "vertical": "0.1em",
          "blur": "0.2em",
          "color": "rgba(0,0,0,0.4)"
        }
      }
    }
  }
}
```

Nota el doble anidamiento `font.font`: el primero es la sub-propiedad del grupo, el segundo es el objeto de valores tipográficos.
`capitalization`: `none`, `uppercase`, `lowercase`, `capitalize`.
`textAlign`: `left`, `center`, `right`, `justify`.
`weightFineTune`: para fuentes variables (ej: "600" con FineTune "650").

### 6.4 `bodyFont` y `headingFont` (para contenido con múltiples tags)

Cuando un grupo contiene HTML mezclado (Text, Testimonial content, Blurb content, Post Content), se usan `bodyFont` (para `<p>`, `<span>`, `<li>`) y `headingFont` (con subllaves por nivel: `h1`, `h2`, `h3`, `h4`, `h5`, `h6`).

```json
{
  "bodyFont": {
    "body": { "font": { "desktop": { "value": { "family": "Inter", "size": "16px" } } } },
    "ul":   { "font": { "desktop": { "value": { "lineHeight": "2em" } } } },
    "ol":   { "font": { "desktop": { "value": { "lineHeight": "2em" } } } }
  },
  "headingFont": {
    "h1": { "font": { "desktop": { "value": { "family": "Plus Jakarta Sans", "weight": "700", "size": "48px" } } } },
    "h2": { "font": { "desktop": { "value": { "family": "Plus Jakarta Sans", "weight": "600", "size": "36px" } } } },
    "h3": { "font": { "desktop": { "value": { "family": "Plus Jakarta Sans", "weight": "600", "size": "24px" } } } }
  }
}
```

### 6.5 `border`

```json
{
  "border": {
    "desktop": {
      "value": {
        "styles": {
          "all": { "width": "1px", "style": "solid", "color": "rgba(0,0,0,0.12)" },
          "top": { "width": "2px" },
          "right": {},
          "bottom": {},
          "left": {}
        },
        "radius": {
          "sync": "on",
          "topLeft": "8px", "topRight": "8px", "bottomRight": "8px", "bottomLeft": "8px"
        }
      }
    }
  }
}
```

Reglas: `styles.all` aplica a los 4 lados; `styles.top/right/bottom/left` override individual. `radius.sync: "on"` significa que los 4 corners comparten el mismo radio; `"off"` significa radios independientes.

### 6.6 `boxShadow`

```json
{
  "boxShadow": {
    "desktop": {
      "value": {
        "style": "preset3",
        "horizontal": "0px",
        "vertical": "7px",
        "blur": "15px",
        "spread": "0px",
        "color": "rgba(0,0,0,0.07)",
        "position": "outer"
      }
    }
  }
}
```

Presets disponibles: `preset1` a `preset7`, o `none`. `position: "outer"` (default) o `"inner"`.

### 6.7 `animation`

```json
{
  "animation": {
    "desktop": {
      "value": {
        "style": "slide",
        "direction": "top",
        "duration": "600ms",
        "delay": "0ms",
        "intensity": { "slide": "10%", "zoom": "50%" },
        "startingOpacity": "0%",
        "speedCurve": "ease-in-out",
        "repeat": "once"
      }
    }
  }
}
```

Estilos: `none`, `fade`, `slide`, `zoom`, `flip`, `fold`, `roll`, `bounce`.
Direcciones: `top`, `right`, `bottom`, `left`, `center`.

### 6.8 `layout`

```json
{
  "layout": {
    "desktop": {
      "value": {
        "display": "flex",
        "flexDirection": "row",
        "flexWrap": "nowrap",
        "alignItems": "center",
        "alignContent": "flex-start",
        "justifyContent": "space-between",
        "columnGap": "16px",
        "rowGap": "16px",
        "gridColumnCount": "3"
      }
    }
  }
}
```

Valores de `display`: `block`, `flex`, `grid`, `inline`, `inline-block`, `none`.
`flexWrap`: `nowrap`, `wrap`, `wrap-reverse`.
`flexDirection`: `row`, `column`, `row-reverse`, `column-reverse`.
`justifyContent`: `start`, `end`, `center`, `space-between`, `space-around`, `space-evenly`.

Para módulos como Blog que soportan grid:

```json
{
  "layout": {
    "desktop":    { "value": { "display": "grid", "gridColumnCount": "3" } },
    "tabletWide": { "value": { "gridColumnCount": "2" } },
    "tablet":     { "value": { "gridColumnCount": "2" } },
    "phoneWide":  { "value": { "gridColumnCount": "1" } },
    "phone":      { "value": { "gridColumnCount": "1" } }
  }
}
```

### 6.9 `sizing`

```json
{
  "sizing": {
    "desktop": {
      "value": {
        "width": "100%",
        "maxWidth": "1200px",
        "minWidth": "",
        "height": "auto",
        "maxHeight": "",
        "minHeight": "",
        "alignment": "center",
        "flexType": "24_24"
      }
    }
  }
}
```

`alignment`: `left`, `center`, `right`.
`flexType`: sistema de 24 columnas (ver 4.3).

### 6.10 `dividers` (top/bottom de section)

```json
{
  "dividers": {
    "bottom": {
      "desktop": {
        "value": {
          "style": "ramp",
          "height": "21vw",
          "flip": ["horizontal"],
          "arrangement": "above",
          "color": "#ffffff"
        }
      }
    },
    "top": {
      "desktop": {
        "value": { "style": "curve", "height": "100px" }
      }
    }
  }
}
```

Estilos disponibles: `curve`, `slant`, `slant2`, `ramp`, `mountains`, `arrow`, `arrow2`, `waves`, `waves2`, `graph`, `triangle`, `triangle2`, `clouds`, `clouds2`, `asymmetric`, `asymmetric2`.

### 6.11 `attributes` (id/class custom en HTML)

```json
{
  "attributes": {
    "desktop": {
      "value": {
        "attributes": [
          {
            "id": "6a4575128fe0c",
            "name": "class",
            "value": "custom-hero-class",
            "adminLabel": "CSS Class",
            "targetElement": ""
          },
          {
            "id": "542a33e3-ed4e-40f0-bcee-198aa2d79316",
            "name": "id",
            "value": "hero-section",
            "adminLabel": "CSS ID"
          }
        ]
      }
    }
  }
}
```

Cada atributo requiere un `id` único (UUID o string aleatorio). Se usa para clases custom, IDs de anchor, atributos ARIA, `data-*` attributes.
`targetElement`: string opcional para dirigir el atributo a un sub-elemento específico (ej: `"logo"` en el módulo `menu`).

### 6.12 `filters`, `transform`, `transition`, `overflow`, `position`, `zIndex`

Estas sub-propiedades siguen el mismo patrón `[breakpoint].[estado].{objeto}`. Uso general:

- **`filters`**: `hueRotate`, `saturate`, `brightness`, `contrast`, `invert`, `sepia`, `opacity`, `blur`, `blendMode`.
- **`transform`**: `scale`, `translate`, `rotate`, `skew`, `origin`.
- **`transition`**: `duration`, `delay`, `speedCurve`.
- **`overflow`**: `x`, `y` (valores: `visible`, `hidden`, `scroll`, `auto`).
- **`position`**: `mode` (`default`, `relative`, `absolute`, `fixed`), `origin`, `offset`.
- **`zIndex`**: valor entero como string.

---

## 7. Manejo de imágenes

### 7.1 Estrategia v1: referencia por URL

Las imágenes se referencian por su URL final en WordPress. La ruta estándar de WordPress es:

```
https://<dominio>/wp-content/uploads/<AÑO>/<MES>/<nombre-archivo>.<ext>
```

Para Greenti dev:
```
https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/hero-bg.jpg
```

**Reglas:**
- La Skill NO embebe imágenes en base64 (la llave `images` del export se emite vacía `{}` o `[]`).
- Los nombres de archivo deben ser descriptivos, sin espacios ni tildes.
- La Skill genera un `assets-checklist.md` que lista todas las imágenes referenciadas con su URL final esperada.
- Antes de importar el JSON, el usuario debe subir manualmente las imágenes al Media Library de WordPress usando los mismos nombres.

### 7.2 Placeholder cuando no hay assets

Cuando el usuario no provee assets:

```json
{
  "image": {
    "innerContent": {
      "desktop": {
        "value": {
          "src": "https://placehold.co/1920x1080/cccccc/333333?text=PENDING+hero-bg",
          "id": 0,
          "width": "1920",
          "height": "1080",
          "alt": "PENDING - Reemplazar imagen desde Divi Builder"
        }
      }
    }
  },
  "module": {
    "meta": {
      "adminLabel": { "desktop": { "value": "IMAGEN PENDIENTE - hero-bg" } }
    }
  }
}
```

El `adminLabel` con prefijo `IMAGEN PENDIENTE` permite identificar visualmente en el árbol de Divi qué módulos necesitan reemplazo.

### 7.3 Datos requeridos en el módulo Image

```json
{
  "src": "URL final",
  "id": 0,
  "width": "ancho en px como string",
  "height": "alto en px como string",
  "alt": "texto alternativo descriptivo",
  "titleText": "opcional (title attribute)",
  "srcset": "opcional",
  "sizes": "opcional",
  "linkUrl": "opcional",
  "linkTarget": "off/on"
}
```

El `id` puede ser `0` cuando la imagen no está en el Media Library todavía (Divi la resolverá al importar si encuentra un match por URL). El `alt` es obligatorio (por a11y y SEO); si el HTML no lo trae, el `seo-auditor` lo genera o lo pide.

**Nota:** en algunos módulos (`menu.logo`, `slide.image`, `team-member.image`, etc.) las imágenes van dentro del grupo correspondiente con el mismo shape.

---

## 8. Catálogo de módulos (Sección A — completa)

### 8.0 Convención del catálogo

Para cada módulo se documenta:
- **Nombre interno** (el que va después de `wp:divi/` en el bloque).
- **Categoría de uso.**
- **Tipo:** Contenedor (con apertura + cierre) o Hoja (self-closing).
- **Grupos disponibles** (los que aplican).
- **Schema en JSON** con las propiedades clave y ejemplo mínimo funcional.
- **Notas** sobre casos especiales.

Los ejemplos usan los design tokens de referencia de Greenti:
- Colores: `#3E0D61` (dark purple), `#541690` (violet), `#1EDFAE` (green), `#F0FDFF` (light blue), `#1A0037` (dark footer).
- Fuentes: `Plus Jakarta Sans` (headings), `Inter` (body).

---

### 8.1 Módulos contenedores estructurales

#### `section` — Sección (contenedor nivel 1)

**Categoría:** Contenedor estructural raíz.
**Uso:** Bloque semántico principal de la página. Toda página tiene al menos una section.
**Tipo:** Contenedor.

**Ejemplo mínimo:**

```
<!-- wp:divi/section {"builderVersion":"5.8.1"} -->
  ...rows...
<!-- /wp:divi/section -->
```

**Ejemplo con background, padding y dividers:**

```json
{
  "module": {
    "meta": {
      "adminLabel": { "desktop": { "value": "Hero Section" } }
    },
    "advanced": {
      "type": { "desktop": { "value": "regular" } },
      "gutter": { "desktop": { "value": { "width": "3" } } },
      "dividers": {
        "bottom": {
          "desktop": { "value": { "style": "curve", "height": "100px", "color": "#ffffff" } }
        }
      }
    },
    "decoration": {
      "background": {
        "desktop": {
          "value": {
            "color": "#3E0D61",
            "image": {
              "url": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/hero-bg.jpg",
              "size": "cover",
              "position": "center center"
            }
          }
        }
      },
      "spacing": {
        "desktop": { "value": { "padding": { "top": "80px", "bottom": "80px" } } }
      },
      "layout": {
        "desktop": { "value": { "display": "block" } }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Propiedades soportadas:**
- `advanced.type`: `"regular"` (default) o `"specialty"`.
- `advanced.gutter.width`: `"1"`, `"2"`, `"3"`, `"4"` (ancho gutter entre columnas).
- `advanced.dividers.top` y `.bottom`: separadores visuales (ver 6.10).
- `decoration.background`, `.spacing`, `.border`, `.boxShadow`, `.animation`, `.layout`, `.sizing`, `.filters`, `.zIndex`.

**Para specialty section:**
```json
{
  "module": {
    "advanced": { "type": { "desktop": { "value": "specialty" } } }
  },
  "column1": { "decoration": {...} },
  "column2": { "decoration": {...} },
  "column3": { "decoration": {...} },
  "innerSizing": { "decoration": { "sizing": {...} } }
}
```

---

#### `row` — Fila

**Categoría:** Contenedor estructural.
**Uso:** Fila que contiene columnas.
**Tipo:** Contenedor.

**Ejemplo básico:**

```json
{
  "module": {
    "advanced": {
      "columnStructure": { "desktop": { "value": "4_4" } },
      "flexColumnStructure": { "desktop": { "value": "equal-columns_1" } }
    },
    "decoration": {
      "layout": { "desktop": { "value": { "flexWrap": "nowrap" } } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Ejemplo con 4 columnas iguales:**

```json
{
  "module": {
    "advanced": {
      "columnStructure": { "desktop": { "value": "1_4,1_4,1_4,1_4" } },
      "flexColumnStructure": { "desktop": { "value": "equal-columns_4" } }
    },
    "decoration": {
      "layout": {
        "desktop":    { "value": { "flexWrap": "nowrap" } },
        "tabletWide": { "value": { "flexWrap": "wrap" } },
        "tablet":     { "value": { "flexWrap": "wrap" } }
      },
      "sizing": {
        "desktop": { "value": { "maxWidth": "80%" } }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Ejemplo con configuración por columna hija:**

```json
{
  "module": {...},
  "columns": {
    "column-1": { "spacing": { "desktop": { "value": { "padding": { "top": "40px" } } } } },
    "column-2": { "spacing": { "desktop": { "value": { "padding": { "top": "40px" } } } } }
  }
}
```

**Ver sección 4.2** para lista completa de valores válidos de `columnStructure` y `flexColumnStructure`.

---

#### `row-inner` — Fila interna (specialty)

**Categoría:** Contenedor estructural.
**Uso:** Row dentro de una column de specialty section.
**Tipo:** Contenedor.

Mismo schema que `row` pero anidado dentro de una `column` de specialty section. Contiene `column-inner`.

---

#### `column` — Columna

**Categoría:** Contenedor estructural.
**Uso:** Columna dentro de un row.
**Tipo:** Contenedor.

**Ejemplo mínimo:**

```json
{
  "module": {
    "advanced": { "type": { "desktop": { "value": "1_2" } } },
    "decoration": {
      "sizing": { "desktop": { "value": { "flexType": "12_24" } } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Ejemplo con responsive flexType:**

```json
{
  "module": {
    "advanced": { "type": { "desktop": { "value": "1_4" } } },
    "decoration": {
      "sizing": {
        "desktop":    { "value": { "flexType": "6_24" } },
        "tabletWide": { "value": { "flexType": "12_24" } },
        "tablet":     { "value": { "flexType": "12_24" } },
        "phoneWide":  { "value": { "flexType": "24_24" } },
        "phone":      { "value": { "flexType": "24_24" } }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Ejemplo con layout flex interno para agrupar módulos:**

```json
{
  "module": {
    "advanced": { "type": { "desktop": { "value": "4_4" } } },
    "decoration": {
      "sizing": { "desktop": { "value": { "flexType": "24_24" } } },
      "layout": {
        "desktop": {
          "value": {
            "flexDirection": "row",
            "justifyContent": "space-around",
            "alignItems": "center"
          }
        },
        "phone": { "value": { "flexWrap": "wrap" } }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

Propiedades comunes: background, spacing, border, boxShadow, layout, sizing, filters.

---

#### `column-inner` — Columna interna (specialty)

**Categoría:** Contenedor estructural.
**Uso:** Column dentro de row-inner.
**Tipo:** Contenedor.

Mismo schema que `column` pero anidado. Adicionalmente puede llevar `advanced.savedSpecialtyColumnType` que indica el ancho de la columna padre en la specialty (`1_2`, `1_3`, etc.).

---

#### `group` — Contenedor de agrupación (nuevo en Divi 5)

**Categoría:** Contenedor de agrupación.
**Uso:** Wrapper genérico dentro de una columna para agrupar varios módulos con layout propio (flex, grid). Actúa como un `<div>` con configuración.
**Tipo:** Contenedor.

**Ejemplo mínimo:**

```json
{
  "builderVersion": "5.8.1"
}
```

**Ejemplo con layout flex:**

```json
{
  "module": {
    "decoration": {
      "layout": {
        "desktop": {
          "value": {
            "display": "flex",
            "flexDirection": "column",
            "alignItems": "start",
            "columnGap": "16px",
            "rowGap": "16px"
          }
        }
      },
      "spacing": {
        "desktop": { "value": { "padding": { "top": "24px", "right": "24px", "bottom": "24px", "left": "24px" } } }
      },
      "background": {
        "desktop": { "value": { "color": "#F0FDFF" } }
      },
      "border": {
        "desktop": { "value": { "radius": { "sync": "on", "topLeft": "12px" } } }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Uso típico:** cards, agrupaciones dentro de dropdowns, wrappers de contenido con background/border propios.

---

#### `group-carousel` — Contenedor de agrupación tipo carrusel (nuevo en Divi 5)

**Categoría:** Contenedor de agrupación.
**Uso:** Wrapper que muestra sus hijos en carrusel (con navegación, autoplay opcional).
**Tipo:** Contenedor.

**Ejemplo mínimo:**

```json
{
  "module": {
    "advanced": {
      "carousel": {
        "desktop": {
          "value": {
            "autoplay": "off",
            "loop": "on",
            "slidesPerView": "3",
            "spaceBetween": "16"
          }
        },
        "tablet":  { "value": { "slidesPerView": "2" } },
        "phone":   { "value": { "slidesPerView": "1" } }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Uso típico:** carruseles de logos, carruseles de cards, sliders horizontales de contenido custom.

---

#### `dropdown` — Contenedor dropdown/modal (nuevo en Divi 5)

**Categoría:** Contenedor de agrupación (interactivo).
**Uso:** Wrapper que oculta/muestra su contenido con interacción del usuario (dropdown, modal, popover).
**Tipo:** Contenedor. Contiene un `group` que a su vez contiene módulos hoja.

**Ejemplo:**

```json
{
  "module": {
    "meta": {
      "forceVisible": { "desktop": { "value": "whileInBuilder" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Estructura HTML resultante:**

```
<!-- wp:divi/dropdown {...} -->
  <!-- wp:divi/group {...} -->
    <!-- wp:divi/heading {...} /-->
    <!-- wp:divi/text {...} /-->
    <!-- wp:divi/button {...} /-->
  <!-- /wp:divi/group -->
<!-- /wp:divi/dropdown -->
```

`forceVisible: "whileInBuilder"` hace que el contenido sea visible mientras se edita en Divi Builder (facilita la edición sin tener que "abrir" el dropdown).

**Nota:** por defecto el dropdown se activa con click. Configuraciones adicionales de trigger (hover, scroll) suelen requerir más advanced properties específicas.

---

### 8.2 Módulos de texto y titulares

#### `heading` — Encabezado (nuevo en Divi 5, dedicado)

**Categoría:** Contenido / Texto.
**Uso:** Título dedicado (H1 a H6). En Divi 5 los headings tienen módulo propio, separado de `text`. Recomendado usar `heading` para títulos de sección aislados, y `text` cuando hay título + párrafo mezclado.
**Tipo:** Hoja (self-closing).

**Ejemplo mínimo:**

```json
{
  "title": {
    "innerContent": { "desktop": { "value": "Tu Título Va Aquí" } }
  },
  "builderVersion": "5.8.1"
}
```

**Ejemplo con configuración de fuente:**

```json
{
  "module": {
    "meta": { "adminLabel": { "desktop": { "value": "Título Hero" } } }
  },
  "title": {
    "innerContent": {
      "desktop": { "value": "Impulsa tu presencia digital" }
    },
    "decoration": {
      "font": {
        "font": {
          "desktop": {
            "value": {
              "headingLevel": "h1",
              "family": "Plus Jakarta Sans",
              "weight": "700",
              "size": "48px",
              "lineHeight": "1.2em",
              "color": "#3E0D61"
            }
          },
          "phone": { "value": { "size": "32px" } }
        }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`.
`title.decoration.font.font.desktop.value.headingLevel`: `h1`, `h2`, `h3`, `h4`, `h5`, `h6`.

---

#### `text` — Texto libre

**Categoría:** Contenido.
**Uso:** Bloque de texto libre con soporte para múltiples niveles de heading y párrafos. Es el módulo más usado. Ideal cuando hay HTML mezclado (título + descripción + lista).
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "meta": { "adminLabel": { "desktop": { "value": "Título de sección" } } },
    "advanced": {
      "text": { "text": { "desktop": { "value": { "orientation": "center" } } } }
    },
    "decoration": {
      "sizing": { "desktop": { "value": { "maxWidth": "550px", "alignment": "center" } } },
      "spacing": { "desktop": { "value": { "margin": { "bottom": "10px" } } } }
    }
  },
  "content": {
    "decoration": {
      "bodyFont": {
        "body": { "font": { "desktop": { "value": { "family": "Inter", "size": "16px", "lineHeight": "1.8em" } } } }
      },
      "headingFont": {
        "h2": { "font": { "desktop": { "value": { "family": "Plus Jakarta Sans", "weight": "600", "size": "36px", "lineHeight": "1.4em" } } } }
      }
    },
    "innerContent": {
      "desktop": {
        "value": "<h2>Nuestros Servicios</h2>\n<p>Lorem ipsum dolor sit amet.</p>"
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `content`.
`content.innerContent.desktop.value` contiene HTML con `<h1>...<h6>`, `<p>`, `<ul>`, `<ol>`, `<li>`, `<a>`, `<strong>`, `<em>`, `<blockquote>`.
`orientation`: `left`, `center`, `right`, `justify`.

---

#### `cta` — Call To Action

**Categoría:** Contenido / Conversión.
**Uso:** Bloque con título + descripción + botón con acción clara (hero, sección promocional). Usar `cta` cuando semánticamente el bloque completo constituye un llamado a la acción; usar `heading` + `text` + `button` separados cuando son elementos independientes.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "meta": { "adminLabel": { "desktop": { "value": "Hero CTA" } } },
    "advanced": {
      "text": { "text": { "desktop": { "value": { "orientation": "left" } } } }
    }
  },
  "title": {
    "innerContent": { "desktop": { "value": "Impulsa tu presencia digital" } },
    "decoration": {
      "font": {
        "font": {
          "desktop": {
            "value": { "headingLevel": "h1", "family": "Plus Jakarta Sans", "weight": "700", "size": "48px", "lineHeight": "1.2em" }
          }
        }
      }
    }
  },
  "content": {
    "decoration": {
      "bodyFont": {
        "body": { "font": { "desktop": { "value": { "family": "Inter", "size": "18px", "lineHeight": "1.6em" } } } }
      }
    },
    "innerContent": { "desktop": { "value": "Ayudamos a marcas de LATAM a crecer con VTEX." } }
  },
  "button": {
    "innerContent": {
      "desktop": {
        "value": { "linkUrl": "/contacto", "text": "Empezar ahora", "id": 0 }
      }
    },
    "decoration": {
      "button": {
        "desktop": { "value": { "enable": "on" } }
      },
      "font": {
        "font": { "desktop": { "value": { "size": "14px", "color": "#ffffff", "family": "Inter", "weight": "600", "style": ["uppercase"] } } }
      },
      "background": { "desktop": { "value": { "color": "#1EDFAE" } } },
      "border": {
        "desktop": { "value": { "styles": { "all": { "width": "0px" } }, "radius": { "sync": "on", "topLeft": "8px" } } }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `content`, `button`.
`button.decoration.button.desktop.value.enable`: `"on"` habilita botón, `"off"` lo oculta.

---

#### `button` — Botón

**Categoría:** Contenido / Conversión.
**Uso:** Botón standalone. Para CTA compuestos, usar módulo `cta`.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "advanced": { "alignment": { "desktop": { "value": "center" } } }
  },
  "button": {
    "innerContent": {
      "desktop": {
        "value": { "linkUrl": "/contacto", "text": "Contáctanos", "id": 0, "linkTarget": "on" }
      }
    },
    "decoration": {
      "button": { "desktop": { "value": { "enable": "on" } } },
      "font": {
        "font": { "desktop": { "value": { "size": "14px", "color": "#ffffff", "family": "Inter", "weight": "600", "style": ["uppercase"] } } }
      },
      "background": { "desktop": { "value": { "color": "#541690" } } },
      "border": {
        "desktop": {
          "value": {
            "styles": { "all": { "width": "0px" } },
            "radius": { "sync": "on", "topLeft": "8px", "topRight": "8px", "bottomRight": "8px", "bottomLeft": "8px" }
          }
        }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

`linkTarget`: `"on"` (abre en nueva pestaña) o `"off"`.
`alignment`: `left`, `center`, `right`.

---

### 8.3 Módulos de contenido con media

#### `image` — Imagen

**Categoría:** Contenido / Media.
**Uso:** Imagen individual (logotipo, ilustración, foto, ícono).
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "advanced": {
      "spacing": { "desktop": { "value": { "margin": { "top": "0px", "bottom": "0px" } } } }
    }
  },
  "image": {
    "innerContent": {
      "desktop": {
        "value": {
          "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/logo-cliente.png",
          "id": 0,
          "width": "200",
          "height": "80",
          "alt": "Logo de Cliente",
          "linkUrl": "",
          "linkTarget": "off",
          "showBottomSpace": "on",
          "alignment": "center"
        }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

---

#### `icon` — Ícono (nuevo en Divi 5)

**Categoría:** Contenido / Media.
**Uso:** Ícono standalone (Divi icon set o Font Awesome). Antes de Divi 5 se lograba con `blurb` o Code Module.
**Tipo:** Hoja (self-closing).

**Ejemplo con ícono de Divi:**

```json
{
  "icon": {
    "innerContent": {
      "desktop": {
        "value": {
          "unicode": "&#xe0fd;",
          "type": "divi",
          "weight": "400"
        }
      }
    },
    "advanced": {
      "size":  { "desktop": { "value": "48px" } },
      "color": { "desktop": { "value": "#1EDFAE" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Ejemplo con ícono Font Awesome:**

```json
{
  "icon": {
    "innerContent": {
      "desktop": {
        "value": {
          "unicode": "&#xf240;",
          "type": "fa",
          "weight": "900"
        }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `icon`.
`icon.innerContent.desktop.value.type`: `divi` (icon set nativo) o `fa` (Font Awesome).
`weight`: `100` (light), `300`, `400` (regular), `900` (solid/bold), depende del set.
`unicode`: código HTML entity. En JSON emitido se codifica: `&#xe0fd;` → `\u0026#xe0fd;`.

---

#### `blurb` — Blurb (ícono/imagen + título + descripción)

**Categoría:** Contenido.
**Uso:** Feature-card: ícono/imagen + título + descripción. Muy usado en grids de features/servicios.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "advanced": {
      "text": { "text": { "desktop": { "value": { "orientation": "center" } } } }
    },
    "decoration": {
      "spacing": { "desktop": { "value": { "margin": { "bottom": "0px" } } } }
    }
  },
  "imageIcon": {
    "innerContent": {
      "desktop": {
        "value": {
          "useIcon": "on",
          "icon": { "unicode": "&#xe0fd;", "type": "divi", "weight": "400" },
          "src": "",
          "id": 0,
          "alt": ""
        }
      }
    },
    "advanced": {
      "color":     { "desktop": { "value": "#1EDFAE" } },
      "placement": { "desktop": { "value": "top" } }
    }
  },
  "title": {
    "innerContent": {
      "desktop": { "value": { "text": "UX/UI a medida", "id": 0 } }
    },
    "decoration": {
      "font": {
        "font": {
          "desktop": {
            "value": { "headingLevel": "h3", "family": "Plus Jakarta Sans", "weight": "600", "size": "24px" }
          }
        }
      }
    }
  },
  "content": {
    "decoration": {
      "bodyFont": {
        "body": { "font": { "desktop": { "value": { "family": "Inter", "size": "16px", "lineHeight": "1.6em" } } } }
      }
    },
    "innerContent": {
      "desktop": { "value": "<p>Diseñamos experiencias que convierten.</p>" }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `imageIcon`, `title`, `content`.
`imageIcon.innerContent.useIcon`: `"on"` (usa icono) o `"off"` (usa imagen; entonces requiere `src`).
`placement`: `top` (default), `left`, `right`.

---

#### `video` — Video (nuevo en Divi 5 como módulo dedicado)

**Categoría:** Contenido / Media.
**Uso:** Video individual (YouTube, Vimeo, MP4 hosted).
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "meta": { "adminLabel": { "desktop": { "value": "Video Testimonial" } } },
    "decoration": {
      "border": {
        "desktop": {
          "value": { "radius": { "sync": "on", "topLeft": "30px", "topRight": "30px", "bottomLeft": "30px", "bottomRight": "30px" } }
        }
      },
      "boxShadow": {
        "desktop": {
          "value": {
            "style": "preset3",
            "horizontal": "0px", "vertical": "12px", "blur": "18px", "spread": "-6px",
            "color": "rgba(0,0,0,0.3)"
          }
        }
      },
      "animation": {
        "desktop": { "value": { "style": "zoom" } }
      }
    }
  },
  "video": {
    "innerContent": {
      "desktop": {
        "value": { "src": "https://www.youtube.com/watch?v=FkQuawiGWUw" }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `video`.
`video.innerContent.desktop.value.src`: URL de YouTube, Vimeo o path a MP4 self-hosted.
También puede incluir `overlay` con imagen previa (thumbnail) opcional.

---

#### `audio` — Audio player

**Categoría:** Contenido / Media.
**Uso:** Reproductor de audio (podcast, música, sample).
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "artistName": {
    "innerContent": { "desktop": { "value": "Nombre Del Artista" } }
  },
  "title": {
    "innerContent": { "desktop": { "value": "Título del track" } }
  },
  "audio": {
    "innerContent": {
      "desktop": { "value": { "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/track.mp3" } }
    }
  },
  "coverArt": {
    "innerContent": {
      "desktop": { "value": { "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/cover.jpg" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `artistName`, `audio`, `coverArt`.

---

#### `gallery` — Galería de imágenes

**Categoría:** Contenido / Media.
**Uso:** Grid o slider de múltiples imágenes.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "advanced": {
      "layout": { "desktop": { "value": "grid" } },
      "orientation": { "desktop": { "value": "landscape" } }
    }
  },
  "gallery": {
    "innerContent": {
      "desktop": {
        "value": {
          "images": [
            { "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/img-1.jpg", "id": 0, "alt": "Imagen 1" },
            { "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/img-2.jpg", "id": 0, "alt": "Imagen 2" },
            { "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/img-3.jpg", "id": 0, "alt": "Imagen 3" }
          ],
          "columns": 3,
          "showCaption": "off",
          "showTitle": "off"
        }
      }
    }
  },
  "pagination": {
    "advanced": {
      "enable":     { "desktop": { "value": "on" } },
      "postsNumber":{ "desktop": { "value": "9" } }
    }
  },
  "overlay": {
    "advanced": {
      "background": { "desktop": { "value": { "color": "rgba(62,13,97,0.7)" } } },
      "icon":       { "desktop": { "value": { "color": "#ffffff" } } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `gallery`, `title`, `caption`, `pagination`, `overlay`.
`layout`: `grid` o `slider`.
`orientation`: `landscape`, `portrait`, `square`.

---

#### `before-after-image` — Comparador antes/después (nuevo en Divi 5)

**Categoría:** Contenido / Media interactivo.
**Uso:** Comparador con slider entre dos imágenes (antes/después). Muy usado en portfolios de diseño, transformaciones.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "beforeImage": {
    "innerContent": {
      "desktop": {
        "value": {
          "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/antes.jpg",
          "id": 0,
          "alt": "Antes de la transformación"
        }
      }
    }
  },
  "afterImage": {
    "innerContent": {
      "desktop": {
        "value": {
          "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/despues.jpg",
          "id": 0,
          "alt": "Después de la transformación"
        }
      }
    }
  },
  "beforeLabel": {
    "innerContent": { "desktop": { "value": "Antes" } }
  },
  "afterLabel": {
    "innerContent": { "desktop": { "value": "Después" } }
  },
  "module": {
    "advanced": {
      "orientation": { "desktop": { "value": "horizontal" } },
      "startPosition": { "desktop": { "value": "50%" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `beforeImage`, `afterImage`, `beforeLabel`, `afterLabel`.
`orientation`: `horizontal` (slider horizontal) o `vertical`.

---

### 8.4 Módulos interactivos (contenedor + item)

#### `accordion` — Acordeón (contenedor)

**Categoría:** Contenido interactivo.
**Uso:** FAQs, secciones colapsables verticales.
**Tipo:** Contenedor. Contiene `accordion-item`s.

**Ejemplo:**

```json
{
  "module": {
    "decoration": {
      "border": { "desktop": { "value": { "styles": { "all": { "color": "#ffffff" } } } } },
      "layout": { "desktop": { "value": { "flexDirection": "column" } } }
    }
  },
  "title": {
    "decoration": {
      "font": { "font": { "desktop": { "value": { "weightFineTune": "600" } } } }
    }
  },
  "closedToggleIcon": {
    "decoration": {
      "icon": { "desktop": { "value": { "color": "#3E0D61" } } }
    }
  },
  "openToggle": {
    "decoration": {
      "background": { "desktop": { "value": { "color": "#e8edff" } } }
    }
  },
  "closedToggle": {
    "decoration": {
      "background": { "desktop": { "value": { "color": "#ffffff" } } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `content`, `openToggle`, `closedToggle`, `openToggleIcon`, `closedToggleIcon`.

---

#### `accordion-item` — Ítem de acordeón

**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "advanced": {
      "open": { "desktop": { "value": "on" } }
    },
    "decoration": {
      "layout": { "desktop": { "value": { "justifyContent": "start", "alignContent": "flex-start" } } }
    }
  },
  "title": {
    "innerContent": { "desktop": { "value": "¿Cuánto tarda un proyecto VTEX?" } }
  },
  "content": {
    "innerContent": {
      "desktop": { "value": "<p>Entre 6 y 12 semanas dependiendo del alcance.</p>" }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `content`.
`module.advanced.open.desktop.value`: `"on"` (item abierto al cargar) o `"off"` (cerrado).

---

#### `toggle` — Toggle simple

**Categoría:** Contenido interactivo.
**Uso:** Toggle único (un item colapsable standalone). Para múltiples usar `accordion`.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "title": {
    "innerContent": { "desktop": { "value": "Ver más detalles" } }
  },
  "content": {
    "innerContent": {
      "desktop": { "value": "<p>Contenido colapsable.</p>" }
    }
  },
  "module": {
    "advanced": {
      "open": { "desktop": { "value": "off" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `content`, `openToggle`, `closedToggle`.

---

#### `tabs` — Pestañas (contenedor)

**Categoría:** Contenido interactivo.
**Uso:** Pestañas horizontales de contenido intercambiable.
**Tipo:** Contenedor. Contiene `tab`s.

**Ejemplo:**

```json
{
  "module": {
    "meta": { "adminLabel": { "desktop": { "value": "Tabs de Servicios" } } },
    "decoration": {
      "border": {
        "desktop": {
          "value": { "radius": { "sync": "on", "topLeft": "20px", "topRight": "20px", "bottomLeft": "20px", "bottomRight": "20px" } }
        }
      }
    }
  },
  "tab": {
    "decoration": {
      "font": {
        "font": { "desktop": { "value": { "family": "Inter", "size": "16px" } } }
      }
    }
  },
  "activeTab": {
    "decoration": {
      "font": {
        "font": { "desktop": { "value": { "color": "#1EDFAE" } } }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `tab` (estilo de tabs inactivos), `activeTab` (estilo del tab activo).

---

#### `tab` — Ítem de tab

**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "title": {
    "innerContent": { "desktop": { "value": "Tab 1" } }
  },
  "content": {
    "innerContent": {
      "desktop": { "value": "<p>Contenido del primer tab.</p>" }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `content`.

---

#### `slider` — Slider genérico (contenedor)

**Categoría:** Contenido interactivo / Media.
**Uso:** Carousel de slides con contenido rico (título, descripción, imagen de fondo, botón).
**Tipo:** Contenedor. Contiene `slide`s.

**Ejemplo:**

```json
{
  "module": {
    "advanced": {
      "showControls": { "desktop": { "value": "on" } },
      "showArrows":   { "desktop": { "value": "on" } },
      "autoplay":     { "desktop": { "value": "off" } }
    }
  },
  "navigationArrows": {
    "decoration": {
      "icon": { "desktop": { "value": { "color": "#ffffff" } } }
    }
  },
  "navigationDots": {
    "decoration": {
      "background": { "desktop": { "value": { "color": "rgba(255,255,255,0.5)" } } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `navigationArrows`, `navigationDots`.

---

#### `slide` — Ítem de slider

**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "advanced": {
      "text": { "text": { "desktop": { "value": { "orientation": "center" } } } }
    },
    "decoration": {
      "background": {
        "desktop": {
          "value": {
            "image": {
              "url": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/slide-bg.jpg",
              "size": "cover", "position": "center center"
            }
          }
        }
      }
    }
  },
  "title": {
    "innerContent": { "desktop": { "value": "Título del Slide" } }
  },
  "content": {
    "innerContent": {
      "desktop": { "value": "<p>Descripción del slide.</p>" }
    }
  },
  "image": {
    "innerContent": {
      "desktop": {
        "value": {
          "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/slide-fg.png",
          "alt": "Slide foreground"
        }
      }
    }
  },
  "button": {
    "innerContent": {
      "desktop": {
        "value": { "linkUrl": "/mas-info", "text": "Ver más", "id": 0 }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `content`, `image`, `button`.

---

### 8.5 Módulos de contadores y estadísticas

#### `number-counter` — Contador numérico

**Categoría:** Widget / Estadística.
**Uso:** Estadística prominente con número grande + label (ej: "500+ clientes atendidos").
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "title": {
    "innerContent": { "desktop": { "value": "Clientes satisfechos" } },
    "decoration": {
      "font": { "font": { "desktop": { "value": { "family": "Inter", "size": "16px", "color": "#3E0D61" } } } }
    }
  },
  "number": {
    "innerContent": { "desktop": { "value": "500" } },
    "advanced": {
      "enablePercentSign": { "desktop": { "value": "off" } }
    },
    "decoration": {
      "font": {
        "font": {
          "desktop": {
            "value": { "family": "Plus Jakarta Sans", "weight": "700", "size": "80px", "color": "#1EDFAE", "lineHeight": "1em" }
          }
        }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `number`.
`enablePercentSign`: `"on"` añade `%` al número.

---

#### `circle-counter` — Contador circular

**Categoría:** Widget / Estadística.
**Uso:** Progreso circular animado con porcentaje.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "title": {
    "innerContent": { "desktop": { "value": "Retención de clientes" } }
  },
  "number": {
    "innerContent": { "desktop": { "value": "92" } }
  },
  "module": {
    "advanced": {
      "circleColor": { "desktop": { "value": "#1EDFAE" } },
      "circleColorAlpha": { "desktop": { "value": "1" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `number`.

---

#### `counters` — Barras de contador (contenedor)

**Categoría:** Widget / Estadística.
**Uso:** Grupo de barras de progreso animadas.
**Tipo:** Contenedor. Contiene `counter`s.

**Ejemplo:**

```json
{
  "module": {
    "meta": { "adminLabel": { "desktop": { "value": "Skills" } } }
  },
  "barProgress": {
    "advanced": {
      "usePercentages": { "desktop": { "value": "on" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `barProgress`.
`usePercentages`: `"on"` muestra el porcentaje en cada barra.

---

#### `counter` — Ítem de barra (progreso individual)

**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "title": {
    "innerContent": { "desktop": { "value": "VTEX Development" } }
  },
  "barProgress": {
    "innerContent": { "desktop": { "value": "95" } }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `barProgress`.
`barProgress.innerContent`: valor 0-100 (porcentaje).

---

#### `countdown-timer` — Cronómetro regresivo

**Categoría:** Widget.
**Uso:** Cuenta regresiva a fecha específica (lanzamientos, ofertas).
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "title": {
    "innerContent": { "desktop": { "value": "Lanzamiento en:" } }
  },
  "module": {
    "advanced": {
      "endDate": { "desktop": { "value": "2026-12-31 00:00" } }
    }
  },
  "numbers": {
    "decoration": {
      "font": {
        "font": {
          "desktop": { "value": { "family": "Plus Jakarta Sans", "weight": "700", "size": "72px", "color": "#1EDFAE" } }
        }
      }
    }
  },
  "separator": {
    "decoration": {
      "font": {
        "font": { "desktop": { "value": { "color": "#3E0D61" } } }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `numbers`, `separator`.
`endDate`: formato `YYYY-MM-DD HH:MM`.

---

### 8.6 Módulos de línea de tiempo (nuevo en Divi 5)

#### `timeline` — Timeline (contenedor)

**Categoría:** Contenido.
**Uso:** Línea de tiempo con eventos (historia de la empresa, roadmap, hitos).
**Tipo:** Contenedor. Contiene `timeline-item`s.

**Ejemplo con configuración completa:**

```json
{
  "module": {
    "advanced": {
      "timeline": {
        "desktop": {
          "value": {
            "direction": "vertical",
            "position": "alternating",
            "startFrom": "left"
          }
        }
      }
    }
  },
  "track": {
    "decoration": {
      "background": { "desktop": { "value": { "color": "#fcf9f9" } } }
    }
  },
  "item": {
    "decoration": {
      "background": { "desktop": { "value": { "color": "#d5c6c6" } } }
    }
  },
  "itemEven": {
    "decoration": {
      "background": { "desktop": { "value": { "color": "#9d2e2e" } } }
    }
  },
  "children": {
    "date": {
      "advanced": {
        "displayOnSpacer": { "desktop": { "value": "on" } }
      }
    }
  },
  "connector": {
    "decoration": {
      "background": { "desktop": { "value": { "color": "#00ff56" } } }
    }
  },
  "marker": {
    "advanced": {
      "desktop": { "value": { "position": "center" } }
    },
    "decoration": {
      "icon": { "desktop": { "value": { "color": "#ff0000" } } },
      "background": { "desktop": { "value": { "color": "#ff0000" } } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `track` (línea de tiempo), `item` (estilo items impares), `itemEven` (estilo items pares), `connector` (conector entre items), `marker` (marcadores/nodos), `children` (config compartida hijos incluyendo `date`).

`direction`: `vertical` (default) o `horizontal`.
`position`: `alternating` (izquierda/derecha alternado), `left`, `right`.
`startFrom`: `left` o `right`.

---

#### `timeline-item` — Ítem de timeline

**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "date": {
    "innerContent": { "desktop": { "value": "Enero 2026" } }
  },
  "title": {
    "innerContent": { "desktop": { "value": "Fundación de Greenti" } }
  },
  "content": {
    "innerContent": {
      "desktop": { "value": "<p>Inicio de operaciones en Chile con el primer cliente VTEX.</p>" }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `date`, `title`, `content`.

---

### 8.7 Módulos de listas y grupos de íconos

#### `icon-list` — Lista con íconos (contenedor)

**Categoría:** Contenido.
**Uso:** Lista de items con íconos (típico para checklists de features, listas de servicios en footers).
**Tipo:** Contenedor. Contiene `icon-list-item`s.

**Ejemplo:**

```json
{
  "module": {
    "meta": { "adminLabel": { "desktop": { "value": "Icon List" } } }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`.

---

#### `icon-list-item` — Ítem de lista con ícono

**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "content": {
    "innerContent": { "desktop": { "value": "Diseño UX/UI a medida" } }
  },
  "icon": {
    "innerContent": {
      "desktop": {
        "value": {
          "unicode": "&#x21;",
          "type": "divi",
          "weight": "400",
          "target": "off"
        }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `content`, `icon`.
`icon.innerContent.desktop.value.target`: `"off"` (default) o `"on"` (icono con link).

---

### 8.8 Módulos de precios

#### `pricing-tables` — Tablas de precios (contenedor)

**Categoría:** Contenido / Conversión.
**Tipo:** Contenedor. Contiene `pricing-table`s.

**Ejemplo:**

```json
{
  "module": {
    "decoration": {
      "spacing": { "desktop": { "value": { "padding": { "top": "30px", "bottom": "30px" } } } }
    }
  },
  "title": { "decoration": { "font": { "font": { "desktop": { "value": { "family": "Plus Jakarta Sans", "weight": "700" } } } } } },
  "price":  { "decoration": { "font": { "font": { "desktop": { "value": { "size": "48px", "color": "#1EDFAE" } } } } } },
  "currencyFrequency": { "decoration": { "font": { "font": { "desktop": { "value": { "family": "Inter" } } } } } },
  "content": {
    "advanced": { "showBullet": { "desktop": { "value": "on" } } }
  },
  "button": {
    "decoration": {
      "button": { "desktop": { "value": { "enable": "on" } } },
      "background": { "desktop": { "value": { "color": "#541690" } } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `price`, `currencyFrequency`, `content`, `button`.

---

#### `pricing-table` — Tabla de precio individual

**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "title": { "innerContent": { "desktop": { "value": "Pro" } } },
  "price": { "innerContent": { "desktop": { "value": "199" } } },
  "currencyFrequency": {
    "innerContent": {
      "desktop": { "value": { "currency": "USD", "per": "mes", "id": 0 } }
    }
  },
  "content": {
    "innerContent": {
      "desktop": { "value": "Hasta 10 usuarios\nSoporte prioritario\nIntegraciones ilimitadas" }
    }
  },
  "button": {
    "innerContent": {
      "desktop": { "value": { "linkUrl": "/checkout", "text": "Elegir Pro", "id": 0 } }
    }
  },
  "builderVersion": "5.8.1"
}
```

`content.innerContent.value` usa saltos de línea `\n` para separar features.

---

### 8.9 Módulos sociales

#### `social-media-follow` — Contenedor de redes sociales

**Categoría:** Widget.
**Tipo:** Contenedor. Contiene `social-media-follow-network`s.

**Ejemplo:**

```json
{
  "module": {
    "advanced": {
      "text": { "text": { "desktop": { "value": { "orientation": "left" } } } }
    }
  },
  "builderVersion": "5.8.1"
}
```

---

#### `social-media-follow-network` — Icono de red social individual

**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "decoration": {
      "background": { "desktop": { "value": { "color": "#3b5998" } } },
      "spacing": { "desktop": { "value": { "padding": { "top": "6px", "right": "6px", "bottom": "6px", "left": "6px" } } } }
    }
  },
  "socialNetwork": {
    "innerContent": {
      "desktop": { "value": { "title": "facebook", "label": "Facebook", "id": 0 } }
    },
    "advanced": {
      "followButton": { "desktop": { "value": "off" } }
    }
  },
  "button": {
    "innerContent": {
      "desktop": {
        "value": { "linkTarget": "on", "linkUrl": "https://facebook.com/greenti", "id": 0 }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

`socialNetwork.innerContent.desktop.value.title` acepta: `facebook`, `twitter`, `instagram`, `linkedin`, `youtube`, `tumblr`, `pinterest`, `flickr`, `vimeo`, `rss`, `google_plus`, `skype`, `snapchat`, `dribbble`, `soundcloud`, `tiktok`, `whatsapp`, `telegram`, `discord`.

---

#### `instagram-feed` — Feed de Instagram (nuevo en Divi 5)

**Categoría:** Widget dinámico.
**Uso:** Grid de posts recientes de Instagram embebido.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "feed": {
    "innerContent": {
      "desktop": {
        "value": {
          "accountId": "greenti_oficial",
          "accessToken": ""
        }
      }
    },
    "advanced": {
      "count":   { "desktop": { "value": "6" } },
      "columns": { "desktop": { "value": "3" } }
    }
  },
  "followButton": {
    "innerContent": {
      "desktop": { "value": { "text": "Síguenos En Instagram" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `feed`, `followButton`.
Requiere configuración de Instagram API en el sitio (se hace desde el admin de Divi).

---

### 8.10 Módulos de navegación y búsqueda

#### `menu` — Menú de navegación

**Categoría:** Navegación.
**Uso:** Menú principal del sitio, típicamente en header. Vinculado a un menú registrado en WordPress.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "decoration": {
      "attributes": {
        "attributes": [
          {
            "id": "frmtm643cq",
            "name": "title",
            "value": "logo-greenti",
            "adminLabel": "Image Title",
            "targetElement": "logo"
          }
        ]
      },
      "layout": {
        "desktop": { "value": { "justifyContent": "space-around" } }
      }
    }
  },
  "logo": {
    "innerContent": {
      "desktop": {
        "value": {
          "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/logo-greenti.png",
          "id": "0",
          "alt": "Greenti",
          "titleText": "Greenti",
          "width": "48",
          "height": "48"
        }
      }
    }
  },
  "menu": {
    "advanced": {
      "menuId": { "desktop": { "value": "none" } },
      "style":  { "desktop": { "value": "left_aligned" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `logo`, `menu`.
`menu.advanced.menuId.desktop.value`: ID del menú registrado en WordPress (Apariencia → Menús) o `"none"`.
`menu.advanced.style.desktop.value`: `left_aligned`, `centered`, `inline_centered_logo`, `slide_in`.

---

#### `search` — Barra de búsqueda

**Categoría:** Widget.
**Uso:** Formulario de búsqueda de WordPress.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "searchPlaceholder": {
    "innerContent": { "desktop": { "value": "Buscar en el sitio..." } }
  },
  "button": {
    "innerContent": {
      "desktop": { "value": { "text": "Buscar" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `searchPlaceholder`, `button`.

**Nota:** puede opcionalmente contener un `heading` como bloque hijo (observado en un export), lo cual convertiría este módulo en contenedor en ese caso específico. Comportamiento no estándar; documentar cuando ocurra.

---

#### `login` — Formulario de login WP

**Categoría:** Widget.
**Uso:** Formulario de login de WordPress.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "title": {
    "innerContent": { "desktop": { "value": "Acceso a tu cuenta" } }
  },
  "content": {
    "innerContent": {
      "desktop": { "value": "<p>Inicia sesión para acceder al portal de clientes.</p>" }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `content`, `button`.

---

#### `sidebar` — Widget Area

**Categoría:** Widget.
**Uso:** Muestra los widgets de una sidebar registrada en WordPress. Útil en templates de post o layouts con aside.
**Tipo:** Hoja (self-closing).

**Ejemplo mínimo:**

```json
{
  "builderVersion": "5.8.1"
}
```

**Ejemplo con configuración:**

```json
{
  "module": {
    "advanced": {
      "sidebar": { "desktop": { "value": "sidebar-1" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

`sidebar.desktop.value`: nombre/slug de la sidebar registrada.

---

### 8.11 Módulos dinámicos (blog, portfolio, posts)

#### `blog` — Grid de posts

**Categoría:** Dinámico.
**Uso:** Grid o lista de posts recientes del blog.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "decoration": {
      "boxShadow": {
        "desktop": {
          "value": { "style": "preset3", "vertical": "7px", "blur": "15px", "color": "rgba(0,0,0,0.07)" }
        }
      }
    }
  },
  "post": {
    "advanced": {
      "number": { "desktop": { "value": "3" } },
      "offset": { "desktop": { "value": "0" } },
      "categories": { "desktop": { "value": [] } },
      "orderBy": { "desktop": { "value": "date_desc" } }
    },
    "decoration": {
      "border": {
        "desktop": {
          "value": { "radius": { "sync": "on", "topLeft": "5px", "topRight": "5px", "bottomRight": "5px", "bottomLeft": "5px" } }
        }
      }
    }
  },
  "pagination": {
    "advanced": { "enable": { "desktop": { "value": "off" } } }
  },
  "meta": {
    "advanced": {
      "showDate":       { "desktop": { "value": "on" } },
      "showCategories": { "desktop": { "value": "on" } },
      "showAuthor":     { "desktop": { "value": "off" } },
      "showComments":   { "desktop": { "value": "off" } }
    }
  },
  "title": {
    "decoration": {
      "font": { "font": { "desktop": { "value": { "family": "Plus Jakarta Sans", "weight": "600", "lineHeight": "1.2em" } } } }
    }
  },
  "content": {
    "decoration": {
      "bodyFont": {
        "body": { "font": { "desktop": { "value": { "family": "Inter", "size": "16px", "lineHeight": "1.8em" } } } }
      }
    }
  },
  "blogGrid": {
    "decoration": {
      "layout": {
        "desktop":    { "value": { "display": "grid", "gridColumnCount": "3" } },
        "tabletWide": { "value": { "gridColumnCount": "2" } },
        "tablet":     { "value": { "gridColumnCount": "2" } },
        "phoneWide":  { "value": { "gridColumnCount": "1" } },
        "phone":      { "value": { "gridColumnCount": "1" } }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `post`, `title`, `content`, `meta`, `pagination`, `blogGrid`.

---

#### `portfolio` — Grid de portfolio items

**Categoría:** Dinámico.
**Uso:** Grid de proyectos de la categoría "portfolio" custom de Divi.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "post": {
    "advanced": {
      "number":  { "desktop": { "value": "9" } },
      "layout":  { "desktop": { "value": "grid" } },
      "categories": { "desktop": { "value": [] } }
    }
  },
  "meta": {
    "advanced": {
      "showTitle":      { "desktop": { "value": "on" } },
      "showCategories": { "desktop": { "value": "on" } }
    }
  },
  "pagination": {
    "advanced": { "enable": { "desktop": { "value": "on" } } }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `image`, `title`, `content`, `meta`, `overlay`, `pagination`.
`layout`: `grid` o `fullwidth`.

---

#### `filterable-portfolio` — Portfolio con filtros

**Categoría:** Dinámico.
**Uso:** Grid de portfolio con botones de filtro por categoría.
**Tipo:** Hoja (self-closing).

Similar a `portfolio` pero con `filter` group adicional para estilizar los botones de filtro.

**Ejemplo:**

```json
{
  "post": {
    "advanced": {
      "number":  { "desktop": { "value": "12" } },
      "categories": { "desktop": { "value": [] } }
    }
  },
  "filter": {
    "decoration": {
      "font": { "font": { "desktop": { "value": { "family": "Inter", "weight": "600", "color": "#3E0D61" } } } },
      "background": {
        "desktop": {
          "value": { "color": "transparent" },
          "hover": { "color": "#1EDFAE" }
        }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `image`, `title`, `content`, `meta`, `filter`, `overlay`, `pagination`.

---

#### `post-slider` — Slider de posts

**Categoría:** Dinámico.
**Uso:** Carousel de posts (título, extracto, imagen destacada, botón "Leer más").
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "post": {
    "advanced": {
      "number": { "desktop": { "value": "5" } },
      "orderBy": { "desktop": { "value": "date_desc" } }
    }
  },
  "meta": {
    "advanced": {
      "showDate": { "desktop": { "value": "on" } },
      "showAuthor": { "desktop": { "value": "on" } }
    }
  },
  "button": {
    "innerContent": {
      "desktop": { "value": { "text": "Leer artículo" } }
    }
  },
  "overlay": {
    "advanced": {
      "background": { "desktop": { "value": { "color": "rgba(62,13,97,0.7)" } } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `image`, `title`, `content`, `meta`, `button`, `overlay`.

---

#### `post-title` — Título de post dinámico (Theme Builder)

**Categoría:** Theme Builder / Dinámico.
**Uso:** Muestra el título del post/página actual. Solo tiene sentido en templates de Theme Builder.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "title": {
    "decoration": {
      "font": {
        "font": { "desktop": { "value": { "family": "Plus Jakarta Sans", "weight": "700", "size": "48px" } } }
      }
    }
  },
  "meta": {
    "advanced": {
      "showDate":       { "desktop": { "value": "on" } },
      "showAuthor":     { "desktop": { "value": "on" } },
      "showCategories": { "desktop": { "value": "on" } }
    }
  },
  "featuredImage": {
    "advanced": {
      "show": { "desktop": { "value": "on" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `meta`, `featuredImage`.

---

#### `post-content` — Contenido de post dinámico (Theme Builder)

**Categoría:** Theme Builder / Dinámico.
**Uso:** Renderiza el contenido del post/página actual (WYSIWYG). Solo en templates.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "decoration": {
      "headingFont": {
        "h1": {
          "font": {
            "desktop": {
              "value": {
                "family": "Plus Jakarta Sans",
                "weight": "700",
                "size": "32px",
                "color": "#3E0D61",
                "capitalization": "capitalize"
              }
            }
          }
        }
      },
      "bodyFont": {
        "body": {
          "font": {
            "desktop": {
              "value": {
                "textAlign": "left",
                "textWrap": "wrap"
              }
            }
          }
        }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`. Se estiliza mediante `headingFont` y `bodyFont` en `module.decoration`.

---

#### `comments` — Comentarios de post

**Categoría:** Dinámico.
**Uso:** Lista de comentarios + formulario de comentario del post actual. Solo en templates de single post.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "decoration": {
      "layout": { "desktop": { "value": { "display": "flex" } } }
    },
    "advanced": {
      "showReply": { "desktop": { "value": "off" } },
      "text": {
        "text": { "desktop": { "value": { "orientation": "center" } } }
      }
    }
  },
  "image": {
    "advanced": {
      "showAvatar": { "desktop": { "value": "off" } }
    }
  },
  "button": {
    "decoration": {
      "background": { "desktop": { "value": { "color": "#541690" } } }
    }
  },
  "commentCount": {
    "advanced": {
      "showCount": { "desktop": { "value": "on" } }
    }
  },
  "meta": {
    "advanced": {
      "showMeta": { "desktop": { "value": "off" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `image` (avatar), `button` (submit), `commentCount`, `meta`, `content`.

---

#### `table-of-contents` — Tabla de contenidos (nuevo en Divi 5)

**Categoría:** Widget de post.
**Uso:** Genera automáticamente un índice navegable con los headings del post actual.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "advanced": {
      "levels": { "desktop": { "value": ["h2", "h3", "h4"] } },
      "sticky": { "desktop": { "value": "on" } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`.
`levels`: array de heading levels a incluir (`h1` a `h6`).
`sticky`: `"on"` mantiene el TOC visible mientras se scrollea.

---

### 8.12 Módulos Fullwidth (los pocos que subsisten en Divi 5)

**Nota importante:** en Divi 5 no existen los módulos `fullwidth-*` que existían en Divi 4 (fullwidth-image, fullwidth-slider, fullwidth-menu, fullwidth-post-slider, fullwidth-post-title). Para lograr comportamiento fullwidth, se ajusta el `row` con `sizing.width: 100%` y `sizing.maxWidth: 100%` + padding 0 (ver sección 4.7).

Solo subsisten como módulos independientes:

#### `fullwidth-header` — Header fullwidth

**Categoría:** Contenido / Hero.
**Uso:** Hero fullwidth con overlay, título grande, subtítulo, hasta 2 botones e imagen.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "title": {
    "innerContent": { "desktop": { "value": "Bienvenido a Greenti" } },
    "decoration": {
      "font": {
        "font": { "desktop": { "value": { "family": "Plus Jakarta Sans", "weight": "700", "size": "64px", "color": "#ffffff" } } }
      }
    }
  },
  "subtitle": {
    "innerContent": { "desktop": { "value": "Consultora digital de LATAM" } }
  },
  "content": {
    "innerContent": {
      "desktop": { "value": "<p>Impulsamos marcas con VTEX, UX y customer intelligence.</p>" }
    }
  },
  "button1": {
    "innerContent": {
      "desktop": { "value": { "linkUrl": "/servicios", "text": "Ver servicios" } }
    }
  },
  "button2": {
    "innerContent": {
      "desktop": { "value": { "linkUrl": "/contacto", "text": "Contáctanos" } }
    }
  },
  "image": {
    "innerContent": {
      "desktop": {
        "value": {
          "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/hero-illustration.png",
          "alt": "Ilustración hero"
        }
      }
    }
  },
  "logo": {
    "innerContent": {
      "desktop": {
        "value": {
          "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/logo-greenti-white.png",
          "alt": "Greenti"
        }
      }
    }
  },
  "module": {
    "advanced": {
      "text": { "text": { "desktop": { "value": { "orientation": "center" } } } }
    },
    "decoration": {
      "background": {
        "desktop": {
          "value": {
            "color": "#3E0D61",
            "image": {
              "url": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/hero-bg.jpg",
              "size": "cover"
            }
          }
        }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `subtitle`, `content`, `button1`, `button2`, `image`, `logo`.

---

#### `fullwidth-portfolio` — Portfolio fullwidth

**Categoría:** Dinámico.
**Uso:** Portfolio edge-to-edge sin bordes de columna.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "post": {
    "advanced": {
      "number":     { "desktop": { "value": "9" } },
      "layout":     { "desktop": { "value": "carousel" } },
      "categories": { "desktop": { "value": [] } }
    }
  },
  "title": {
    "decoration": {
      "font": { "font": { "desktop": { "value": { "family": "Plus Jakarta Sans", "weight": "700" } } } }
    }
  },
  "image": {
    "decoration": {
      "border": { "desktop": { "value": { "radius": { "sync": "on", "topLeft": "0px" } } } }
    }
  },
  "overlay": {
    "advanced": {
      "background": { "desktop": { "value": { "color": "rgba(62,13,97,0.7)" } } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `image`, `title`, `overlay`.
`layout`: `grid` o `carousel`.

---

### 8.13 Módulos de mapa

#### `map` — Mapa (contenedor)

**Categoría:** Contenido / Widget.
**Uso:** Mapa Google Maps con pines geolocalizados.
**Tipo:** Contenedor. Contiene `map-pin`s.

**Ejemplo:**

```json
{
  "module": {
    "advanced": {
      "center": {
        "desktop": {
          "value": {
            "lat": -33.4172,
            "lng": -70.6042,
            "zoom": 12
          }
        }
      }
    },
    "decoration": {
      "sizing": { "desktop": { "value": { "height": "400px" } } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`.
Requiere que Google Maps API key esté configurada en Divi (Options → General).

---

#### `map-pin` — Pin de mapa (nuevo en Divi 5 como módulo capturado)

**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "pin": {
    "innerContent": {
      "desktop": {
        "value": {
          "lat": -33.4172,
          "lng": -70.6042,
          "address": "San Pascual 398, Las Condes, Santiago",
          "zoom": 18
        }
      }
    }
  },
  "title": {
    "innerContent": { "desktop": { "value": "Oficina Central Greenti" } }
  },
  "content": {
    "innerContent": {
      "desktop": { "value": "<p>Visítanos de lunes a viernes de 9:00 a 18:00.</p>" }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `pin`, `title`, `content`.

---

### 8.14 Otros módulos

#### `testimonial` — Testimonio

**Categoría:** Contenido.
**Uso:** Testimonial con cita, autor, cargo, empresa y foto.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "advanced": { "text": { "text": { "desktop": { "value": { "color": "dark" } } } } }
  },
  "content": {
    "innerContent": {
      "desktop": { "value": "<p>Greenti transformó nuestra tienda VTEX. Aumentamos 40% las conversiones.</p>" }
    }
  },
  "author": {
    "innerContent": { "desktop": { "value": "María González" } }
  },
  "jobTitle": {
    "innerContent": { "desktop": { "value": "Gerente de eCommerce" } }
  },
  "company": {
    "innerContent": {
      "desktop": { "value": { "text": "Retail Corp", "id": 0, "linkUrl": "https://retailcorp.com" } }
    }
  },
  "portrait": {
    "innerContent": {
      "desktop": {
        "value": {
          "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/portrait-maria.jpg",
          "id": 0, "width": "90", "height": "90",
          "alt": "Foto de María González"
        }
      }
    }
  },
  "quoteIcon": {
    "decoration": {
      "icon": { "desktop": { "value": { "show": "on", "color": "#1EDFAE" } } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `content`, `author`, `jobTitle`, `company`, `portrait`, `quoteIcon`.
`advanced.text.text.color`: `"dark"` o `"light"` (para adaptar contraste al fondo).

---

#### `team-member` — Miembro del equipo

**Categoría:** Contenido.
**Uso:** Card de perfil con foto, nombre, cargo, descripción y redes sociales.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "advanced": { "text": { "text": { "desktop": { "value": { "orientation": "center" } } } } },
    "decoration": {
      "background": { "desktop": { "value": { "color": "#ffffff" } } },
      "spacing": { "desktop": { "value": { "padding": { "top": "32px", "bottom": "32px" } } } },
      "border": { "desktop": { "value": { "radius": { "sync": "on", "topLeft": "12px" } } } }
    }
  },
  "image": {
    "innerContent": {
      "desktop": {
        "value": {
          "src": "https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/team-juan.jpg",
          "alt": "Juan Pérez"
        }
      }
    }
  },
  "title": {
    "innerContent": { "desktop": { "value": "Juan Pérez" } },
    "decoration": {
      "font": { "font": { "desktop": { "value": { "family": "Plus Jakarta Sans", "weight": "600", "size": "20px" } } } }
    }
  },
  "position": {
    "innerContent": { "desktop": { "value": "Director de eCommerce" } }
  },
  "content": {
    "innerContent": {
      "desktop": { "value": "<p>10 años impulsando marcas retail en LATAM.</p>" }
    }
  },
  "socialNetwork": {
    "innerContent": {
      "desktop": {
        "value": {
          "facebook": "",
          "twitter": "",
          "linkedin": "https://linkedin.com/in/juanperez",
          "instagram": ""
        }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `image`, `title`, `position`, `content`, `socialNetwork`.

---

#### `divider` — Divisor

**Categoría:** Utilidad.
**Uso:** Línea divisora horizontal entre secciones o dentro de columnas.
**Tipo:** Hoja (self-closing).

**Ejemplo:**

```json
{
  "module": {
    "decoration": {
      "spacing": { "desktop": { "value": { "padding": { "top": "0px", "bottom": "0px" } } } }
    }
  },
  "divider": {
    "advanced": {
      "line": {
        "desktop": {
          "value": {
            "color": "rgba(0,0,0,0.12)",
            "show": "on",
            "position": "center",
            "style": "solid",
            "weight": "1px"
          }
        }
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

`line.style`: `solid`, `dashed`, `dotted`, `double`.
`line.position`: `top`, `center`, `bottom`.
`line.show`: `"on"` muestra la línea, `"off"` oculta (usar como espaciador invisible).

---

### 8.15 Módulos de formularios (Divi nativos, uso limitado en v1)

#### `contact-form` — Formulario de contacto nativo

**Categoría:** Formulario.
**Uso limitado en v1:** Greenti prefiere Contact Form 7 externo. Cuando el HTML tenga `<form>`, la Skill emite un **Code Module** con placeholder de CF7 (ver `code` en 8.16), no este módulo. Se documenta por completitud.
**Tipo:** Contenedor. Envuelve `contact-field`s.

**Ejemplo:**

```json
{
  "module": {
    "decoration": {
      "background": { "desktop": { "value": { "color": "#ffffff" } } },
      "sizing": { "desktop": { "value": { "maxWidth": "450px", "alignment": "center" } } }
    }
  },
  "title": {
    "innerContent": { "desktop": { "value": "Contáctanos" } }
  },
  "field": {
    "decoration": {
      "font": { "font": { "desktop": { "value": { "family": "Inter" } } } },
      "border": { "desktop": { "value": { "styles": { "all": { "width": "1px", "color": "rgba(0,0,0,0.12)" } } } } }
    }
  },
  "button": {
    "decoration": {
      "background": { "desktop": { "value": { "color": "#541690" } } }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `title`, `field`, `checkbox`, `radio`, `button`.

---

#### `contact-field` — Campo de formulario

**Tipo:** Hoja (self-closing). Solo dentro de `contact-form`.

**Ejemplo:**

```json
{
  "fieldItem": {
    "advanced": {
      "id":             { "desktop": { "value": "Name" } },
      "allowedSymbols": { "desktop": { "value": "letters" } }
    },
    "innerContent": {
      "desktop": { "value": "Tu Nombre" }
    }
  },
  "builderVersion": "5.8.1"
}
```

---

#### `signup` — Formulario de suscripción

**Categoría:** Formulario / Conversión.
**Uso limitado en v1:** igual que contact-form, la Skill lo trata como CF7 placeholder si aparece un formulario de email opt-in.
**Tipo:** Hoja (self-closing).

Grupos: `module`, `content`, `button`, `field`, `checkbox`, `radio`, `resultMessage`.

---

### 8.16 Code Module (crítico)

#### `code` — Code Module

**Categoría:** Utilidad avanzada / Escape hatch.
**Uso:** Insertar HTML, CSS o JavaScript custom, o shortcodes de otros plugins. Es el "escape hatch" para cualquier patrón que no cubran los módulos nativos.
**Tipo:** Hoja (self-closing).

**Ejemplo con shortcode de CF7 (uso estándar en Greenti):**

```json
{
  "module": {
    "meta": {
      "adminLabel": {
        "desktop": { "value": "Formulario CF7 - reemplazar shortcode" }
      }
    },
    "decoration": {
      "layout": { "desktop": { "value": { "display": "block" } } }
    }
  },
  "content": {
    "innerContent": {
      "desktop": {
        "value": "[contact-form-7 id=\"INSERTA_ID_AQUI\" title=\"Inserta shortcode del formulario aquí\"]"
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Ejemplo con HTML+CSS custom (accordion custom, no que use el nativo):**

```json
{
  "module": {
    "meta": {
      "adminLabel": { "desktop": { "value": "Accordion custom - FAQ" } }
    }
  },
  "content": {
    "innerContent": {
      "desktop": {
        "value": "<style>.faq-item{border-bottom:1px solid #ccc;padding:16px 0}</style>\n<div class=\"faq-item\">\n  <details>\n    <summary>¿Cuánto tarda el desarrollo?</summary>\n    <p>Entre 4 y 8 semanas.</p>\n  </details>\n</div>"
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Grupos:** `module`, `content`.
El `content.innerContent.desktop.value` acepta HTML, CSS (dentro de `<style>`) y JS (dentro de `<script>`). El HTML se renderiza tal cual sin sanitizar — la Skill nunca inyecta código no revisado por el usuario.

---

## 9. Sección B — Módulos pendientes

En v1.1 esta sección está prácticamente vacía. Los 48 módulos observados en los 9 exports analizados cubren la totalidad de módulos nativos operativos en Divi 5.8.1 relevantes para el pipeline de Greenti.

Módulos que **no se documentan** en v1 porque quedan fuera de scope:
- Módulos de **WooCommerce** (Shop, Cart, Checkout, Product, Product Images, Product Price, etc.). Se agregarán en una v2 solo si Greenti empieza a construir tiendas WooCommerce en Divi.
- Módulos de **Extra** (Category Blurb, etc.) — extensión adicional que no aplica.

Si en el futuro aparece un módulo no documentado en un HTML de entrada, la Skill aplica el protocolo:

1. Detecta el módulo desconocido y avisa al usuario con nombre + contexto donde apareció.
2. Solicita export mínimo del módulo desde una instalación de Divi 5.8.1 (protocolo estándar).
3. El mantenedor del catálogo actualiza esta sección A con el schema del módulo.
4. La Skill vuelve a procesar la entrada.

---

## 10. Reglas específicas para `builderVersion 5.8.1`

- **Valor fijo** `"builderVersion": "5.8.1"` en cada bloque emitido por la Skill (independientemente de que módulos anteriores puedan traer versiones distintas en exports reales).
- Los `presets` se mantienen en `null`.
- Los `global_colors` y `global_variables` se mantienen en `[]` (salvo que el HTML de entrada declare variables globales; en ese caso se respetan y avisan).
- El wrapper `placeholder` es obligatorio.
- El bloque `placeholder` no lleva JSON de configuración (solo apertura y cierre).
- El bloque `section` que actúa como "slot vacío" al final del placeholder es opcional pero recomendado.
- `columnStructure` y `flexColumnStructure` se emiten siempre en el `row` (aunque sea `4_4` / `equal-columns_1`).
- `flexType` se emite siempre en el `column.decoration.sizing` alineado con el `advanced.type`.

---

## 11. Templates skeleton

Ver la carpeta `templates/` cuando se generen. Contendrá:

- `minimal-page.json` — página vacía importable.
- `section-basic.json` — sección estándar con 1 row + 1 column vacía.
- `section-3-columns.json` — sección con row de 3 columnas iguales.
- `section-specialty.json` — sección con sidebar (specialty).
- `code-module.json` — Code Module vacío listo para pegar shortcode o HTML.
- `cf7-placeholder.json` — Code Module con placeholder de CF7 (uso Greenti estándar).

Se generan en la Tanda 3 del pipeline de armado.

---

## 12. Referencias y notas finales

- Documento vivo: se actualiza cuando aparezcan módulos no cubiertos, se detecten patrones nuevos, o cambie la versión de Divi objetivo.
- Cualquier discrepancia observada entre este documento y un export real de Divi 5.8.1 debe reportarse y actualizarse aquí primero, antes de tocar la Skill o los subagentes.
- **Versión actual:** v1.1 — consolidación completa tras análisis de 9 exports reales.
- **Historial:**
  - v1.0 — 24 módulos documentados con schema, 30 pendientes en sección B, 3 breakpoints.
  - v1.1 — 48 módulos con schema real, sección B vacía, 5 breakpoints, sistema `columnStructure` + `flexColumnStructure`, `flexType` de 24 columnas, variables globales, patrón "contenedor + item" formalizado, notas sobre fullwidth.
