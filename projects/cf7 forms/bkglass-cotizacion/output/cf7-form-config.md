# Configuración de Contact Form 7 — bkglass-cotizacion

Formulario **"Solicitar cotización gratuita"** reconstruido desde
`html/form-cotizacion-maqueta.html`.

- Modo de emisión: **B (con wrappers de label)** — la maqueta tiene labels visibles.
- Convención de clases: `gt-cf7-*` (Skill html-to-divi v1.3.0).
- Campos detectados: **11** (+ 1 hidden de trazabilidad).
- Estilos compañeros: `output/cf7-form-styles.css`.

## Instrucciones de uso

1. En el admin de WordPress, ve a **Contact → Contact Forms → Add New**.
2. Ponle un título descriptivo: `BKGlass — Solicitar cotización`.
3. En la pestaña **Form**, borra el contenido default y pega el bloque "Form" de más abajo.
4. En la pestaña **Mail**, pega el contenido del bloque "Mail" y reemplaza el `To:` cuando tengas el correo real.
5. En la pestaña **Messages**, revisa los textos (opcional).
6. Guarda. WordPress te asigna un ID (visible en la lista de forms como `[contact-form-7 id="XX"]`).
7. Copia el ID.
8. Abre el Code Module en Divi con `adminLabel: "Formulario CF7 - reemplazar shortcode"`.
9. Reemplaza el placeholder `id="INSERTA_ID_AQUI"` por el ID real.
10. Pega el contenido de `cf7-form-styles.css` en Divi → Opciones → CSS personalizado.

## ⚠ Requisito importante: desactivar wpautop en CF7

Contact Form 7 aplica auto-formato de párrafos (`wpautop`) por defecto, lo cual inyecta
`<br>` y `<p>` que **rompen el layout de columnas** de este formulario.

Greenti actualmente NO tiene este filtro desactivado globalmente. Hay que añadirlo al
`functions.php` del child theme (o al plugin de configuración de Greenti):

```php
// Desactivar wpautop en Contact Form 7 (evita <br>/<p> automáticos)
add_filter( 'wpcf7_autop_or_not', '__return_false' );
```

**Si no se hace esto, el formulario se verá roto: columnas colapsadas, gaps enormes entre
labels y campos, etc.**

## ⚠ Plugins requeridos

| Plugin | Para qué | ¿Obligatorio? |
|---|---|---|
| **Drag and Drop Multiple File Upload – Contact Form 7** | Tag `[mfile]` — la maqueta declara `<input type="file" multiple>` | Sí, si quieres upload múltiple. Ver alternativa sin plugin más abajo. |
| **Contact Form 7 Hidden Field** (o equivalente) | Tag `[hidden page-title]` — trazabilidad de página de origen | No. Si no lo instalas, borra la línea `[hidden page-title id:page-title]` y la línea `Página de origen:` del Mail. |

---

## Pestaña "Form"

Pegar tal cual (reemplazando el contenido por defecto de CF7):

```
[hidden page-title id:page-title]

<div class="gt-cf7-form">

  <h3 class="gt-cf7-title">Solicitar cotización gratuita</h3>
  <p class="gt-cf7-intro">Detalla ubicación + medidas en los comentarios: "Living: ancho 200cm, alto 150cm, modelo corredera"</p>

  <div class="gt-cf7-half">
    <label class="gt-cf7-field">
      <span class="gt-cf7-label">Nombre completo *</span>
      [text* nombre autocomplete:name placeholder "Juan Pérez"]
    </label>
    <label class="gt-cf7-field">
      <span class="gt-cf7-label">Teléfono *</span>
      [tel* telefono autocomplete:tel placeholder "+56 9 ···· ····"]
    </label>
  </div>

  <div class="gt-cf7-full">
    <label class="gt-cf7-field">
      <span class="gt-cf7-label">Correo electrónico *</span>
      [email* email autocomplete:email placeholder "tu@correo.cl"]
    </label>
  </div>

  <div class="gt-cf7-half-2-1">
    <label class="gt-cf7-field">
      <span class="gt-cf7-label">Dirección</span>
      [text direccion autocomplete:street-address placeholder "Av. Ejemplo 123"]
    </label>
    <label class="gt-cf7-field">
      <span class="gt-cf7-label">Comuna</span>
      [text comuna autocomplete:address-level2 placeholder "Las Condes"]
    </label>
  </div>

  <div class="gt-cf7-half">
    <label class="gt-cf7-field">
      <span class="gt-cf7-label">Tipo de vivienda</span>
      [select tipo_vivienda first_as_label "Seleccionar" "Casa" "Apartamento" "Oficina" "Edificio"]
    </label>
    <label class="gt-cf7-field">
      <span class="gt-cf7-label">¿Construcción nueva?</span>
      [select construccion_nueva first_as_label "Seleccionar" "Sí" "No"]
    </label>
  </div>

  <div class="gt-cf7-half">
    <label class="gt-cf7-field">
      <span class="gt-cf7-label">Color de marcos</span>
      [select color_marcos first_as_label "Seleccionar" "Blanco" "Grafito" "Negro" "Imitación madera"]
    </label>
    <label class="gt-cf7-field">
      <span class="gt-cf7-label">Ventanas actuales</span>
      [select ventanas_actuales first_as_label "Seleccionar" "Aluminio" "Madera" "PVC" "Sin ventanas"]
    </label>
  </div>

  <div class="gt-cf7-full">
    <label class="gt-cf7-field">
      <span class="gt-cf7-label">Descripción del proyecto</span>
      [textarea descripcion rows:4 placeholder "Ej: Living: ancho 200cm, alto 150cm, modelo corredera..."]
    </label>
  </div>

  <div class="gt-cf7-dropzone">
    <span class="gt-cf7-label gt-cf7-sr-only">Adjuntar archivos</span>
    [mfile archivos filetypes:pdf|dwg|jpg|jpeg|png max-file:10mb]
    <p class="gt-cf7-hint">Planos, fotos o medidas · PDF, DWG, JPG, PNG · Máx. 10 MB por archivo</p>
  </div>

  <div class="gt-cf7-submit-wrap">
    [submit class:gt-cf7-submit "Enviar solicitud de cotización →"]
  </div>

  <p class="gt-cf7-foot">También puedes adjuntar planos al correo <a href="mailto:ventanaspvc@bkglass.cl">ventanaspvc@bkglass.cl</a></p>

</div>
```

### Alternativa sin plugin de upload múltiple

Si prefieres no instalar el addon "Drag and Drop Multiple File Upload", reemplaza la
línea del `[mfile]` por esta (acepta **un solo archivo**, con el mismo límite y tipos):

```
[file archivos limit:10485760 filetypes:pdf|dwg|jpg|jpeg|png]
```

`limit:10485760` = 10 MB en bytes. El CSS de `cf7-form-styles.css` cubre ambos casos
(input nativo `[type="file"]` y el markup del addon `.codedropz-upload-handler`).

### Notas de los form-tags

- **Campos obligatorios (`*`):** `nombre`, `telefono`, `email`. La maqueta HTML **no
  declara `required` en ningún campo**; se marcaron estos tres como obligatorios por ser
  el mínimo necesario para poder responder una cotización. Si quieres el comportamiento
  literal de la maqueta (todo opcional), quita los asteriscos. Si quieres exigir adjuntos,
  cambia `[mfile archivos ...]` por `[mfile* archivos ...]`.
- **Los `<select>`** usan `first_as_label "Seleccionar"`, que replica el
  `<option value="">Seleccionar</option>` de la maqueta: se ve como placeholder y no se
  envía como valor válido.
- **Labels en mayúsculas:** los `<span class="gt-cf7-label">` van escritos en formato
  normal; el CSS aplica `text-transform: uppercase` + `letter-spacing: 0.5px` para
  reproducir el look de la maqueta. Así el texto queda legible para lectores de pantalla.
- **`[hidden page-title]`** va **fuera** del `<div class="gt-cf7-form">`, al inicio, para
  que no altere el layout en flex.
- **`[response]` no se incluye**: el mensaje de CF7 se renderiza por defecto al final del
  `<form>`, como hermano del wrapper. El CSS lo estiliza vía
  `.gt-cf7-form ~ .wpcf7-response-output`. Si lo mueves dentro del wrapper, el CSS también
  contempla `.gt-cf7-form .wpcf7-response-output`.
- **`.gt-cf7-foot`** es una extensión de la convención estándar para la nota de pie de la
  maqueta ("También puedes adjuntar planos al correo…"). Está documentada en `notes.md`.
- **`.gt-cf7-sr-only`** oculta visualmente el label "Adjuntar archivos" pero lo deja
  disponible para lectores de pantalla: la maqueta no lo muestra (el texto de arrastre
  cumple esa función), pero el campo necesita un nombre accesible.
- **Textos del dropzone:** el addon los trae **en inglés** ("Drag & Drop Files Here" /
  "Browse Files"). Para dejarlos como la maqueta, cámbialos en los ajustes del plugin
  (Contact → Drag & Drop Upload) o vía filtro en `functions.php`. Verifica cómo lo expone
  la versión que instales.

---

## Pestaña "Mail"

```
To: [_site_admin_email]
```
> ⚠ **REEMPLAZAR** por el correo real de recepción (probablemente `ventanaspvc@bkglass.cl`).
> CF7 no permite guardar con el campo `To` vacío, por eso el placeholder.

```
From: BKGlass Web <wordpress@TU-DOMINIO.cl>
Subject: Nueva cotización BKGlass — [nombre]
Additional headers:
Reply-To: [email]
File attachments: [archivos]
```

**Message body:**

```
Nueva solicitud de cotización desde el sitio web.

— Datos de contacto —
Nombre:                [nombre]
Teléfono:              [telefono]
Correo:                [email]
Dirección:             [direccion]
Comuna:                [comuna]

— Proyecto —
Tipo de vivienda:      [tipo_vivienda]
¿Construcción nueva?:  [construccion_nueva]
Color de marcos:       [color_marcos]
Ventanas actuales:     [ventanas_actuales]

Descripción:
[descripcion]

Archivos adjuntos: ver adjuntos de este correo.

Página de origen: [page-title]

---
Enviado automáticamente por WordPress.
```

- Deja **desmarcada** la casilla "Use HTML content type" (el cuerpo es texto plano).
- `From:` debe usar un dominio del propio sitio para no fallar SPF/DKIM. No pongas ahí el
  correo del usuario: para responderle está el `Reply-To`.
- Si borraste el `[hidden page-title]`, borra también la línea `Página de origen:`.

---

## Pestaña "Mail (2)" — Autorespuesta al cliente (opcional)

Activa la casilla "Mail (2)" si quieres confirmar recepción al usuario:

```
To: [email]
From: BKGlass <wordpress@TU-DOMINIO.cl>
Subject: Recibimos tu solicitud — BKGlass
Additional headers:
Reply-To: [_site_admin_email]
```

**Message body:**

```
Hola [nombre],

Recibimos tu solicitud de cotización y te contactaremos a la brevedad.

Resumen de lo que nos enviaste:
- Tipo de vivienda: [tipo_vivienda]
- Comuna: [comuna]
- Descripción: [descripcion]

Si necesitas agregar planos o fotos, respóndenos este correo con los archivos adjuntos.

Saludos,
Equipo BKGlass
```

---

## Pestaña "Messages"

Los mensajes por defecto de CF7 en español sirven. Sugerencias:

| Mensaje | Texto sugerido |
|---|---|
| Sender's message was sent successfully | ¡Gracias! Recibimos tu solicitud y te contactaremos pronto. |
| Validation errors occurred | Uno o más campos tienen errores. Por favor verifica e intenta nuevamente. |
| There is a field that the sender must fill in | Este campo es obligatorio. |
| Email address that the sender entered is invalid | El correo ingresado no es válido. |
| Uploaded file is too large | El archivo supera el tamaño máximo permitido (10 MB). |
| Uploaded file is not allowed for file type | Ese tipo de archivo no está permitido. Usa PDF, DWG, JPG o PNG. |

---

## Pestaña "Additional Settings" (opcional)

Dejar vacío salvo indicación específica.

Ejemplos de uso:
- `subscribers_only: on` — solo usuarios logueados pueden enviar.
- `demo_mode: on` — no envía correos, solo simula (útil para probar el layout sin spamear).
- Configuraciones para plugins de webhook, honeypot, etc.

---

## Shortcode final (para el Code Module de Divi)

Tras guardar, CF7 te entrega algo como:

```
[contact-form-7 id="1234" title="BKGlass — Solicitar cotización"]
```

Reemplaza con eso el placeholder del Code Module:

```
[contact-form-7 id="INSERTA_ID_AQUI"]
```

---

## Notas importantes

- **Placeholder `[_site_admin_email]` en Mail:** el equipo debe reemplazarlo por el correo
  real donde se recibirán las consultas.
- **Integración con webhook (Puente OS u otro):** requiere plugin adicional tipo
  "CF7 to Webhook". Se configura por separado en la pestaña del formulario después de
  guardarlo.
- **Anti-spam:** este formulario no tiene captcha. Si el sitio recibe spam, considerar
  añadir un `[quiz]` con una pregunta simple, o configurar reCAPTCHA v3 global.
- **Cumplimiento legal (Ley 19.628 en Chile):** este formulario recopila datos personales
  y la maqueta **no incluye** checkbox de consentimiento. Recomendado añadir antes del
  submit:
  ```
  <label class="gt-cf7-tyc">
    [acceptance acepta] Acepto las <a href="/politicas-de-privacidad/">políticas de privacidad y el tratamiento de mis datos*</a> [/acceptance]
  </label>
  ```
  El CSS de `cf7-form-styles.css` ya contempla `.gt-cf7-tyc` por si lo agregas.
- **Uploads múltiples:** instalar plugin "Drag and Drop Multiple File Upload – Contact
  Form 7" o usar la alternativa `[file]` de un solo archivo (ver arriba).
- **Límite del servidor:** aunque CF7 acepte 10 MB, el `upload_max_filesize` /
  `post_max_size` de PHP puede ser menor (típico: 2 MB o 8 MB). Verificar con el hosting.
