# Design tokens — bkglass-cotizacion

Manifiesto inferido desde los estilos inline de `html/form-cotizacion-maqueta.html`.
Origen de cada token: `[extracted]` = leído literal del HTML · `[inferred]` = derivado
por la cascada de inferencia de la Skill (`rules/design-tokens-inference.md` +
`rules/cf7-form-generation.md` § "Reglas para resolver los tokens en el CSS").

## Color

| Token | Valor | Origen | Nota |
|---|---|---|---|
| `color.fondoTarjeta` | `#0B1A2E` | `[extracted]` | `background-color` del `<form>`. |
| `color.bordeTarjeta` | `rgba(168,212,232,0.1)` | `[extracted]` | `border` del `<form>`. |
| `color.textoBase` | `#D8EEF7` | `[extracted]` | Color de texto de inputs y textarea. |
| `color.textoTitulo` | `#FFFFFF` | `[extracted]` | Color del `<h3>`. |
| `color.textoSecundario` | `#8CA0B8` | `[extracted]` | Labels, intro y textos de ayuda. |
| `color.acento` | `#38C3FF` | `[extracted]` | Botón submit, link del correo, botón "Seleccionar archivos". |
| `color.acentoHover` | `#5BD0FF` | `[inferred]` | Acento con lightening ~10% para `:hover` del submit. |
| `color.textOnAcento` | `#070F1C` | `[extracted]` | `color` del `<button type="submit">`. Contraste vs `#38C3FF` ≈ 12.6:1 (WCAG AAA). |
| `color.error` | `#FF6B6B` | `[inferred]` | No declarado en la maqueta. Default de la Skill para proyectos oscuros. |
| `color.success` | `#25D366` | `[inferred]` | No declarado en la maqueta. Default de la Skill. |

## Formulario

| Token | Valor | Origen | Nota |
|---|---|---|---|
| `form.background` | `#152540` | `[extracted]` | Fondo de inputs, selects y textarea. |
| `form.border` | `rgba(168,212,232,0.15)` | `[extracted]` | Borde de inputs en reposo. |
| `form.borderFocus` | `#38C3FF` | `[inferred]` | La maqueta no declara `:focus`. Se usa el acento (regla de la Skill). |
| `form.text` | `#D8EEF7` | `[extracted]` | |
| `form.placeholder` | `rgba(140,160,184,0.6)` | `[inferred]` | `textoSecundario` a opacidad 0.6 (regla de la Skill). |
| `form.label` | `#8CA0B8` | `[extracted]` | |
| `form.radius` | `8px` | `[extracted]` | |
| `form.padding` | `12px 14px` | `[extracted]` | |
| `form.dropzoneBorder` | `rgba(168,212,232,0.2)` dashed 1.5px | `[extracted]` | |
| `form.dropzoneBg` | `rgba(168,212,232,0.03)` | `[extracted]` | |

## Tipografía

| Token | Valor | Origen |
|---|---|---|
| `font.body.family` | `'DM Sans', sans-serif` | `[extracted]` |
| `font.body.weight` | `300` | `[extracted]` |
| `font.body.size` | `14px` | `[extracted]` |
| `font.heading.family` | `'DM Sans', sans-serif` | `[extracted]` |
| `font.heading.weight` | `500` | `[extracted]` |
| `font.heading.size` | `20px` | `[extracted]` |
| `font.label.weight` | `500` | `[extracted]` |
| `font.label.size` | `12px` | `[extracted]` |
| `font.label.letterSpacing` | `0.5px` | `[extracted]` |
| `font.hint.weight` | `300` | `[extracted]` |
| `font.hint.size` | `11px` | `[extracted]` |
| `font.submit.size` | `16px` / weight `500` / `letter-spacing: 0.3px` | `[extracted]` |

## Espaciado y radios

| Token | Valor | Origen |
|---|---|---|
| `spacing.md` | `16px` | `[extracted]` (gap del grid y `margin-bottom` entre filas) |
| `spacing.labelGap` | `6px` | `[extracted]` |
| `spacing.cardPadding` | `40px` | `[extracted]` |
| `spacing.cardPaddingPhone` | `24px` | `[inferred]` (la maqueta no declara breakpoint; reducción estándar de la Skill) |
| `borderRadius.sm` | `4px` | `[inferred]` (checkbox de acceptance; no hay caso en la maqueta) |
| `borderRadius.md` | `8px` | `[extracted]` |
| `borderRadius.lg` | `16px` | `[extracted]` (tarjeta del formulario) |
| `borderRadius.btnGhost` | `6px` | `[extracted]` (botón "Seleccionar archivos") |

## Layout de filas detectado

Inferencia por CSS (el HTML de entrada **no** usa la convención `gt-form-*`, así que se
aplicó el fallback de `rules/convencion-html-formularios.md` § "Cascada de fallback").

| Fila | HTML de origen | Clase CF7 emitida |
|---|---|---|
| Nombre + Teléfono | `grid-template-columns: 1fr 1fr` | `.gt-cf7-half` |
| Correo electrónico | sin grid | `.gt-cf7-full` |
| Dirección + Comuna | `grid-template-columns: 1.5fr 1fr` | `.gt-cf7-half-2-1` ⚠ aproximación |
| Tipo vivienda + ¿Construcción nueva? | `1fr 1fr` | `.gt-cf7-half` |
| Color marcos + Ventanas actuales | `1fr 1fr` | `.gt-cf7-half` |
| Descripción del proyecto | sin grid | `.gt-cf7-full` |
| Adjuntar archivos | sin grid, `border: dashed` | `.gt-cf7-dropzone` |

**Modo de emisión: B (con wrappers de label)** — la maqueta tiene `<label>` visible
arriba de cada input.
