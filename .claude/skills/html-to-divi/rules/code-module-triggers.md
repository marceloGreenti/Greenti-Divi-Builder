# Patrones que fuerzan Code Module

Este documento define cuándo el `divi-json-builder` debe emitir un Code Module en vez de un módulo nativo. La regla general de la Skill es priorizar módulos nativos siempre que sea posible; el Code Module es el "escape hatch" solo cuando ningún módulo nativo cubre el patrón.

## Cuándo emitir Code Module

### 1. Formularios (regla operativa de Greenti)

Cuando el HTML tenga cualquier `<form>`, emitir Code Module con este contenido:

```
[contact-form-7 id="INSERTA_ID_AQUI" title="Inserta shortcode del formulario aquí"]
```

Y `adminLabel` = `Formulario CF7 - reemplazar shortcode`.

**Excepciones que NO son formularios de contacto:**
- Formulario `<form role="search">` → mapear al módulo `search`.
- Formulario de login WP → mapear al módulo `login`.
- Formulario de comentarios de post → mapear al módulo `comments`.

### 2. Componentes con JavaScript custom no cubiertos por módulos nativos

Ejemplos:
- Slider/carousel con lógica muy específica que Divi no reproduce (transiciones custom, controles no estándar, integración con datos externos).
- Widgets de calculadoras interactivas.
- Player de audio custom con lógica de tracklist.
- Chat widgets embebidos (Intercom, Zendesk, etc.).
- Cotizadores dinámicos.

Regla: si el componente requiere JS que no cabe en las opciones de un módulo nativo, emitir Code Module con el HTML+JS tal cual, `adminLabel` = `CODE: <descripción breve>`.

### 3. Estilos CSS con features no soportadas por Divi

Ejemplos:
- `clip-path` complejos.
- `mask-image` con SVG.
- `filter` con múltiples filtros combinados no expuestos en Divi.
- Animaciones CSS complejas con múltiples keyframes.
- CSS Grid con `grid-template-areas` custom que no se puede reproducir con Divi rows.
- Container queries.
- CSS anchor positioning.
- `:has()` con lógica compleja.

Regla: si al mapear los estilos del HTML al módulo Divi correspondiente se pierde fidelidad visual significativa por limitaciones del módulo, emitir Code Module. Registrar en `notes.md` el motivo.

### 4. Web Components (elementos custom con guión en el nombre)

Ejemplos:
- `<mi-widget>`
- `<lit-slider>`
- `<stripe-checkout>`

Regla: siempre emitir Code Module. Divi no puede editar Web Components desde el constructor.

### 5. Iframes de terceros

Ejemplos:
- `<iframe src="https://tally.so/...">` (Tally forms)
- `<iframe src="https://calendly.com/...">` (Calendly)
- `<iframe src="https://player.vimeo.com/...">` (cuando el módulo `video` no es suficiente)
- Google Maps embebidos con configuración custom que no cubre el módulo `map`.

Regla: emitir Code Module con el iframe tal cual. `adminLabel` = `CODE: <servicio>`.

### 6. Shortcodes de otros plugins

Cualquier shortcode que no sea `[contact-form-7]` y que aparezca en el HTML:
- `[calendar id="..."]`
- `[woocommerce_cart]`
- `[testimonial_grid]`
- etc.

Regla: emitir Code Module con el shortcode tal cual. `adminLabel` = `CODE: <nombre-shortcode>`.

### 7. Contenido con marcadores dinámicos no resolubles

Ejemplos:
- `{{ACF:campo_nombre}}`
- `{{user.name}}`
- `[dynamic:field]`

Regla: si son marcadores conocidos de Divi (`%%POST_TITLE%%`, etc.), preservarlos. Si son de otro sistema, emitir Code Module y avisar en `notes.md`.

### 8. Estilos inline con `!important` insistente

Cuando el HTML tiene estilos inline críticos con `!important` que no se pueden expresar naturalmente en un módulo Divi (porque Divi genera su propio CSS que compite), emitir Code Module.

### 9. Contenido con lógica condicional

Ejemplos:
- Bloques que se muestran solo si el usuario está logueado.
- Bloques con lógica de A/B testing.
- Contenido geolocalizado.

Regla: en v1 la Skill no soporta contenido condicional. Emitir Code Module con el HTML tal cual y avisar en `notes.md` que la lógica condicional se maneja fuera de Divi.

## Cuándo NO usar Code Module (aunque tiente)

- **Layouts complejos que Divi puede replicar con specialty sections + group.** Antes de rendirse al Code Module, intentar structurar con specialty section o `group` con `layout` flex/grid.
- **Animaciones que Divi soporta con `decoration.animation`.** Si es fade, slide, zoom, flip, fold, roll, bounce → usar el módulo nativo con esas propiedades.
- **Hover states complejos.** El sistema de `hover` en Divi cubre la mayoría de casos.
- **Componentes tipo "card" custom.** Casi siempre se pueden hacer con `group` + módulos hoja adentro.
- **Grids CSS estándar de N columnas.** Divi los soporta nativamente con `row` + `column` o con `blogGrid.layout.gridColumnCount`.

Regla: agotar las opciones de módulos nativos antes de recurrir al Code Module. Si hay dudas, preguntar al usuario.

## Estructura del Code Module emitido

Todo Code Module emitido lleva:

```json
{
  "module": {
    "meta": {
      "adminLabel": {
        "desktop": { "value": "CODE: <descripción clara>" }
      }
    },
    "decoration": {
      "layout": { "desktop": { "value": { "display": "block" } } }
    }
  },
  "content": {
    "innerContent": {
      "desktop": {
        "value": "<contenido HTML/CSS/JS escapado>"
      }
    }
  },
  "builderVersion": "5.8.1"
}
```

**Reglas del `adminLabel`:**

- Prefijo `CODE:` obligatorio para identificación visual en el árbol.
- Seguido de descripción concreta: `CODE: Formulario CF7 - reemplazar shortcode`, `CODE: Cotizador Puente OS`, `CODE: Iframe de Calendly`.
- No usar términos técnicos oscuros. Un usuario no-técnico debe entender qué hay ahí.

## Registro en `notes.md`

Toda emisión de Code Module se registra en `notes.md` con esta estructura:

```markdown
## Code Modules emitidos

### CODE: Formulario CF7 - reemplazar shortcode
- Ubicación: Sección "Contáctanos" → Row 1 → Column 2.
- Razón: Greenti gestiona formularios vía Contact Form 7 externo.
- Acción requerida antes de publicar: reemplazar el placeholder `[contact-form-7 id="INSERTA_ID_AQUI"]` por el shortcode real del formulario correspondiente.

### CODE: Cotizador Puente OS
- Ubicación: Sección "Servicios" → Row 3 → Column 1.
- Razón: Componente con lógica JS custom no cubierta por módulos nativos.
- Acción requerida: verificar que el JS del cotizador esté cargado en el sitio.
- Notas: el código puede necesitar ajustes de compatibilidad con el runtime de Divi.
```

## Cuando el patrón es ambiguo

Si el `divi-json-builder` duda entre emitir un módulo nativo o un Code Module, debe **preguntar al usuario** con esta estructura:

```
Detecté un patrón en el HTML que podría emitirse de 2 formas:

Opción A - Módulo nativo:
Emitir como `<módulo>` con configuración `<resumen>`.
Pro: administrable desde el constructor de Divi.
Contra: <limitación específica>.

Opción B - Code Module:
Emitir como Code Module con el HTML tal cual.
Pro: fidelidad visual completa.
Contra: no editable desde el constructor sin conocer código.

¿Cuál prefieres?
```

Registrar la decisión en `notes.md`.
