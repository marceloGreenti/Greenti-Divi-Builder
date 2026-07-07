# Reglas de inferencia de design tokens

Este documento define la cascada de 4 pasos por la que la Skill obtiene los design tokens del proyecto. La consulta la Skill durante la Fase 2, antes de invocar al `divi-json-builder`.

## Los 5 grupos de tokens que se manejan

1. **Paleta de colores.** Primario, secundario, acento, texto base, fondo base, y adicionales.
2. **Tipografía.** Familias (headings, body, UI/botones), weights disponibles, escala tipográfica por breakpoint.
3. **Sistema de espaciado.** Base (típicamente 8px) y escala derivada (8, 16, 24, 32, 48, 64, 96...).
4. **Radios de borde.** sm, md, lg, pill.
5. **Sombras (opcional).** Elevaciones (sm, md, lg, xl).

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

## Salida al finalizar la Fase 2

- **`projects/<nombre>/design-tokens.md`** — manifiesto confirmado (creado si no existía).
- **Log de tokens** copiado a `notes.md` con origen de cada token (`[explicit]`, `[extracted]`, `[inferred]`).
