# Reglas de inferencia de design tokens

Este documento define la cascada de 4 pasos por la que la Skill obtiene los design tokens del proyecto. La consulta la Skill durante la Fase 2, antes de invocar al `divi-json-builder`.

## Los 7 grupos de tokens que se manejan

1. **Paleta de colores.** Primario, secundario, acento, texto base, fondo base, y adicionales.
2. **Tipografía.** Familias (headings, body, UI/botones), weights disponibles, escala tipográfica por breakpoint.
3. **Sistema de espaciado.** Base (típicamente 8px) y escala derivada (8, 16, 24, 32, 48, 64, 96...).
4. **Radios de borde.** sm, md, lg, pill.
5. **Sombras (opcional).** Elevaciones (sm, md, lg, xl).
6. **Sistema de contenedores.** Padding horizontal y vertical de sections, y ancho máximo de rows en pantallas extrawide. Se detalla en el bloque específico más abajo.
7. **Formulario (v1.3.0+, inferido por defecto).** Colores y estilos específicos del formulario (fondo, borde, focus, placeholder, label). Ver bloque específico más abajo.

## Cascada de 4 pasos

La Skill aplica los pasos en orden estricto. Cada paso solo se ejecuta si el anterior no cubrió el token.

### Paso 1 — Tokens explícitos (`design-tokens.md`)

Si existe `projects/<nombre>/design-tokens.md`, se lee y se aplica tal cual. Es la máxima autoridad; **no se cuestiona ni se completa** sin autorización explícita del usuario.

Formato esperado del archivo:

```markdown
# Design Tokens — <nombre-proyecto>

## Paleta de colores
- Primario:     #3E0D61  (uso: fondos primarios, botones principales)
- Secundario:   #541690  (uso: fondos secundarios, hovers)
- Acento:       #1EDFAE  (uso: CTAs, íconos destacados)
- Texto base:   #1A0037  (uso: párrafos y headings)
- Texto claro:  #FFFFFF  (uso: texto sobre fondos oscuros)
- Fondo base:   #FFFFFF
- Fondo claro:  #F0FDFF
- Fondo oscuro: #1A0037

## Tipografía
- Familia headings:   Plus Jakarta Sans   weights: 400, 600, 700
- Familia body:       Inter               weights: 400, 500, 600
- Familia UI/botones: Inter               weights: 500, 600

## Escala tipográfica
- H1:      desktop 48px / tablet 36px / phone 28px    line-height 1.2  letter-spacing 0
- H2:      desktop 36px / tablet 28px / phone 24px    line-height 1.3
- H3:      desktop 24px / tablet 22px / phone 20px    line-height 1.4
- H4:      desktop 20px / tablet 18px / phone 18px    line-height 1.4
- Body:    desktop 16px / tablet 16px / phone 15px    line-height 1.6
- Small:   desktop 14px / tablet 14px / phone 13px    line-height 1.5

## Sistema de espaciado
Base 8pt. Escala: 8, 16, 24, 32, 48, 64, 96, 128, 160.
Padding sections desktop: 96px vertical / 24px horizontal.
Padding sections phone:   48px vertical / 16px horizontal.

## Radios de borde
- sm: 4px  (inputs, badges)
- md: 8px  (botones, cards)
- lg: 16px (contenedores grandes)
- pill: 999px (chips redondeados)

## Sombras
- sm: 0 1px 2px rgba(0,0,0,0.05)
- md: 0 4px 8px rgba(0,0,0,0.08)
- lg: 0 8px 24px rgba(0,0,0,0.12)
- xl: 0 16px 48px rgba(0,0,0,0.16)

## Sistema de contenedores

### Padding horizontal de sections
Se aplica uniformemente a TODAS las sections del proyecto para garantizar alineación consistente.
- desktop:    80px
- tabletWide: 80px  (hereda de desktop si no se declara)
- tablet:     40px
- phoneWide:  24px  (interpolado si no se declara)
- phone:      20px

### Padding vertical de sections
- desktop:    100px (sections de contenido estándar)
- tabletWide: 100px (hereda de desktop)
- tablet:     72px
- phoneWide:  64px  (interpolado)
- phone:      56px

### Ancho máximo de contenedor (`contentMaxWidth`)
Opcional. Se aplica al maxWidth de los rows para evitar que el contenido se estire demasiado en pantallas ultrawide.
- desktop: 1400px  (o null si el diseño debe ser 100% fluido)

Valores típicos: null, 1400px, 1600px, 1800px.

### Padding del header (`headerPadding`)
El header/navbar tiene padding propio, más compacto que las sections de contenido. Se aplica al `<header>` semántico (o al primer bloque con clase equivalente).
- desktop:    14/48/14/48  (top/right/bottom/left)
- tabletWide: 14/48/14/48  (hereda de desktop)
- tablet:     12/32/12/32
- phoneWide:  10/24/10/24  (interpolado)
- phone:      10/20/10/20

Nota: es diferente del `sectionPadding` (que aplica a sections de contenido).

### Border-bottom del header (opcional)
Separación visual sutil entre el header y el hero.
- 1px solid rgba(<primaryLight>, 0.10)
```

Si el archivo existe pero está incompleto, la Skill completa los huecos con Pasos 2-4 y avisa qué se agregó.

### Paso 2 — Extracción del HTML

Si no hay `design-tokens.md` o hay huecos, extraer del HTML de entrada:

1. **Colores.** Detectar todos los valores hex, rgb, rgba en `style="..."`, `<style>`, y agruparlos. Los más frecuentes son los principales.
2. **Fuentes.** Detectar `font-family` en cualquier lugar (inline styles, `<style>`, atributos data-* de sistemas de diseño). Categorizar por uso (headings vs body).
3. **Tamaños tipográficos.** Detectar todos los `font-size`. Si forman una progresión clara (48, 36, 24, 20, 16, 14), construir la escala.
4. **Spacing.** Detectar valores repetidos en `padding` y `margin`. Si aparecen múltiplos de 8, asumir base 8pt.
5. **Radios de borde.** Detectar `border-radius`. Agrupar valores iguales.
6. **Sombras.** Detectar `box-shadow`. Extraer los distintos niveles.

Esto se hace **automáticamente**, sin preguntar al usuario. Los tokens extraídos van al manifiesto marcados como `[extracted]`.

### Paso 3 — Inferencia por buenas prácticas

Si aún quedan huecos después del Paso 2, aplicar heurísticas de sistemas de diseño:

1. **Colores complementarios.** Si tienes un color primario, generar un secundario armónico (más claro o más oscuro), un acento por contraste, y grises tonales.
2. **Escala tipográfica modular.** Si tienes 2-3 tamaños, extrapolar el resto con ratios comunes (1.25 Major Third, 1.333 Perfect Fourth, 1.5 Perfect Fifth).
3. **Sistema de espaciado base 8pt** por defecto: 8, 16, 24, 32, 48, 64, 96, 128.
4. **Radios de borde estándar:** sm 4px, md 8px, lg 16px, pill 999px.
5. **Sombras estándar:** sm/md/lg/xl con opacidad progresiva sobre negro.
6. **Fuentes por defecto** si no se puede inferir: `Inter` para body, `Plus Jakarta Sans` para headings. Ambas están en Google Fonts y cubren buenas prácticas modernas.

Los tokens generados así van al manifiesto marcados como `[inferred]`.

### Paso 3.5 — Inferencia específica del sistema de contenedores

El sistema de contenedores tiene reglas propias porque afecta la consistencia visual crítica del sitio. Se procesa así:

**A) Padding horizontal de sections:**

1. Buscar en el HTML las declaraciones de padding lateral de sections (`<section style="padding: X Y">`, `@media (max-width: 980px) { .section { padding: ... } }`, etc.).
2. Si hay valores consistentes (todas las sections del HTML usan el mismo padding en un breakpoint), extraerlos como el token.
3. Si hay variaciones menores (ej: la mayoría usa 80px pero una usa 100px), usar el más frecuente y avisar en notas.
4. Si el HTML no declara padding lateral, aplicar defaults: 80px desktop, 40px tablet, 20px phone.
5. Interpolar `tabletWide` y `phoneWide` a partir de desktop/tablet y tablet/phone respectivamente.

**B) Padding vertical de sections:**

1. Igual método: buscar declaraciones de padding vertical en sections del HTML.
2. Si hay consistencia, extraer.
3. Si no, aplicar defaults: 100px desktop, 72px tablet, 56px phone.

**C) `contentMaxWidth` (crítico — decisión inferencia B):**

1. Buscar en el HTML declaraciones de `max-width` en wrappers globales (ej: `.container { max-width: 1400px }`, `.wrapper { max-width: 1200px }`, o media queries que afecten el ancho máximo del contenido).
2. Si el HTML declara un valor, usarlo tal cual: no preguntar al usuario.
3. Si el HTML NO declara ningún `max-width` para el contenedor de contenido:
   - **Preguntar al usuario explícitamente**: "¿Quieres aplicar un `contentMaxWidth` a los rows para pantallas ultrawide? Opciones típicas: null (diseño 100% fluido), 1400px, 1600px, 1800px. Sugerencia: 1400px."
   - No aplicar default silencioso. Este token requiere confirmación explícita para evitar decisiones invisibles.

**D) `headerPadding` (nuevo en v1.3.0):**

El header/navbar tiene padding propio, distinto del `sectionPaddingHorizontal` y `sectionPaddingVertical` de contenido. Se procesa así:

1. Buscar en el HTML el `<header>` semántico (o clase equivalente `.header`, `.site-header`, `.main-header`, `.navbar`).
2. Si el header declara padding en el CSS, extraerlo directamente.
3. Si no lo declara, aplicar defaults más compactos que las sections de contenido:
   - desktop:    14/48/14/48 (vertical/horizontal más comprimido)
   - tablet:     12/32/12/32
   - phone:      10/20/10/20
4. Interpolar `tabletWide` y `phoneWide` con la misma cascada.
5. Estos valores son intencionalmente MÁS PEQUEÑOS que los del contenido — el header debe ser visualmente compacto para no restar altura al hero.

**Regla operativa:** el `headerPadding` NUNCA es igual al `sectionPadding`. Si al inferir del HTML resultan iguales, avisar al usuario y sugerir valores más compactos para el header.

### Nota importante sobre el sistema de contenedores

El `contentMaxWidth` funciona en tandem con el padding horizontal de sections. La lógica final es:

```
sections tienen padding horizontal uniforme → esto define el "margen interior" en cualquier viewport.
rows dentro tienen width: 100% + maxWidth: contentMaxWidth (o 100% si null).
  - Si viewport es más pequeño que contentMaxWidth: el row ocupa todo el ancho disponible.
  - Si viewport es más grande que contentMaxWidth: el row se limita al maxWidth y queda centrado.
```

Este sistema es la base de la "unificación de anchos" en todo el proyecto.

### Paso 4 — Confirmación con el usuario

Al terminar los 3 pasos anteriores, la Skill **presenta el manifiesto completo al usuario** con marca de origen de cada token:

```
=== Design Tokens — Manifiesto propuesto ===

Paleta de colores:
- Primario:     #3E0D61   [explicit]
- Secundario:   #541690   [extracted del HTML]
- Acento:       #1EDFAE   [explicit]
- Texto base:   #1A0037   [inferred - color oscuro derivado del primario]
- Fondo claro:  #F0FDFF   [extracted del HTML]
- Fondo oscuro: #1A0037   [inferred]

Tipografía:
- Familia headings:   Plus Jakarta Sans   [extracted del HTML]
- Familia body:       Inter               [extracted del HTML]
- Familia UI:         Inter               [inferred - mismo que body]

Escala tipográfica:
- H1: desktop 48px / phone 28px   [extracted]
- H2: desktop 36px / phone 24px   [inferred - interpolación de H1 y body]
- H3: desktop 24px / phone 20px   [inferred]
- Body: desktop 16px / phone 15px [extracted]

Spacing base: 8pt   [inferred - patrón detectado en el HTML]

Radios: sm 4px / md 8px / lg 16px / pill 999px   [inferred - estándar]

Sistema de contenedores:
- Padding horizontal sections:
    desktop 80px / tabletWide 80px / tablet 40px / phoneWide 24px / phone 20px   [extracted]
- Padding vertical sections:
    desktop 100px / tablet 72px / phone 56px   [extracted]
- contentMaxWidth (ancho máximo de rows en extrawide):
    1400px   [inferred - PREGUNTA AL USUARIO si no viene explícito]
- Padding del header (más compacto que sections):
    desktop 14/48 / tablet 12/32 / phone 10/20   [inferred - default]

¿Confirmas este manifiesto? Puedes:
1. Aceptar tal cual → escribe "OK" y continúa.
2. Ajustar tokens específicos → indica los cambios.
3. Cancelar y proporcionar un design-tokens.md completo → indica.
```

Los `[inferred]` se resaltan visualmente para revisión especial.

## Manifiesto confirmado como vinculante

Una vez el usuario confirma, el manifiesto queda **vinculante para todo el proyecto**. Se guarda como `projects/<nombre>/design-tokens.md` (si no existía) y el `divi-qa-validator` lo consulta para detectar desviaciones en el JSON emitido.

Si en una fase posterior el `divi-json-builder` detecta que necesita un valor fuera del manifiesto, debe:
1. Registrar la desviación en el log.
2. Preguntar al usuario si acepta la excepción o prefiere ajustar al token más cercano.

## Casos especiales

### Fuentes que no están en Google Fonts

Si una familia declarada no está en Google Fonts (ej: `Neue Haas Grotesk`, `Söhne`, fuentes propias del cliente), la Skill:

1. Avisa: "La fuente `<nombre>` no está en Google Fonts."
2. Explica el paso manual: subir el archivo `.woff2` al Divi Fonts Uploader antes de importar el JSON.
3. Emite el JSON con `font-family` referenciando el nombre exacto, para que Divi lo resuelva cuando encuentre la fuente en el uploader.

### Colores con nombres semánticos vs hex

El manifiesto puede usar nombres semánticos (`primario`, `acento`) o hex directos. En el JSON de Divi, siempre se emiten los hex/rgba resueltos, no los nombres.

### Modo oscuro

Fuera de v1. Si el HTML declara variantes dark mode (ej: clase `.dark` o `@media (prefers-color-scheme: dark)`), la Skill:
1. Ignora las variantes dark.
2. Avisa en `notes.md`: "El HTML declara modo oscuro. La Skill v1 no lo soporta. Configurar manualmente si es necesario."

## Tokens de formulario (nuevo en v1.3.0)

Estos tokens se aplican solo cuando el proyecto tiene formularios. En v1.3.0 se **infieren por defecto** de los tokens generales del proyecto. En una versión futura (v1.4.0+) podrán declararse explícitamente.

### Tokens inferidos por defecto

| Token | Inferencia |
|---|---|
| `tokens.form.background` | `tokens.color.fondoBase` con lightening del 5-8%, o color específico si el HTML lo declara. |
| `tokens.form.border` | Derivado del `textoBase` con opacidad 0.15 (patrón sutil). |
| `tokens.form.borderFocus` | `tokens.color.acento` directamente. |
| `tokens.form.text` | `tokens.color.textoBase`. |
| `tokens.form.placeholder` | `tokens.color.textoSecundario` con opacidad 0.6. |
| `tokens.form.label` | `tokens.color.textoSecundario`. |
| `tokens.form.radius` | `tokens.borderRadius.md`. |

### Tokens semánticos derivados

Estos también se infieren cuando no se declaran explícitamente:

| Token | Inferencia |
|---|---|
| `tokens.color.error` | Rojo semántico. Default: `#FF6B6B` (para diseños oscuros) o `#F71963` según intensidad del acento. |
| `tokens.color.success` | Verde semántico. Default: `#25D366`. |
| `tokens.color.textOnAcento` | Color de texto sobre el fondo del acento (calcular contraste WCAG AA). |
| `tokens.color.acentoHover` | Acento con lightening del 10% para hover states. |

### Ejemplo real (BKGlass)

Con estos tokens generales del proyecto:
- `tokens.color.fondoBase = #070F1C`
- `tokens.color.textoBase = #D8EEF7`
- `tokens.color.textoSecundario = #8CA0B8`
- `tokens.color.acento = #38C3FF`
- `tokens.borderRadius.md = 8px`

Se infieren estos tokens de formulario:
- `tokens.form.background = #152540` (fondoBase con lightening)
- `tokens.form.border = rgba(216, 238, 247, 0.15)` (textoBase con opacidad)
- `tokens.form.borderFocus = #38C3FF` (acento)
- `tokens.form.text = #D8EEF7` (textoBase)
- `tokens.form.placeholder = rgba(140, 160, 184, 0.6)` (textoSecundario 0.6)
- `tokens.form.label = #8CA0B8` (textoSecundario)
- `tokens.form.radius = 8px` (borderRadius.md)
- `tokens.color.error = #FF6B6B` (rojo semántico oscuro)
- `tokens.color.success = #25D366` (verde semántico)
- `tokens.color.textOnAcento = #070F1C` (fondoBase, alto contraste sobre acento)
- `tokens.color.acentoHover = #5BD0FF` (acento con lightening)

Estos valores se usan en la generación del CSS del formulario (ver `rules/cf7-form-generation.md`).

## Salida al finalizar la Fase 2

- **`projects/<nombre>/design-tokens.md`** — manifiesto confirmado (creado si no existía).
- **Log de tokens** copiado a `notes.md` con origen de cada token (`[explicit]`, `[extracted]`, `[inferred]`).
