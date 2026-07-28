# Generación de Contact Form 7 (v1.3.0)

Este documento define cómo la Skill genera los archivos compañeros para Contact Form 7 cuando el usuario elige esa vía en la Fase 2. La consulta el subagente `divi-json-builder` durante la Fase 3.

## Contexto

Contact Form 7 (CF7) es un plugin de WordPress muy usado para formularios de contacto. Cuando el usuario elige CF7:

1. El Code Module con placeholder ya se emite en el JSON (`[contact-form-7 id="INSERTA_ID_AQUI"]`).
2. La Skill genera dos archivos compañeros:
   - `output/cf7-form-config.md` — configuración lista para pegar en CF7 admin.
   - `output/cf7-form-styles.css` — CSS custom para pegar en Divi → Opciones → CSS personalizado.

**Cambios importantes en v1.3.0:**

- Convención de clases estandarizada `gt-cf7-*` (Greenti CF7).
- Enfoque híbrido: compacto cuando la maqueta no tiene labels visibles, con wrappers de label cuando sí.
- CSS scopeado bajo el wrapper raíz para NO afectar otros formularios del sitio.
- Aviso automático sobre `wpcf7_autop_or_not` que Greenti no tiene desactivado globalmente.
- Detección de patrones de layout HTML → mapeo directo a clases CF7.
- Inferencia completa de estilos desde design tokens del proyecto.
- Manejo de casos especiales: aceptación de términos, upload de archivos, selects con placeholder, autorespuesta.

## Convención de clases estándar (v1.3.0+)

Todas las clases usan el prefijo `gt-cf7-` para evitar colisión con clases de Divi, WordPress, CF7 o el theme.

### Wrappers de layout (siempre presentes)

| Clase | Uso |
|---|---|
| `.gt-cf7-form` | Wrapper raíz del formulario. Contenedor principal que aísla los estilos. |
| `.gt-cf7-full` | Fila de 1 columna (input ancho completo). |
| `.gt-cf7-half` | Fila de 2 columnas iguales (50/50). |
| `.gt-cf7-half-1-2` | Fila de 2 columnas asimétricas (1/3 + 2/3). |
| `.gt-cf7-half-2-1` | Fila de 2 columnas asimétricas (2/3 + 1/3). |
| `.gt-cf7-third` | Fila de 3 columnas iguales (33/33/33). |

### Wrappers opcionales (solo cuando la maqueta tiene labels visibles)

| Clase | Uso |
|---|---|
| `.gt-cf7-field` | Wrapper de un campo con label visible arriba del input. |
| `.gt-cf7-label` | El `<span>` del label. |

### Elementos especiales

| Clase | Uso |
|---|---|
| `.gt-cf7-dropzone` | Wrapper de subida de archivos (drag and drop). |
| `.gt-cf7-hint` | Texto de ayuda pequeño (bajo el label o dentro de dropzone). |
| `.gt-cf7-tyc` | Wrapper de aceptación de términos y condiciones. |
| `.gt-cf7-submit-wrap` | Wrapper opcional del botón submit (útil para alinear/estilizar). |
| `.gt-cf7-title` | Título del formulario (`<h3>` o similar dentro del form). |
| `.gt-cf7-intro` | Párrafo introductorio bajo el título. |

### Responsive por defecto

Todas las filas multi-columna colapsan a 1 columna en viewports `< 768px`. Sin necesidad de declararlo por regla — es parte del CSS emitido por defecto.

## Modo híbrido de emisión (v1.3.0)

La Skill decide entre dos modos según el HTML de entrada:

### Modo A — Compacto (sin wrappers de label)

Se activa cuando la maqueta HTML muestra los campos **solo con placeholder**, sin `<label>` visible arriba del input.

Ejemplo de HTML de entrada:
```html
<form>
  <input type="email" name="email" required placeholder="Correo electrónico">
  <input type="text" name="nombre" required placeholder="Nombre y Apellido">
  <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px">
    <input type="tel" name="telefono" required placeholder="Teléfono">
    <input type="text" name="pais" required placeholder="País">
  </div>
</form>
```

Output CF7 emitido:
```html
<div class="gt-cf7-form">
  <div class="gt-cf7-full">[email* email autocomplete:email placeholder "Correo electrónico"]</div>
  <div class="gt-cf7-full">[text* nombre autocomplete:name placeholder "Nombre y Apellido"]</div>
  <div class="gt-cf7-half">[tel* telefono placeholder "Teléfono"][text* pais placeholder "País"]</div>
</div>
```

Nota: en modo compacto, los shortcodes CF7 van **directamente dentro del wrapper de fila**, sin `<label>` ni `<span>` adicional. CF7 crea automáticamente un `<span class="wpcf7-form-control-wrap">` alrededor de cada control, y el CSS lo estiliza.

### Modo B — Con wrappers de label

Se activa cuando la maqueta HTML tiene `<label>` visible arriba de cada input (o clases equivalentes como `.gt-form-label`).

Ejemplo de HTML de entrada:
```html
<form>
  <div class="gt-form-row-2col">
    <label class="gt-form-field">
      <span class="gt-form-label">NOMBRE COMPLETO</span>
      <input type="text" name="nombre" required placeholder="Juan Pérez">
    </label>
    <label class="gt-form-field">
      <span class="gt-form-label">TELÉFONO</span>
      <input type="tel" name="telefono" required placeholder="+56 9 XXXX XXXX">
    </label>
  </div>
</form>
```

Output CF7 emitido:
```html
<div class="gt-cf7-form">
  <div class="gt-cf7-half">
    <label class="gt-cf7-field">
      <span class="gt-cf7-label">Nombre completo *</span>
      [text* nombre placeholder "Juan Pérez"]
    </label>
    <label class="gt-cf7-field">
      <span class="gt-cf7-label">Teléfono *</span>
      [tel* telefono placeholder "+56 9 XXXX XXXX"]
    </label>
  </div>
</div>
```

Nota: en modo con wrappers, cada campo lleva su `<label>` con `<span class="gt-cf7-label">` para el texto visible del label. El shortcode CF7 sigue funcionando dentro.

## Detección de la convención Greenti HTML

Si el HTML de entrada usa la convención estándar `gt-form-*` (definida en el "contrato" con la Skill de Figma→HTML), la Skill hace mapeo directo sin adivinar:

| Clase HTML entrada | Clase CF7 emitida |
|---|---|
| `.gt-form-row-1col` | `.gt-cf7-full` |
| `.gt-form-row-2col` | `.gt-cf7-half` |
| `.gt-form-row-2col-1-2` | `.gt-cf7-half-1-2` |
| `.gt-form-row-2col-2-1` | `.gt-cf7-half-2-1` |
| `.gt-form-row-3col` | `.gt-cf7-third` |
| `.gt-form-field` | `.gt-cf7-field` (activa Modo B) |
| `.gt-form-label` | `.gt-cf7-label` (activa Modo B) |
| `.gt-form-tyc` | `.gt-cf7-tyc` |
| `.gt-form-submit` | `[submit class:gt-cf7-submit]` |

Ver la convención completa en `docs/convencion-html-formularios.md`.

## Detección por inferencia CSS (fallback)

Cuando el HTML NO usa la convención `gt-form-*`, la Skill infiere el layout del CSS:

| Patrón HTML detectado | Clase CF7 emitida |
|---|---|
| Grid con `grid-template-columns: 1fr` (o simplemente sin grid) | `.gt-cf7-full` |
| Grid con `grid-template-columns: 1fr 1fr` | `.gt-cf7-half` |
| Grid con `grid-template-columns: 1fr 2fr` | `.gt-cf7-half-1-2` |
| Grid con `grid-template-columns: 2fr 1fr` | `.gt-cf7-half-2-1` |
| Grid con `grid-template-columns: 1.5fr 1fr` | `.gt-cf7-half-2-1` (aproximación) |
| Grid con `grid-template-columns: 1fr 1fr 1fr` | `.gt-cf7-third` |
| Flex con 2 children `flex: 1` o `width: 50%` | `.gt-cf7-half` |
| Flex con 3 children `flex: 1` | `.gt-cf7-third` |
| Input suelto sin wrapper padre multi-column | `.gt-cf7-full` |
| `<label>` con `<span>` visible arriba del input | Activa Modo B (con wrappers) |

Para grids asimétricos con proporciones inusuales (`3fr 2fr`, etc.), aproximar al más cercano (`.gt-cf7-half-2-1`) y avisar en `notes.md`.

## Sintaxis de CF7 (referencia técnica)

CF7 usa su propia sintaxis de shortcodes dentro del contenido del formulario. Formato general de un campo:

```
[field-type* nombre-interno atributos...]
```

- `*` opcional al final del type indica campo required.
- `nombre-interno` es el ID del campo (sin espacios ni caracteres especiales).
- Atributos son opcionales según el tipo.

### Tipos de campo soportados

| Type CF7 | Uso |
|---|---|
| `text` | Input de texto simple. |
| `email` | Input tipo email con validación automática. |
| `tel` | Input tipo teléfono. |
| `url` | Input tipo URL con validación. |
| `number` | Input tipo número. |
| `date` | Input tipo fecha. |
| `textarea` | Área de texto multi-línea. |
| `select` | Dropdown. Requiere lista de opciones. |
| `checkbox` | Casillas de verificación (una o varias). |
| `radio` | Radio buttons. |
| `acceptance` | Checkbox de aceptación (típicamente términos). |
| `file` | Upload de archivo simple. |
| `mfile` | Upload múltiple con drag and drop (requiere plugin "Drag and Drop Multiple File Upload"). |
| `submit` | Botón de envío. |
| `quiz` | Anti-spam basado en pregunta. |
| `recaptcha` | Google reCAPTCHA (requiere configuración global). |
| `hidden` | Campo oculto (requiere plugin "CF7 Hidden Field" o alternativa). |

### Atributos comunes

- `placeholder "texto"` — placeholder del input.
- `default:"valor"` — valor por defecto.
- `id:mi-id` — CSS ID del campo.
- `class:mi-clase` — CSS class del campo.
- `autocomplete:name/email/tel/etc.` — hint de autocompletado para el navegador.
- `size:XX` — ancho en caracteres (uso limitado).
- `maxlength:XX` — máximo de caracteres.
- `minlength:XX` — mínimo de caracteres.
- Para `select`: `"Opción 1" "Opción 2" "Opción 3"` como lista.
- Para `select` con placeholder: `first_as_label "Seleccionar"` + lista.
- Para `checkbox` y `radio`: mismas opciones que select más `use_label_element` `default:1` (opción por defecto marcada).
- Para `file`: `limit:XX` (bytes), `filetypes:pdf|jpg|png|...`.
- Para `mfile`: `max-file:10mb` (formato humano).

## Mapeo de campos HTML → sintaxis CF7

Para cada `<input>`, `<textarea>` o `<select>` del HTML, la Skill emite el equivalente CF7:

| Elemento HTML | Sintaxis CF7 generada |
|---|---|
| `<input type="text" name="nombre" required placeholder="Tu nombre">` | `[text* nombre placeholder "Tu nombre"]` |
| `<input type="email" name="correo" required placeholder="tu@email.com" autocomplete="email">` | `[email* correo autocomplete:email placeholder "tu@email.com"]` |
| `<input type="tel" name="telefono" placeholder="+56 9 XXXX XXXX">` | `[tel telefono placeholder "+56 9 XXXX XXXX"]` |
| `<input type="url" name="sitio">` | `[url sitio]` |
| `<input type="number" name="cantidad" min="1" max="10">` | `[number cantidad min:1 max:10]` |
| `<input type="date" name="fecha">` | `[date fecha]` |
| `<textarea name="mensaje" placeholder="¿En qué podemos ayudarte?"></textarea>` | `[textarea mensaje placeholder "¿En qué podemos ayudarte?"]` |
| `<textarea name="mensaje" rows="4">` | `[textarea mensaje rows:4]` |
| `<select name="comuna"><option value="">Seleccionar</option><option>Las Condes</option><option>Providencia</option></select>` | `[select* comuna first_as_label "Seleccionar" "Las Condes" "Providencia"]` |
| `<input type="checkbox" name="acepto" required>` (aceptación TyC) | `[acceptance acepto]` |
| `<input type="checkbox" name="opciones" value="A">` + otros con mismo name (grupo) | `[checkbox opciones "A" "B" "C"]` |
| `<input type="radio" name="tipo" value="A">` + `<input type="radio" name="tipo" value="B">` | `[radio tipo "A" "B"]` |
| `<input type="file" name="planos" accept=".pdf,.jpg">` | `[file planos limit:10485760 filetypes:pdf|jpg]` |
| `<input type="file" name="planos" multiple accept=".pdf,.jpg,.png">` | `[mfile planos filetypes:pdf|jpg|png max-file:10mb]` + aviso plugin |
| `<button type="submit">Enviar</button>` | `[submit "Enviar"]` |
| `<button type="submit" class="gt-form-submit">Enviar</button>` | `[submit class:gt-cf7-submit "Enviar"]` |

## Reglas para completar cada bloque

### Bloque "Form"

- **NO envolver TODOS los campos en `<label>` individual salvo Modo B.** En Modo A, los campos van sueltos dentro del wrapper de fila.
- **Preservar el orden de los campos** del HTML.
- **Campos required (`*`) del HTML** → asterisco `*` en el type CF7.
- **Placeholders del HTML** → atributo `placeholder` del CF7.
- **`autocomplete` del HTML** → atributo `autocomplete:` del CF7 cuando sea relevante (email, name, tel).
- **Nombres de campos:** derivar del atributo `name` del HTML si viene, o del label visible (snake_case, ASCII, minúsculas). Ver reglas en la sección de `divi-json-builder.md`.
- **Selects con "Seleccionar" placeholder:** usar `first_as_label` en CF7.
- **Aceptación de términos:** usar `[acceptance]`, NO `[checkbox]`. Envolver en `.gt-cf7-tyc` con label descriptivo.
- **Uploads múltiples:** usar `[mfile]` y avisar en `notes.md` que requiere plugin "Drag and Drop Multiple File Upload – Contact Form 7".
- **Botón submit:** siempre `[submit "Texto"]`. Añadir `class:gt-cf7-submit` si el HTML lo declara con clase.
- **Response tag opcional:** al final del form, añadir `[response]` si el HTML declara un mensaje de respuesta explícito.
- **Hidden `page-title`:** al inicio del form, añadir `[hidden page-title id:page-title]` para trazabilidad en el email de notificación.

### Bloque "Mail"

- El `Subject` debe incluir al menos un campo del formulario para trazabilidad (típicamente el nombre).
- El `Reply-To` debe ser el correo del usuario (`[email]` o similar) para que las respuestas vayan al remitente.
- El `Message body` debe listar TODOS los campos del formulario con formato `Campo: [nombre-campo]`.
- **Destinatario (`To`):** placeholder `[_site_admin_email]` con nota de que debe reemplazarse por el correo real cuando Greenti lo defina.
- **`File attachments:`** si hay campos `[file]` o `[mfile]`, incluir el nombre del campo.
- **Include page-title:** si se emitió el `[hidden page-title]`, incluirlo en el body para saber desde qué página vino el envío.

### Bloque "Messages"

- Traducir al español natural (Greenti opera en LATAM).
- Los defaults son razonables; personalizar solo si el diseño lo pide explícitamente.
- Sugerencia para Success: `"¡Gracias! Recibimos tu solicitud y te contactaremos pronto."`

### Bloque "Additional Settings" (nuevo en v1.3.0)

Este bloque es OPCIONAL en CF7. La Skill NO lo genera con contenido por defecto pero SÍ menciona en las notas que el equipo puede añadir configuraciones específicas como:

- `subscribers_only: on` — solo usuarios logueados pueden enviar.
- `demo_mode: on` — no envía correos, solo simula.
- Configuraciones para plugins de webhook, honeypot, etc.

## Estructura del archivo `cf7-form-config.md`

El archivo generado tiene esta estructura obligatoria:

```markdown
# Configuración de Contact Form 7 — <nombre-proyecto>

## Instrucciones de uso

1. En el admin de WordPress, ve a **Contact → Contact Forms → Add New**.
2. Ponle un título descriptivo: `<nombre-proyecto> — <ubicación del form>`.
3. En la pestaña **Form**, borra el contenido default y pega el bloque "Form" de más abajo.
4. En la pestaña **Mail**, pega el contenido del bloque "Mail" y reemplaza el `To:` cuando tengas el correo real.
5. En la pestaña **Messages**, revisa los textos (opcional).
6. Guarda. WordPress te asigna un ID (visible en la lista de forms como `[contact-form-7 id="XX"]`).
7. Copia el ID.
8. Abre el Code Module en Divi con `adminLabel: "Formulario CF7 - reemplazar shortcode"`.
9. Reemplaza el placeholder `id="INSERTA_ID_AQUI"` por el ID real.
10. Pega el contenido de `cf7-form-styles.css` en Divi → Opciones → CSS personalizado.

## ⚠ Requisito importante: desactivar wpautop en CF7

Contact Form 7 aplica auto-formato de párrafos (`wpautop`) por defecto, lo cual inyecta `<br>` y `<p>` que **rompen el layout de columnas** de este formulario.

Greenti actualmente NO tiene este filtro desactivado globalmente. Hay que añadirlo al `functions.php` del child theme (o al plugin de configuración de Greenti):

\`\`\`php
// Desactivar wpautop en Contact Form 7 (evita <br>/<p> automáticos)
add_filter( 'wpcf7_autop_or_not', '__return_false' );
\`\`\`

**Si no se hace esto, el formulario se verá roto: columnas colapsadas, gaps enormes entre labels y campos, etc.**

Alternativa por formulario (menos limpia): añadir en la pestaña "Additional Settings" del formulario:
\`\`\`
skip_mail: off
\`\`\`
(No desactiva wpautop pero permite otros ajustes por-form.)

---

## Pestaña "Form"

<contenido generado del formulario CF7>

---

## Pestaña "Mail"

To: [_site_admin_email]  ⚠ REEMPLAZAR por correo real
From: [_site_title] <wordpress@<dominio>>
Subject: Nueva consulta desde <nombre-proyecto>: [nombre]
Additional headers:
Reply-To: [email]

File attachments: [archivos]  (si aplica)

Message body:
Se ha recibido una nueva consulta desde <nombre-proyecto>.

<campos del formulario mapeados>

Página de origen: [page-title]

---
Enviado automáticamente por WordPress.

---

## Pestaña "Mail (2)" — Autorespuesta al cliente (opcional)

Actívalo en la pestaña Mail (2) si quieres confirmar recepción al usuario:

To: [email]
From: <nombre-proyecto> <wordpress@<dominio>>
Subject: Recibimos tu solicitud — <nombre-proyecto>
Message body:
Hola [nombre],

Recibimos tu solicitud y te contactaremos pronto.

Saludos,
Equipo <nombre-proyecto>

---

## Pestaña "Messages"

Los mensajes por defecto de CF7 en español sirven. Sugerencias:
- Success: ¡Gracias! Recibimos tu solicitud y te contactaremos pronto.
- Validation error: Uno o más campos tienen errores. Por favor verifica e intenta nuevamente.
- Required field: Este campo es obligatorio.
- Invalid email: El correo ingresado no es válido.

---

## Pestaña "Additional Settings" (opcional)

Dejar vacío salvo indicación específica.

Ejemplos de uso:
- `subscribers_only: on` — solo usuarios logueados pueden enviar.
- `demo_mode: on` — no envía correos, solo simula.
- Configuraciones para plugins de webhook, honeypot, etc.

---

## Notas importantes

- **Placeholder `[_site_admin_email]` en Mail:** el equipo debe reemplazarlo por el correo real donde se recibirán las consultas.
- **Integración con webhook (Puente OS u otro):** requiere plugin adicional tipo "CF7 to Webhook". Se configura por separado en la pestaña del formulario después de guardarlo.
- **Anti-spam:** si el sitio recibe spam, considerar añadir `[quiz]` con una pregunta simple, o configurar reCAPTCHA global.
- **Cumplimiento legal (Ley 19.628 en Chile):** si el formulario recopila datos personales, incluir un `[acceptance]` con consentimiento explícito (ya incluido si la maqueta lo trae).
- **Uploads múltiples:** si hay campos `[mfile]`, instalar plugin "Drag and Drop Multiple File Upload – Contact Form 7".
```

## Estructura del archivo `cf7-form-styles.css`

El CSS generado hace que el formulario CF7 se vea idéntico al HTML de la maqueta. Está **scopeado bajo `.gt-cf7-form`** para NO afectar otros formularios del sitio.

```css
/* =====================================================================
   <nombre-proyecto> · Estilos del formulario Contact Form 7
   Generado por la Skill html-to-divi v1.3.0
   Copiar y pegar en: Divi → Opciones → CSS personalizado

   ⚠ REQUISITO: desactivar wpautop de CF7 en functions.php:
      add_filter( 'wpcf7_autop_or_not', '__return_false' );
   (ver cf7-form-config.md sección "Requisito importante")

   Todo scopeado bajo .gt-cf7-form → no afecta otros formularios.
============================================================ */

/* ---- Contenedor raíz ------------------------------------------------- */
.gt-cf7-form {
  display: flex;
  flex-direction: column;
  gap: <tokens.spacing.md>;
  font-family: <tokens.font.body.family>;
  box-sizing: border-box;
}
.gt-cf7-form *,
.gt-cf7-form *::before,
.gt-cf7-form *::after { box-sizing: border-box; }

/* ---- Filas de layout ------------------------------------------------- */
.gt-cf7-form .gt-cf7-full {
  display: block;
  width: 100%;
}

.gt-cf7-form .gt-cf7-half,
.gt-cf7-form .gt-cf7-half-1-2,
.gt-cf7-form .gt-cf7-half-2-1,
.gt-cf7-form .gt-cf7-third {
  display: grid;
  gap: <tokens.spacing.md>;
  width: 100%;
}

.gt-cf7-form .gt-cf7-half        { grid-template-columns: 1fr 1fr; }
.gt-cf7-form .gt-cf7-half-1-2    { grid-template-columns: 1fr 2fr; }
.gt-cf7-form .gt-cf7-half-2-1    { grid-template-columns: 2fr 1fr; }
.gt-cf7-form .gt-cf7-third       { grid-template-columns: 1fr 1fr 1fr; }

/* Colapso a 1 columna en phone */
@media (max-width: 767px) {
  .gt-cf7-form .gt-cf7-half,
  .gt-cf7-form .gt-cf7-half-1-2,
  .gt-cf7-form .gt-cf7-half-2-1,
  .gt-cf7-form .gt-cf7-third {
    grid-template-columns: 1fr;
  }
}

/* ---- Wrappers de campo (Modo B: cuando hay label visible) ---------- */
.gt-cf7-form .gt-cf7-field {
  display: flex;
  flex-direction: column;
  gap: 6px;
  margin: 0;
  min-width: 0;
}

.gt-cf7-form .gt-cf7-label {
  display: block;
  font-weight: <tokens.font.label.weight>;
  font-size: <tokens.font.label.size>;
  line-height: 1.2;
  color: <tokens.color.textoSecundario>;
  letter-spacing: 0.5px;
  margin: 0;
}

/* Hint (texto de ayuda pequeño) */
.gt-cf7-form .gt-cf7-hint {
  display: block;
  font-weight: 300;
  font-size: 11px;
  line-height: 1.4;
  color: <tokens.color.textoSecundario con opacidad 0.6>;
  margin: 0;
}

/* CF7 envuelve cada control en este span */
.gt-cf7-form .wpcf7-form-control-wrap {
  display: block;
  width: 100%;
  margin: 0;
}

/* ---- Inputs, selects y textarea ------------------------------------ */
.gt-cf7-form .wpcf7-form-control:not(.wpcf7-submit) {
  width: 100%;
  background-color: <tokens.form.background>;
  border: 1px solid <tokens.form.border>;
  border-radius: <tokens.borderRadius.md>;
  padding: 12px 14px;
  font-family: <tokens.font.body.family>;
  font-weight: <tokens.font.body.weight>;
  font-size: <tokens.font.body.size>;
  line-height: 1.4;
  color: <tokens.color.textoBase>;
  outline: none;
  transition: border-color 0.15s ease, box-shadow 0.15s ease;
  -webkit-appearance: none;
  appearance: none;
}

.gt-cf7-form textarea.wpcf7-form-control {
  min-height: 110px;
  resize: vertical;
}

.gt-cf7-form .wpcf7-form-control::placeholder {
  color: <tokens.color.textoSecundario con opacidad 0.6>;
  opacity: 1;
}

/* Focus state */
.gt-cf7-form .wpcf7-form-control:not(.wpcf7-submit):focus {
  border-color: <tokens.color.acento>;
  box-shadow: 0 0 0 3px <tokens.color.acento con opacidad 0.15>;
}

/* Autofill (Chrome) — mantener fondo del proyecto */
.gt-cf7-form .wpcf7-form-control:-webkit-autofill,
.gt-cf7-form .wpcf7-form-control:-webkit-autofill:focus {
  -webkit-text-fill-color: <tokens.color.textoBase>;
  -webkit-box-shadow: 0 0 0 1000px <tokens.form.background> inset;
  caret-color: <tokens.color.textoBase>;
}

/* ---- Select con flecha personalizada ------------------------------- */
.gt-cf7-form select.wpcf7-form-control {
  padding-right: 40px;
  cursor: pointer;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath fill='none' stroke='<tokens.color.acento URL-encoded>' stroke-width='1.6' d='M1 1.5 6 6l5-4.5'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 14px center;
  background-size: 12px 8px;
}

.gt-cf7-form select.wpcf7-form-control option {
  color: <tokens.color.textoBase>;
  background-color: <tokens.form.background>;
}

/* ---- Upload de archivos -------------------------------------------- */
.gt-cf7-form .gt-cf7-dropzone {
  border: 1.5px dashed <tokens.color.textoSecundario con opacidad 0.2>;
  border-radius: <tokens.borderRadius.md>;
  padding: 20px;
  text-align: center;
  background-color: <tokens.color.textoSecundario con opacidad 0.03>;
}

.gt-cf7-form input.wpcf7-form-control[type="file"],
.gt-cf7-form .codedropz-upload-handler {
  border: 1.5px dashed <tokens.color.textoSecundario con opacidad 0.20>;
  border-radius: <tokens.borderRadius.md>;
  background-color: <tokens.color.textoSecundario con opacidad 0.03>;
  color: <tokens.color.textoSecundario>;
  padding: 14px;
  font-size: 13px;
  cursor: pointer;
  width: 100%;
}

/* Addon Drag&Drop (si se usa [mfile]) */
.gt-cf7-form .codedropz-upload-handler {
  padding: 20px;
  text-align: center;
}
.gt-cf7-form .codedropz-upload-container .codedropz-btn-remove {
  color: <tokens.color.acento>;
}
.gt-cf7-form .dnd-upload-details {
  color: <tokens.color.textoBase>;
}

/* ---- Aceptación de términos ---------------------------------------- */
.gt-cf7-form .gt-cf7-tyc {
  color: <tokens.color.textoBase>;
  display: flex;
  flex-direction: column;
  gap: 4px;
  font-size: 13px;
  line-height: 1.5;
}

.gt-cf7-form .gt-cf7-tyc a {
  color: <tokens.color.acento>;
  text-decoration: underline;
  font-weight: 500;
}

.gt-cf7-form .gt-cf7-tyc .wpcf7-list-item {
  margin: 0;
}

/* Checkbox personalizado del acceptance */
.gt-cf7-form .wpcf7-list-item input[type="checkbox"] {
  appearance: none;
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  border: 1px solid <tokens.color.textoBase>;
  border-radius: <tokens.borderRadius.sm>;
  background: transparent;
  cursor: pointer;
  display: inline-block;
  vertical-align: middle;
  position: relative;
  transition: all 0.2s ease;
  margin-right: 8px;
}

.gt-cf7-form .wpcf7-list-item input[type="checkbox"]:checked::after {
  content: '';
  position: absolute;
  left: 6px;
  top: 2px;
  width: 4px;
  height: 8px;
  border: solid <tokens.color.textoBase>;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
}

/* ---- Botón submit ---------------------------------------------------- */
.gt-cf7-form .gt-cf7-submit-wrap {
  margin-top: 4px;
}

.gt-cf7-form input.wpcf7-submit {
  width: 100%;
  background-color: <tokens.color.acento>;
  color: <tokens.color.textOnAcento>;
  font-family: <tokens.font.body.family>;
  font-weight: 500;
  font-size: 16px;
  letter-spacing: 0.3px;
  padding: 16px;
  border: none;
  border-radius: <tokens.borderRadius.md>;
  cursor: pointer;
  transition: background-color 0.15s ease, transform 0.05s ease;
}

.gt-cf7-form input.wpcf7-submit:hover {
  background-color: <tokens.color.acentoHover>;
}

.gt-cf7-form input.wpcf7-submit:active {
  transform: translateY(1px);
}

.gt-cf7-form input.wpcf7-submit:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* Spinner de CF7 */
.gt-cf7-form .wpcf7-spinner {
  margin: 12px auto 0;
  display: block;
}

/* ---- Validación por campo ------------------------------------------ */
.gt-cf7-form .wpcf7-not-valid-tip {
  font-size: 12px;
  font-weight: 400;
  color: <tokens.color.error>;
  margin-top: 4px;
}

.gt-cf7-form .wpcf7-form-control.wpcf7-not-valid {
  border-color: <tokens.color.error>;
}

/* ---- Mensaje de respuesta global (solo de ESTE form) --------------- */
.gt-cf7-form ~ .wpcf7-response-output {
  font-family: <tokens.font.body.family>;
  font-size: 13px;
  border-radius: <tokens.borderRadius.md>;
  padding: 12px 16px;
  margin: 16px 0 0;
  color: <tokens.color.textoBase>;
}

.gt-cf7-form ~ .wpcf7-response-output.wpcf7-mail-sent-ok {
  border: 1px solid <tokens.color.success>;
  background-color: <tokens.color.success con opacidad 0.08>;
}

.gt-cf7-form ~ .wpcf7-response-output.wpcf7-validation-errors,
.gt-cf7-form ~ .wpcf7-response-output.wpcf7-mail-sent-ng {
  border: 1px solid <tokens.color.error>;
  background-color: <tokens.color.error con opacidad 0.08>;
}

/* ---- Título e intro del formulario (opcional) ---------------------- */
.gt-cf7-form .gt-cf7-title {
  font-family: <tokens.font.heading.family>;
  font-weight: <tokens.font.heading.weight>;
  font-size: 20px;
  color: <tokens.color.textoBase>;
  margin: 0 0 8px;
}

.gt-cf7-form .gt-cf7-intro {
  font-family: <tokens.font.body.family>;
  font-weight: 300;
  font-size: 13px;
  color: <tokens.color.textoSecundario>;
  margin: 0 0 24px;
  line-height: 1.6;
}
```

## Reglas para resolver los tokens en el CSS

Cada `<tokens.color.X>` o `<tokens.font.Y>` en el CSS anterior se reemplaza con el valor real del manifiesto confirmado en Fase 2.

### Tokens de formulario (inferidos por defecto en v1.3.0)

Si el manifiesto no declara tokens específicos de formulario, se infieren así:

- **`tokens.form.background`**: `tokens.color.fondoBase` con lightening del 5-8% (o color específico si el HTML lo declara). Para BKGlass: HTML declara `#152540` (más claro que el fondo base `#070F1C`).
- **`tokens.form.border`**: derivado del `textoBase` con opacidad 0.15 (patrón sutil). Ejemplo: `rgba(216, 238, 247, 0.15)` sobre fondo oscuro.
- **`tokens.form.borderFocus`**: usar `tokens.color.acento` directamente.
- **`tokens.form.text`**: usar `tokens.color.textoBase`.
- **`tokens.form.placeholder`**: `tokens.color.textoSecundario` con opacidad 0.6.
- **`tokens.form.label`**: `tokens.color.textoSecundario`.
- **`tokens.form.radius`**: `tokens.borderRadius.md`.

### Tokens semánticos (nuevos en v1.3.0)

Si el manifiesto no define estos tokens explícitamente, la Skill los infiere:

- **`tokens.color.error`**: si no viene declarado, generar `#FF6B6B` (rojo semántico coherente con proyectos oscuros) o `#F71963` según intensidad del acento.
- **`tokens.color.success`**: si no viene declarado, generar `#25D366` (verde semántico).
- **`tokens.color.textOnAcento`**: color de texto sobre el fondo del acento (calcular contraste para asegurar WCAG AA). Para BKGlass con acento `#38C3FF`: `#070F1C`.
- **`tokens.color.acentoHover`**: acento con lightening del 10% para el hover del botón. Para BKGlass: `#5BD0FF`.

### Consideraciones responsive

El CSS emitido incluye media queries para colapso a 1 columna en phone (`< 768px`). No se emiten breakpoints intermedios salvo que el HTML de entrada declare comportamientos específicos por breakpoint.

## Casos especiales manejados en v1.3.0

### Caso 1 — Formulario con aceptación de términos

Cuando el HTML tiene un checkbox de TyC:

Input HTML:
```html
<label class="gt-form-tyc">
  <input type="checkbox" name="acepta" required>
  <span>Acepto los <a href="/politicas-privacidad">términos y condiciones</a> *</span>
</label>
```

Output CF7:
```html
<label class="gt-cf7-tyc">
  <b>Aceptación de términos</b>
  [acceptance acepta] Acepto haber leído las <a href="/politicas-privacidad/">Políticas de privacidad, términos y condiciones*</a> [/acceptance]
</label>
```

Nota: el `<b>` opcional para dar énfasis visual al título "Aceptación de términos". Puede omitirse si el diseño no lo pide.

### Caso 2 — Upload múltiple con validación

Cuando el HTML declara `<input type="file" multiple accept="...">`:

Output CF7:
```html
<div class="gt-cf7-dropzone">
  <span class="gt-cf7-label">Adjuntar archivos *</span>
  <p class="gt-cf7-hint">Planos, fotos o medidas · PDF, DWG, JPG, PNG · Máx. 10 MB por archivo</p>
  [mfile* archivos filetypes:pdf|dwg|jpg|jpeg|png max-file:10mb]
</div>
```

Y en `notes.md`:
```
- Upload múltiple detectado. Requiere plugin: "Drag and Drop Multiple File Upload – Contact Form 7".
```

### Caso 3 — Select con placeholder "Seleccionar"

Cuando el HTML tiene `<select>` con `<option value="">Seleccionar</option>`:

Output CF7:
```
[select* tipo_vivienda first_as_label "Seleccionar" "Casa" "Apartamento" "Oficina"]
```

Nota: `first_as_label` hace que la primera opción actúe como placeholder (no seleccionable como valor válido).

### Caso 4 — Autorespuesta al usuario

Siempre incluir el bloque "Mail (2)" en el `cf7-form-config.md` como opcional, con la nota de que se activa manualmente si Greenti quiere confirmación automática al usuario.

### Caso 5 — Campos de trazabilidad

Al inicio del form, siempre añadir:
```
[hidden page-title id:page-title]
```

Y en Mail body:
```
Página de origen: [page-title]
```

Esto permite saber desde qué página se envió el formulario cuando hay múltiples formularios en el sitio.

## Registro en `notes.md`

Cuando se genera un formulario CF7, en `notes.md` se añade:

```markdown
## Formulario CF7 generado

- Ubicación en la página: <section, row>
- Modo de emisión: <Compacto | Con label wrappers>
- Campos detectados: N
- Archivos generados:
  - `output/cf7-form-config.md` — pegar en el admin de CF7.
  - `output/cf7-form-styles.css` — pegar en Divi → Opciones → CSS personalizado.

### ⚠ Acciones requeridas antes de publicar

1. **CRÍTICO: Desactivar wpautop de CF7** en `functions.php` del child theme:
   ```php
   add_filter( 'wpcf7_autop_or_not', '__return_false' );
   ```
   Sin esto, el formulario aparecerá roto (columnas colapsadas, gaps enormes).

2. Crear el CF7 en WordPress admin siguiendo `cf7-form-config.md`.

3. Reemplazar el `id="INSERTA_ID_AQUI"` en el Code Module por el ID que WordPress asigne.

4. Configurar correo destino en `cf7-form-config.md` sección Mail.

5. Copiar el CSS de `cf7-form-styles.css` a Divi → Opciones → CSS personalizado.

6. Si hay campos [mfile] (upload múltiple): instalar plugin "Drag and Drop Multiple File Upload – Contact Form 7".

7. (Opcional) Configurar plugin de webhook si se requiere integración con Puente OS u otro servicio.
```

## Cuándo NO generar los archivos CF7

- Si el usuario eligió Divi Form nativo → no generar archivos CF7.
- Si el HTML no tiene formularios → no generar nada.
- Si el `<form>` es de tipo `search`, `login` o `comments` → mapear al módulo Divi correspondiente, no generar CF7.
