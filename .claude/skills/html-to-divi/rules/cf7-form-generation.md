# Generación de Contact Form 7

Este documento define cómo la Skill genera los archivos compañeros para Contact Form 7 cuando el usuario elige esa vía en la Fase 2. La consulta el subagente `divi-json-builder` durante la Fase 3.

## Contexto

Contact Form 7 (CF7) es un plugin de WordPress muy usado para formularios de contacto. Cuando el usuario elige CF7:

1. El Code Module con placeholder ya se emite en el JSON (`[contact-form-7 id="INSERTA_ID_AQUI"]`).
2. La Skill genera dos archivos compañeros:
   - `output/cf7-form-config.md` — configuración lista para pegar en CF7 admin.
   - `output/cf7-form-styles.css` — CSS custom para pegar en Divi → Opciones → CSS personalizado.

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
| `file` | Upload de archivo. |
| `submit` | Botón de envío. |
| `quiz` | Anti-spam basado en pregunta. |
| `recaptcha` | Google reCAPTCHA (requiere configuración global). |

### Atributos comunes

- `placeholder "texto"` — placeholder del input.
- `default:"valor"` — valor por defecto.
- `id:mi-id` — CSS ID del campo.
- `class:mi-clase` — CSS class del campo.
- `size:XX` — ancho en caracteres (uso limitado).
- `maxlength:XX` — máximo de caracteres.
- `minlength:XX` — mínimo de caracteres.
- Para `select`: `"Opción 1" "Opción 2" "Opción 3"` como lista.
- Para `checkbox` y `radio`: mismas opciones que select más `use_label_element` `default:1` (opción por defecto marcada).

## Mapeo de campos HTML → sintaxis CF7

Para cada `<input>`, `<textarea>` o `<select>` del HTML, la Skill emite el equivalente CF7:

| Elemento HTML | Sintaxis CF7 generada |
|---|---|
| `<input type="text" name="nombre" required placeholder="Tu nombre">` | `[text* nombre placeholder "Tu nombre"]` |
| `<input type="email" name="correo" required placeholder="tu@email.com">` | `[email* correo placeholder "tu@email.com"]` |
| `<input type="tel" name="telefono" placeholder="+56 9 XXXX XXXX">` | `[tel telefono placeholder "+56 9 XXXX XXXX"]` |
| `<input type="url" name="sitio">` | `[url sitio]` |
| `<input type="number" name="cantidad" min="1" max="10">` | `[number cantidad min:1 max:10]` |
| `<input type="date" name="fecha">` | `[date fecha]` |
| `<textarea name="mensaje" placeholder="¿En qué podemos ayudarte?"></textarea>` | `[textarea mensaje placeholder "¿En qué podemos ayudarte?"]` |
| `<select name="comuna"><option>Las Condes</option><option>Providencia</option></select>` | `[select comuna "Las Condes" "Providencia"]` |
| `<input type="checkbox" name="acepto" required>` | `[acceptance acepto]` (si es de aceptación de términos) o `[checkbox acepto "Opción"]` (si es checkbox general) |
| `<input type="radio" name="tipo" value="A">` + `<input type="radio" name="tipo" value="B">` | `[radio tipo "A" "B"]` |
| `<input type="file" name="planos" accept=".pdf,.jpg">` | `[file planos limit:10mb filetypes:pdf\|jpg]` |
| `<button type="submit">Enviar</button>` | `[submit "Enviar"]` |

## Estructura del archivo `cf7-form-config.md`

El archivo generado tiene esta estructura obligatoria:

```markdown
# Configuración de Contact Form 7 — <nombre-proyecto>

## Instrucciones de uso

1. En el admin de WordPress, ve a **Contact → Contact Forms → Add New**.
2. Ponle un título descriptivo: `<nombre-proyecto> — <ubicación del form>`.
3. En la pestaña **Form**, borra el contenido default y pega el bloque "Form" de más abajo.
4. En la pestaña **Mail**, pega el contenido del bloque "Mail".
5. En la pestaña **Messages**, pega el contenido del bloque "Messages" (opcional, puedes usar los defaults).
6. En la pestaña **Additional Settings**, pega el contenido del bloque "Additional Settings" (si aplica).
7. Guarda. WordPress te asigna un ID (visible en la lista de forms como `[contact-form-7 id="XX"]`).
8. Copia el ID.
9. Abre el Code Module en Divi con `adminLabel: "Formulario CF7 - reemplazar shortcode"`.
10. Reemplaza el placeholder `id="INSERTA_ID_AQUI"` por el ID real.
11. (Opcional) Pega el contenido de `cf7-form-styles.css` en Divi → Opciones → CSS personalizado.

---

## Form

<contenido generado del formulario CF7>

---

## Mail

To: <PENDIENTE - correo destino>
From: [_site_title] <wordpress@<dominio>>
Subject: Nueva consulta desde <nombre-proyecto>: [nombre]
Additional headers:
Reply-To: [correo]

Message body:
Se ha recibido una nueva consulta desde el formulario de <nombre-proyecto>.

<campos del formulario mapeados>

---
Enviado automáticamente por WordPress.

---

## Messages

Success: Gracias por contactarnos. Hemos recibido tu consulta y te responderemos pronto.
Validation error: Uno o más campos tienen errores. Por favor verifica e intenta nuevamente.
Spam: Hubo un error al enviar tu mensaje. Por favor intenta nuevamente.
Required field: Este campo es obligatorio.
Invalid email: El correo ingresado no es válido.

---

## Additional Settings

(Opcional, dejar vacío salvo indicación específica)

---

## Notas importantes

- **Placeholder `<PENDIENTE - correo destino>` en la sección Mail:** el equipo debe reemplazarlo por el correo real donde se recibirán las consultas.
- **Integración con webhook (Puente OS u otro):** requiere un plugin adicional tipo "CF7 to Webhook". Se configura por separado en la pestaña del formulario después de guardarlo por primera vez.
- **Anti-spam:** si el sitio recibe spam, considerar añadir `[quiz]` con una pregunta simple, o configurar reCAPTCHA global.
- **Cumplimiento legal (Ley 19.628 en Chile):** si el formulario recopila datos personales, añadir un `[acceptance]` con texto de consentimiento explícito.
```

### Reglas para completar cada bloque

**Bloque "Form":**
- Envolver cada campo en un `<label>` con el texto de label visible del HTML.
- Preservar el orden de los campos del HTML.
- Los campos required del HTML → asterisco `*` en el type CF7.
- Los placeholders del HTML → atributo `placeholder` del CF7.
- Los grupos de campos en 2 columnas del HTML → añadir clase CF7 tipo `class:form-row-2col` para poder estilizar con CSS.

**Bloque "Mail":**
- El `Subject` debe incluir al menos un campo del formulario para trazabilidad (típicamente el nombre).
- El `Reply-To` debe ser el correo del usuario (no el correo del sitio) para que las respuestas vayan al remitente.
- El `Message body` debe listar TODOS los campos del formulario con formato `Campo: [nombre-campo]`.

**Bloque "Messages":**
- Traducir al español natural (Greenti opera en LATAM).
- Los defaults son razonables; personalizar solo si el diseño lo pide explícitamente.

## Estructura del archivo `cf7-form-styles.css`

El CSS generado hace que el formulario CF7 se vea idéntico al HTML de la maqueta. Estructura:

```css
/* ============================================================
   Contact Form 7 — <nombre-proyecto>
   Estilos generados por la Skill html-to-divi v1.2.0
   Copiar y pegar en: Divi → Opciones → CSS personalizado
============================================================ */

/* Contenedor del formulario */
.wpcf7 {
  /* respetar container si aplica */
}

/* Layout de filas (2 columnas cuando aplique) */
.wpcf7 .form-row-2col {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  margin-bottom: 16px;
}

/* Layout de filas colapsa a 1 columna en phone */
@media (max-width: 767px) {
  .wpcf7 .form-row-2col {
    grid-template-columns: 1fr;
  }
}

/* Inputs y textarea */
.wpcf7 input[type="text"],
.wpcf7 input[type="email"],
.wpcf7 input[type="tel"],
.wpcf7 input[type="url"],
.wpcf7 input[type="number"],
.wpcf7 textarea,
.wpcf7 select {
  /* estilo respetando design tokens del proyecto */
  background-color: <tokens.color.formBackground>;
  border: 1px solid <tokens.color.formBorder>;
  border-radius: <tokens.borderRadius.md>;
  padding: 12px 16px;
  font-family: <tokens.font.body>;
  font-size: <tokens.font.body.size>;
  color: <tokens.color.textPrimary>;
  width: 100%;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

/* Placeholder color específico */
.wpcf7 input::placeholder,
.wpcf7 textarea::placeholder {
  color: <tokens.color.textSecondary>;
  opacity: 1;
}

/* Focus state */
.wpcf7 input:focus,
.wpcf7 textarea:focus,
.wpcf7 select:focus {
  outline: none;
  border-color: <tokens.color.accent>;
  box-shadow: 0 0 0 3px <tokens.color.accent con opacidad 0.15>;
}

/* Hover state */
.wpcf7 input:hover,
.wpcf7 textarea:hover,
.wpcf7 select:hover {
  border-color: <tokens.color.formBorderHover>;
}

/* Invalid state */
.wpcf7 .wpcf7-not-valid {
  border-color: <tokens.color.error>;
}

/* Labels */
.wpcf7 label {
  display: block;
  margin-bottom: 6px;
  font-family: <tokens.font.body>;
  font-size: 14px;
  color: <tokens.color.textSecondary>;
  font-weight: 500;
}

/* Botón submit */
.wpcf7 input[type="submit"] {
  background-color: <tokens.color.primary>;
  color: <tokens.color.textOnPrimary>;
  border: none;
  border-radius: <tokens.borderRadius.md>;
  padding: 14px 32px;
  font-family: <tokens.font.button>;
  font-weight: 600;
  font-size: 14px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  cursor: pointer;
  transition: background-color 0.2s ease, transform 0.1s ease;
  width: 100%;
}

.wpcf7 input[type="submit"]:hover {
  background-color: <tokens.color.primaryHover>;
}

.wpcf7 input[type="submit"]:active {
  transform: translateY(1px);
}

.wpcf7 input[type="submit"]:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* Mensajes de éxito y error de CF7 */
.wpcf7 .wpcf7-response-output {
  margin: 24px 0 0;
  padding: 16px;
  border-radius: <tokens.borderRadius.md>;
  font-family: <tokens.font.body>;
  font-size: 14px;
}

.wpcf7 .wpcf7-mail-sent-ok {
  background-color: <tokens.color.success con opacidad 0.15>;
  border: 1px solid <tokens.color.success>;
  color: <tokens.color.success>;
}

.wpcf7 .wpcf7-validation-errors,
.wpcf7 .wpcf7-mail-sent-ng {
  background-color: <tokens.color.error con opacidad 0.15>;
  border: 1px solid <tokens.color.error>;
  color: <tokens.color.error>;
}

/* Mensajes de validación por campo */
.wpcf7 .wpcf7-not-valid-tip {
  color: <tokens.color.error>;
  font-size: 13px;
  margin-top: 4px;
  display: block;
}

/* Checkboxes y radios */
.wpcf7 .wpcf7-checkbox,
.wpcf7 .wpcf7-radio,
.wpcf7 .wpcf7-acceptance {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-top: 8px;
}

.wpcf7 .wpcf7-list-item-label {
  color: <tokens.color.textSecondary>;
  font-size: 14px;
}
```

### Reglas para resolver los tokens en el CSS

Cada `<tokens.color.X>` en el CSS anterior se reemplaza con el valor real del manifiesto confirmado en Fase 2. Ejemplos con tokens de BKGlass:

- `<tokens.color.formBackground>` → si el HTML tiene un fondo específico para inputs, usarlo; si no, un color derivado del fondo principal.
- `<tokens.color.formBorder>` → borde sutil, típicamente el color primario con 10-15% opacidad.
- `<tokens.color.accent>` → color de acento del proyecto.
- `<tokens.color.error>` → si no está definido, generar un rojo semántico coherente.
- `<tokens.color.success>` → si no está definido, generar un verde semántico coherente.

### Consideraciones responsive

El CSS debe incluir media queries para los 3 breakpoints principales cuando el diseño lo requiera:
- Padding/margin diferentes en phone.
- Font-size ajustados en phone.
- Layout de 2 columnas colapsa a 1 en phone.

## Registro en `notes.md`

Cuando se genera un formulario CF7, en `notes.md` se añade:

```markdown
## Formulario CF7 generado

- Ubicación en la página: <section, row>
- Campos detectados: N
- Archivos generados:
  - `output/cf7-form-config.md` — pegar en el admin de CF7.
  - `output/cf7-form-styles.css` — pegar en Divi → Opciones → CSS personalizado.

### Acciones requeridas antes de publicar

1. Crear el CF7 en WordPress admin siguiendo `cf7-form-config.md`.
2. Reemplazar el `id="INSERTA_ID_AQUI"` en el Code Module por el ID que WordPress asigne.
3. Configurar correo destino en `cf7-form-config.md` sección Mail.
4. Copiar el CSS de `cf7-form-styles.css` a Divi.
5. (Opcional) Instalar plugin de webhook si se requiere integración con Puente OS u otro servicio.
```

## Cuándo NO generar los archivos CF7

- Si el usuario eligió Divi Form nativo → no generar archivos CF7.
- Si el HTML no tiene formularios → no generar nada.
- Si el `<form>` es de tipo `search`, `login` o `comments` → mapear al módulo Divi correspondiente, no generar CF7.
