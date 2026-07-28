# Notas — bkglass-cotizacion

## Alcance de esta entrega

La entrada fue **solo el `<form>` extraído de la maqueta**, no una página completa.
Por eso esta corrida ejecuta únicamente la rama de Contact Form 7 de la Skill
(`rules/cf7-form-generation.md`) y **no** genera `divi-import-page.json`,
`divi-import-header.json`, `divi-import-footer.json`, `seo-meta.md`,
`performance-checklist.md` ni `assets-checklist.md`. Cuando llegue el HTML completo de la
página, esos archivos se generan en la misma carpeta sin rehacer el formulario.

## Formulario CF7 generado

- Ubicación prevista en la página: sección "Contacto" → Code Module.
- Modo de emisión: **B (con wrappers de label)** — la maqueta muestra `<label>` visible
  sobre cada input.
- Campos detectados: **11** + `[hidden page-title]` de trazabilidad.
- Archivos generados:
  - `output/cf7-form-config.md` — pegar en el admin de CF7.
  - `output/cf7-form-styles.css` — pegar en Divi → Opciones → CSS personalizado.
  - `design-tokens.md` — manifiesto de tokens inferidos desde los estilos inline.
  - `html/form-cotizacion-maqueta.html` — copia de la fuente para trazabilidad.

## Code Module para Divi (cuando se arme la página)

```
[contact-form-7 id="INSERTA_ID_AQUI"]
```
`adminLabel`: `Formulario CF7 - reemplazar shortcode`

---

## Decisiones tomadas (revisar antes de publicar)

### 1. Campos obligatorios — asunción explícita

La maqueta **no declara `required` en ningún campo**. Un formulario CF7 sin campos
obligatorios permite envíos completamente vacíos, así que se marcaron como obligatorios
los tres mínimos para poder responder una cotización:

| Campo | Estado emitido |
|---|---|
| `nombre` | obligatorio `[text*]` |
| `telefono` | obligatorio `[tel*]` |
| `email` | obligatorio `[email*]` |
| resto (dirección, comuna, 4 selects, descripción, archivos) | opcional |

**Si prefieres el comportamiento literal de la maqueta**, quita los asteriscos en la
pestaña Form. **Si quieres exigir adjuntos**, cambia `[mfile archivos …]` por
`[mfile* archivos …]`.

### 2. Fila "Dirección + Comuna" — aproximación de grilla

La maqueta usa `grid-template-columns: 1.5fr 1fr`. La convención estándar de la Skill no
tiene una clase para esa proporción, así que se emitió `.gt-cf7-half-2-1` (2fr 1fr), que es
la más cercana. Diferencia real: 66/33 en vez de 60/40 — visualmente casi idéntico.

`cf7-form-styles.css` incluye un bloque **comentado** para volver a `1.5fr 1fr` exacto si
quieres fidelidad 1:1 con el HTML original.

### 3. `.gt-cf7-foot` — extensión de la convención

La maqueta cierra con una nota de pie ("También puedes adjuntar planos al correo
ventanaspvc@bkglass.cl") que la convención `gt-cf7-*` v1.3.0 no cubre. Se añadió la clase
`.gt-cf7-foot` como extensión de proyecto, ya estilizada en el CSS.

**Sugerencia para la Skill:** promover `.gt-cf7-foot` a la convención estándar en v1.4.0
(es un patrón recurrente: nota legal / canal alternativo bajo el submit).

### 4. Labels en mayúsculas por CSS, no por texto

La maqueta escribe los labels en MAYÚSCULAS. Se emitieron en formato normal
("Nombre completo *") y el CSS aplica `text-transform: uppercase` + `letter-spacing: 0.5px`.
El resultado visual es idéntico y el texto queda legible para lectores de pantalla.

### 5. Color del `<select>` en estado placeholder

La maqueta pinta el `<select>` en `#8CA0B8` (gris de placeholder) porque está mostrando
"Seleccionar". El CSS reproduce eso, pero **el color se mantiene gris también al elegir una
opción** (CSS no puede detectar el estado seleccionado de un `<select>` sin JS).

Si te molesta, hay dos salidas:
- Dejar el `<select>` siempre en `#D8EEF7` (texto base) — cambia `color: #8CA0B8` por
  `color: #D8EEF7` en la regla `.gt-cf7-form select.wpcf7-form-control`.
- Añadir un JS mínimo que ponga una clase al cambiar el valor.

### 6. Sin consentimiento de datos personales

La maqueta no incluye checkbox de aceptación. El formulario recopila datos personales
(nombre, teléfono, correo, dirección), lo que en Chile cae bajo la **Ley 19.628**.
`cf7-form-config.md` incluye el snippet `[acceptance]` listo para pegar y el CSS ya
contempla `.gt-cf7-tyc`. **Decisión pendiente del cliente.**

### 7. Sin anti-spam

No hay captcha ni honeypot. Si el formulario recibe spam, ver la sección "Notas
importantes" de `cf7-form-config.md`.

### 8. Dropzone: orden, borde y textos

Ajustes hechos tras comparar el render contra la maqueta:

- **Sin doble borde.** El wrapper `.gt-cf7-dropzone` y el `.codedropz-upload-handler` del
  addon dibujaban cada uno su caja punteada (caja dentro de caja). El CSS ahora neutraliza
  el borde del addon cuando va anidado.
- **Hint bajo el botón.** La maqueta ordena: texto de arrastre → hint → botón. El addon
  renderiza texto y botón como un bloque indivisible, así que el hint quedó **debajo del
  botón**. Es la única desviación de orden respecto a la maqueta; el agrupamiento
  (letra chica al final) se lee bien. Con la alternativa `[file]` sin plugin el orden es
  input → hint.
- **El label "Adjuntar archivos" va en `.gt-cf7-sr-only`** (visible solo para lectores de
  pantalla). La maqueta no muestra ese label —el texto de arrastre cumple esa función— pero
  dejar el campo sin nombre accesible sería una regresión de a11y. Esto se desvía del
  Caso 2 de `rules/cf7-form-generation.md`, que lo emite visible.
- **Los textos del addon están en inglés por defecto.** Para dejarlos como la maqueta
  ("⬆ Arrastra archivos aquí o haz clic para seleccionar" / "Seleccionar archivos"), hay
  que cambiarlos en los ajustes del plugin (Contact → Drag & Drop Upload) o vía filtro en
  `functions.php`. **Verificar cómo lo expone la versión que instalen** — cambia entre
  versiones del addon.

---

## ⚠ Acciones requeridas antes de publicar

1. **CRÍTICO: Desactivar wpautop de CF7** en `functions.php` del child theme:
   ```php
   add_filter( 'wpcf7_autop_or_not', '__return_false' );
   ```
   Sin esto, el formulario aparecerá roto (columnas colapsadas, gaps enormes entre labels
   y campos).

2. Crear el CF7 en WordPress admin siguiendo `cf7-form-config.md`.

3. Reemplazar el `id="INSERTA_ID_AQUI"` en el Code Module por el ID que WordPress asigne.

4. **Configurar el correo destino** en la pestaña Mail (hoy va `[_site_admin_email]` como
   placeholder). Probablemente `ventanaspvc@bkglass.cl`.

5. Copiar el CSS de `cf7-form-styles.css` a Divi → Opciones → CSS personalizado.

6. **Instalar plugin "Drag and Drop Multiple File Upload – Contact Form 7"** (el campo
   `[mfile]` lo requiere). Si no quieres el plugin, usar la alternativa `[file]` de un solo
   archivo documentada en `cf7-form-config.md`.

7. **Traducir los textos del addon de upload** al español desde los ajustes del plugin
   (vienen en inglés por defecto). Ver punto 8 de "Decisiones tomadas".

8. **Instalar plugin de campo hidden** para `[hidden page-title id:page-title]`, o borrar
   esa línea y la línea `Página de origen:` del Mail.

9. **Verificar `upload_max_filesize` / `post_max_size` de PHP** con el hosting. CF7 acepta
   10 MB, pero el servidor puede estar limitado a 2 MB y el upload fallará en silencio.

10. Verificar que la fuente **DM Sans** esté cargada en el sitio (Divi → Opciones de diseño
    o el theme). El CSS la referencia pero no la importa.

11. (Opcional) Configurar plugin de webhook si se requiere integración con Puente OS u otro
    servicio.

12. (Opcional) Decidir sobre el `[acceptance]` de políticas de privacidad (punto 6 de
    "Decisiones tomadas").

---

## Verificación visual

Se renderizó `preview/cf7-preview.html` en Chromium headless (1360px y 420px) y se comparó
contra la maqueta objetivo. El preview reproduce el markup real que CF7 genera —incluidos
los `<span class="wpcf7-form-control-wrap">` y el markup del addon drag & drop— y carga
`output/cf7-form-styles.css` directamente, sin copiarlo.

Capturas: `preview/qa-desktop-1360px.png` · `preview/qa-phone-420px.png`.
El preview es solo para QA — **no se despliega**.

**Coincide** con la maqueta en: tarjeta (fondo, borde, radio, padding 40px), grilla de las
5 filas, tipografía y jerarquía, labels en mayúsculas con letter-spacing, campos, flecha
custom de los selects, dropzone punteada, botón full-width y nota de pie. En 420px todas
las filas colapsan a 1 columna y el padding baja a 24px.

**Diferencias conocidas del render:**

- El preview usa DM Sans desde Google Fonts. En producción debe estar cargada por el theme
  (ver acción requerida #10).
- El texto del dropzone se renderizó en español. El addon lo trae **en inglés por defecto**
  ("Drag & Drop Files Here" / "Browse Files") — ver punto 8 de "Decisiones tomadas".
- No se pudo verificar en un WordPress real: falta validar interacción (estados de error de
  CF7, spinner, lista de archivos cargados) y que ningún CSS global del theme pise el
  formulario.
