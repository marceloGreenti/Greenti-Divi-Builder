# Convención de HTML para formularios — Greenti (v1.3.0)

Este documento define el **contrato de convención** entre las Skills de Greenti que producen o consumen HTML de formularios. Es la referencia común para:

- **Skill de Figma → HTML** (o cualquier proceso que genere el HTML de las maquetas): debe emitir el HTML de formularios siguiendo esta convención.
- **Skill de HTML → Divi (html-to-divi)**: consume esta convención para mapear correctamente a Divi Form o Contact Form 7.
- **Skill de HTML → CF7 (futura, si se separa)**: consume esta convención directamente.

Cuando el HTML de entrada sigue esta convención, la Skill html-to-divi hace mapeo determinístico sin adivinar. Cuando NO la sigue, hace inferencia por CSS (fallback) con menor precisión.

## Estructura estándar de un formulario

```html
<form class="gt-form" action="#" method="post">

  <!-- Título del formulario (opcional) -->
  <h3 class="gt-form-title">Solicitar cotización gratuita</h3>
  <p class="gt-form-intro">Detalla ubicación + medidas en los comentarios...</p>

  <!-- Fila de 1 columna -->
  <div class="gt-form-row gt-form-row-1col">
    <label class="gt-form-field">
      <span class="gt-form-label">Correo electrónico *</span>
      <input type="email" name="email" required autocomplete="email" placeholder="tu@correo.cl">
    </label>
  </div>

  <!-- Fila de 2 columnas iguales -->
  <div class="gt-form-row gt-form-row-2col">
    <label class="gt-form-field">
      <span class="gt-form-label">Nombre completo *</span>
      <input type="text" name="nombre" required autocomplete="name" placeholder="Juan Pérez">
    </label>
    <label class="gt-form-field">
      <span class="gt-form-label">Teléfono *</span>
      <input type="tel" name="telefono" required placeholder="+56 9 XXXX XXXX">
    </label>
  </div>

  <!-- Fila de 2 columnas asimétricas (1/3 + 2/3) -->
  <div class="gt-form-row gt-form-row-2col-1-2">
    <label class="gt-form-field">
      <span class="gt-form-label">Comuna</span>
      <input type="text" name="comuna" placeholder="Las Condes">
    </label>
    <label class="gt-form-field">
      <span class="gt-form-label">Dirección</span>
      <input type="text" name="direccion" placeholder="Av. Ejemplo 123">
    </label>
  </div>

  <!-- Fila de 3 columnas -->
  <div class="gt-form-row gt-form-row-3col">
    <label class="gt-form-field">
      <span class="gt-form-label">País</span>
      <input type="text" name="pais" placeholder="Chile">
    </label>
    <label class="gt-form-field">
      <span class="gt-form-label">Ciudad</span>
      <input type="text" name="ciudad" placeholder="Santiago">
    </label>
    <label class="gt-form-field">
      <span class="gt-form-label">Región</span>
      <input type="text" name="region" placeholder="RM">
    </label>
  </div>

  <!-- Select con placeholder -->
  <div class="gt-form-row gt-form-row-1col">
    <label class="gt-form-field">
      <span class="gt-form-label">Tipo de vivienda</span>
      <select name="tipo_vivienda">
        <option value="">Seleccionar</option>
        <option>Casa</option>
        <option>Apartamento</option>
        <option>Oficina</option>
      </select>
    </label>
  </div>

  <!-- Textarea -->
  <div class="gt-form-row gt-form-row-1col">
    <label class="gt-form-field">
      <span class="gt-form-label">Mensaje</span>
      <textarea name="mensaje" rows="4" placeholder="¿En qué podemos ayudarte?"></textarea>
    </label>
  </div>

  <!-- Upload de archivos con dropzone visual -->
  <div class="gt-form-row gt-form-row-1col">
    <div class="gt-form-dropzone">
      <span class="gt-form-label">Adjuntar archivos</span>
      <p class="gt-form-hint">Planos, fotos o medidas · PDF, DWG, JPG, PNG · Máx. 10 MB por archivo</p>
      <input type="file" name="archivos" multiple accept=".pdf,.dwg,.jpg,.jpeg,.png">
    </div>
  </div>

  <!-- Aceptación de términos y condiciones -->
  <div class="gt-form-row gt-form-row-1col">
    <label class="gt-form-tyc">
      <input type="checkbox" name="acepta" required>
      <span>Acepto las <a href="/politicas-privacidad">políticas de privacidad y términos de uso</a> *</span>
    </label>
  </div>

  <!-- Botón submit -->
  <button type="submit" class="gt-form-submit">Enviar solicitud</button>

</form>
```

## Convención de clases

### Wrapper raíz

| Clase | Elemento | Requerido |
|---|---|---|
| `.gt-form` | `<form>` raíz | Sí, siempre |

### Filas (wrappers de layout)

| Clase | Layout resultante | Colapsa a 1 col en phone |
|---|---|---|
| `.gt-form-row` | Clase base común (siempre presente junto a un modificador) | — |
| `.gt-form-row-1col` | 1 columna (ancho completo) | — |
| `.gt-form-row-2col` | 2 columnas iguales (50/50) | Sí |
| `.gt-form-row-2col-1-2` | 2 columnas asimétricas (1/3 + 2/3) | Sí |
| `.gt-form-row-2col-2-1` | 2 columnas asimétricas (2/3 + 1/3) | Sí |
| `.gt-form-row-3col` | 3 columnas iguales (33/33/33) | Sí |

### Campos (wrappers dentro de las filas)

| Clase | Elemento HTML | Uso |
|---|---|---|
| `.gt-form-field` | `<label>` que envuelve un input + su label visible | Cuando hay label textual arriba del input |
| `.gt-form-label` | `<span>` con el texto del label | Dentro de `.gt-form-field` |

**Cuándo omitir `.gt-form-field`:** si el campo solo usa placeholder (sin label visible arriba), el input puede ir directo dentro de la fila. En ese caso, la Skill html-to-divi emitirá el CF7 en Modo Compacto.

### Elementos especiales

| Clase | Elemento | Uso |
|---|---|---|
| `.gt-form-title` | `<h3>` o similar | Título del formulario |
| `.gt-form-intro` | `<p>` | Párrafo introductorio bajo el título |
| `.gt-form-dropzone` | `<div>` | Wrapper visual de subida de archivos (drag and drop UI) |
| `.gt-form-hint` | `<p>` o `<small>` | Texto de ayuda pequeño (bajo el label, dentro de dropzone, etc.) |
| `.gt-form-tyc` | `<label>` | Wrapper de aceptación de términos y condiciones |
| `.gt-form-submit` | `<button>` o `<input type="submit">` | Botón de envío |

## Atributos HTML recomendados

Todos los inputs deben incluir estos atributos cuando sean aplicables:

| Atributo | Uso |
|---|---|
| `name` | **Obligatorio.** Nombre del campo en snake_case (`nombre_completo`, `correo_electronico`). |
| `type` | Tipo semántico (`text`, `email`, `tel`, `url`, `number`, `date`, `file`, `checkbox`, `radio`). |
| `required` | Cuando el campo es obligatorio. |
| `placeholder` | Texto de ejemplo (no reemplaza el label). |
| `autocomplete` | Hint para el navegador: `name`, `email`, `tel`, `postal-code`, etc. Ver [MDN autocomplete](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete). |
| `pattern` | Solo si se necesita validación regex custom (raro). |
| `accept` | Para `type="file"`: extensiones aceptadas. |
| `multiple` | Para `type="file"`: aceptar múltiples archivos. |

## Convención de nombres (`name` de los campos)

- **Snake_case:** `nombre_completo`, `correo_electronico`, `codigo_postal`.
- **Minúsculas.**
- **Sin tildes ni ñ:** `descripcion` (no `descripción`), `ano` (no `año`).
- **ASCII puro.**
- **Máximo 40 caracteres.**
- **Descriptivos y consistentes** entre proyectos: usar `nombre`, `email` (o `correo`), `telefono`, `mensaje`, `direccion` cuando sea posible.

## Layout responsive (implícito)

Todas las filas multi-columna colapsan automáticamente a 1 columna en viewports `< 768px`. **No es necesario declarar media queries** en el CSS del HTML de la maqueta. La Skill html-to-divi lo maneja al generar el CSS del formulario.

Si por algún motivo el diseño requiere que una fila NO colapse (raro), añadir la clase modificadora `.gt-form-row--no-collapse`. Este es un caso excepcional y debería documentarse en la maqueta.

## Cascada de fallback

Si el HTML NO sigue esta convención (viene de otra fuente, HTML "legacy" o proceso manual), la Skill html-to-divi hace fallback por inferencia CSS:

1. Detecta `display: grid` con `grid-template-columns` → mapea a fila multi-columna.
2. Detecta `display: flex` con children `flex: 1` → mapea a fila multi-columna equivalente.
3. Detecta `<label>` con `<span>` visible arriba del input → activa Modo B (con wrappers de label).
4. Detecta `<input>` suelto sin wrapper de fila → emite `.gt-cf7-full`.

La precisión del fallback es menor y puede requerir revisión manual del output. Por eso se recomienda **siempre seguir la convención** desde la Skill de Figma→HTML.

## Ejemplo real: formulario BKGlass en la convención Greenti

Este es el formulario de "Solicitar cotización" de BKGlass, expresado en la convención estándar:

```html
<form class="gt-form" action="#" method="post">

  <h3 class="gt-form-title">Solicitar cotización gratuita</h3>
  <p class="gt-form-intro">Detalla ubicación + medidas en los comentarios: "Living: ancho 200cm, alto 150cm, modelo corredera"</p>

  <!-- Nombre + Teléfono -->
  <div class="gt-form-row gt-form-row-2col">
    <label class="gt-form-field">
      <span class="gt-form-label">Nombre completo *</span>
      <input type="text" name="nombre" required autocomplete="name" placeholder="Juan Pérez">
    </label>
    <label class="gt-form-field">
      <span class="gt-form-label">Teléfono *</span>
      <input type="tel" name="telefono" required placeholder="+56 9 XXXX XXXX">
    </label>
  </div>

  <!-- Email -->
  <div class="gt-form-row gt-form-row-1col">
    <label class="gt-form-field">
      <span class="gt-form-label">Correo electrónico *</span>
      <input type="email" name="email" required autocomplete="email" placeholder="tu@correo.cl">
    </label>
  </div>

  <!-- Dirección + Comuna (asimétrico: dirección más ancha) -->
  <div class="gt-form-row gt-form-row-2col-2-1">
    <label class="gt-form-field">
      <span class="gt-form-label">Dirección</span>
      <input type="text" name="direccion" placeholder="Av. Ejemplo 123">
    </label>
    <label class="gt-form-field">
      <span class="gt-form-label">Comuna</span>
      <input type="text" name="comuna" placeholder="Las Condes">
    </label>
  </div>

  <!-- Tipo vivienda + Construcción nueva -->
  <div class="gt-form-row gt-form-row-2col">
    <label class="gt-form-field">
      <span class="gt-form-label">Tipo de vivienda</span>
      <select name="tipo_vivienda">
        <option value="">Seleccionar</option>
        <option>Casa</option>
        <option>Apartamento</option>
        <option>Oficina</option>
        <option>Edificio</option>
      </select>
    </label>
    <label class="gt-form-field">
      <span class="gt-form-label">¿Construcción nueva?</span>
      <select name="construccion_nueva">
        <option value="">Seleccionar</option>
        <option>Sí</option>
        <option>No</option>
      </select>
    </label>
  </div>

  <!-- Color marcos + Ventanas actuales -->
  <div class="gt-form-row gt-form-row-2col">
    <label class="gt-form-field">
      <span class="gt-form-label">Color de marcos</span>
      <select name="color_marcos">
        <option value="">Seleccionar</option>
        <option>Blanco</option>
        <option>Grafito</option>
        <option>Negro</option>
        <option>Imitación madera</option>
      </select>
    </label>
    <label class="gt-form-field">
      <span class="gt-form-label">Ventanas actuales</span>
      <select name="ventanas_actuales">
        <option value="">Seleccionar</option>
        <option>Aluminio</option>
        <option>Madera</option>
        <option>PVC</option>
        <option>Sin ventanas</option>
      </select>
    </label>
  </div>

  <!-- Descripción -->
  <div class="gt-form-row gt-form-row-1col">
    <label class="gt-form-field">
      <span class="gt-form-label">Descripción del proyecto</span>
      <textarea name="descripcion" rows="4" placeholder="Ej: Living: ancho 200cm, alto 150cm, modelo corredera..."></textarea>
    </label>
  </div>

  <!-- Upload de archivos -->
  <div class="gt-form-row gt-form-row-1col">
    <div class="gt-form-dropzone">
      <span class="gt-form-label">Adjuntar archivos</span>
      <p class="gt-form-hint">Planos, fotos o medidas · PDF, DWG, JPG, PNG · Máx. 10 MB por archivo</p>
      <input type="file" name="archivos" multiple accept=".pdf,.dwg,.jpg,.jpeg,.png">
    </div>
  </div>

  <!-- Submit -->
  <button type="submit" class="gt-form-submit">Enviar solicitud de cotización</button>

</form>
```

Este HTML es la fuente de verdad ideal. La Skill html-to-divi lo convertirá directamente al CF7 correspondiente sin ambigüedad.

## Notas de implementación

### Para la Skill de Figma→HTML

Cuando detectes un formulario en la maqueta de Figma, emítelo siguiendo esta convención:

1. Envolver todo en `<form class="gt-form">`.
2. Agrupar campos en filas según el layout visible en Figma.
3. Cada fila lleva `.gt-form-row` + un modificador de columnas.
4. Cada campo lleva `.gt-form-field` con `.gt-form-label` si hay label visible.
5. Los `name` de los campos derivarlos del label en snake_case.
6. Los inputs de aceptación de términos van en `.gt-form-tyc`.
7. El botón submit lleva `.gt-form-submit`.

### Para la Skill html-to-divi

Al detectar un `<form class="gt-form">`, activar la ruta de mapeo determinística (sin adivinar). Si el HTML NO trae la convención, hacer fallback por inferencia CSS. Ver `rules/cf7-form-generation.md` para detalles.

## Versionado de la convención

Esta convención es v1.3.0. Cambios futuros incluirán:

- **v1.4.0 posible:** filas con más de 3 columnas (`.gt-form-row-4col`).
- **v1.4.0 posible:** clases modificadoras para campos condicionales.
- **v1.5.0 posible:** soporte para floating labels con `.gt-form-field--floating`.

Cualquier cambio de la convención se documenta en `CHANGELOG.md` y se coordina con las Skills que la consumen.
