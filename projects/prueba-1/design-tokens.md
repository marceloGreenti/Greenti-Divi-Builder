# Design Tokens — prueba-1 (BKGlass)

> Manifiesto **vinculante** para el proyecto. Inferido desde `html/bkglass_home_responsive.html`
> (no existía `design-tokens.md` previo). Tema **oscuro**. Fuente única: **DM Sans**.
> Origen de cada token marcado como `[extracted]` (leído del HTML) o `[inferred]` (heurística/decisión).
>
> Decisiones confirmadas con el usuario (2026-07-06):
> - Títulos unificados a **DM Sans 500** (se elimina la fuente Syne).
> - Números display (stats/pasos) y logo → **DM Sans 700** para conservar jerarquía visual.
> - **Sin sombras** (el tema oscuro usa bordes + tintes translúcidos).
> - Sin assets: las imágenes se emiten como **placeholders**.

## Paleta de colores (tema oscuro)

| Rol | Token | Hex / valor | Origen | Uso |
|-----|-------|-------------|--------|-----|
| Acento / CTA primario | `--color-accent` | `#38C3FF` | [extracted] | Botones, links activos, íconos, bordes destacados, texto de marca |
| Fondo base (más oscuro) | `--color-bg-base` | `#070F1C` | [extracted] | Body; secciones ventajas/proceso/contacto; texto sobre acento |
| Fondo secundario | `--color-bg-secondary` | `#0B1A2E` | [extracted] | Navbar, hero, secciones alternas, footer, formulario |
| Fondo elevado (cards) | `--color-bg-elevated` | `#152540` | [extracted] | Cards de producto, inputs, thumbs |
| Fondo placeholder imagen | `--color-bg-placeholder` | `#1E3252` | [extracted] | Áreas de imagen (se reemplaza por imagen real) |
| Texto claro (headings) | `--color-text-strong` | `#FFFFFF` | [extracted] | Títulos |
| Texto base (body) | `--color-text-base` | `#D8EEF7` | [extracted] | Color de texto por defecto |
| Texto atenuado (muted) | `--color-text-muted` | `#8CA0B8` | [extracted] | Párrafos secundarios, labels, links de navegación |
| Tinte de borde/overlay | `--color-border-tint` | `rgba(168,212,232, α)` | [extracted] | Bordes sutiles y fondos translúcidos. α: 0.05 / 0.08 / 0.10 / 0.12 / 0.15 |
| Tinte de acento | `--color-accent-tint` | `rgba(56,195,255, α)` | [extracted] | Fondos de íconos y bordes destacados. α: 0.10 / 0.12 / 0.20 / 0.30 / 0.35 |
| WhatsApp (fijo marca) | `--color-whatsapp` | `#25D366` | [extracted] | Exclusivo del botón de WhatsApp |

## Tipografía

Fuente única: **DM Sans** (Google Fonts). Weights: **300, 500, 700**.

```
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;500;700&display=swap" rel="stylesheet" />
```

| Rol | Familia | Weight | Origen |
|-----|---------|--------|--------|
| Títulos (H1, H2, H3) | DM Sans | **500** | [inferred - unificación pedida por el usuario; original Syne 800] |
| Display (números stats/pasos, logo) | DM Sans | **700** | [inferred - conserva jerarquía del original Syne 800] |
| Body / descripciones | DM Sans | **300** | [extracted] |
| UI / labels / botones | DM Sans | **500** | [extracted] |

## Escala tipográfica (desktop / tablet / phone)

| Rol | Desktop | Tablet | Phone | Line-height | Letter-spacing | Origen |
|-----|--------:|-------:|------:|:-----------:|:--------------:|--------|
| H1 (hero) | 56px | 44px | 34px | 1.05 | -0.5px | [extracted] |
| H2 (sección) | 40px | 36px | 30px | 1.15 | 0 | [extracted] (contacto 36px; tablet interpolado [inferred]) |
| H3 (cards) | 18px | 18px | 18px | 1.3 | 0 | [extracted] (17px ventajas/proceso; 20px título form) |
| Body / desc | 16px | 16px | 14px | 1.7 | 0 | [extracted] |
| Subtítulo | 16px | 16px | 14px | 1.7 | 0 | [extracted] |
| Small | 14px | 14px | 13px | 1.6 | 0 | [extracted] |
| Caption / list | 13px | 13px | 13px | 1.6 | 0 | [extracted] |
| Eyebrow | 11px | 11px | 10px | 1.4 | 2.5px (uppercase) | [extracted] |
| Micro-label | 12px | 12px | 12px | 1.4 | 0.5–1.5px | [extracted] |
| Stat número (display) | 32px | 32px | 32px | 1.0 | 0 | [extracted] |
| Paso número (display) | 40px | 40px | 40px | 1.0 | 0 | [extracted] |

## Sistema de espaciado

Base **4pt** `[inferred - patrón de múltiplos de 4/8 en el HTML]`.
Escala usada: `4, 6, 8, 10, 12, 14, 16, 20, 24, 28, 32, 36, 40, 52, 64, 80, 100`.

Padding de secciones (vertical / horizontal):

| Breakpoint | Padding | Origen |
|-----------|---------|--------|
| Desktop | 100px / 80px | [extracted] |
| Tablet (≤980px) | 72px / 40px | [extracted] |
| Phone (≤767px) | 56px / 20px | [extracted] |

Gaps de grid: `24px` (cards), `16px` (galería), `12px` (thumbs).

## Sistema de contenedores unificado (Skill v1.1.0)

Aplicado a **todas** las sections y rows para garantizar alineación consistente en todo el sitio.

| Token | Desktop | Tablet (≤980px) | Phone (≤767px) | Origen |
|-------|---------|-----------------|----------------|--------|
| `sectionPaddingHorizontal` | 80px | 40px | 20px | [extracted] — padding lateral idéntico en todas las sections |
| `sectionPaddingVertical` | 100px | 72px | 56px | [extracted] — hero usa 80/64/48 propio |
| `contentMaxWidth` | **null (100% fluido)** | — | — | [confirmed] — el usuario eligió diseño 100% fluido (2026-07-08); el HTML no declaraba tope |

- **Rows:** todos emiten `sizing.width: "100%"` + `sizing.maxWidth: "100%"` (nunca sin declarar → evita el default 1080px de Divi que rompe la unificación).
- **`contentMaxWidth = null`** → el contenido siempre ocupa el ancho disponible menos los 80px de padding lateral; no hay tope en pantallas ultrawide.

## Radios de borde

| Token | Valor | Origen | Uso |
|-------|-------|--------|-----|
| `--radius-sm` | 6px | [extracted] | Badges, botones |
| `--radius-md` | 8px | [extracted] | Inputs, cajas de ícono, botón WhatsApp |
| `--radius-lg` | 12px | [extracted] | Cards, fotos de galería (10px→normalizado a 12) |
| `--radius-xl` | 16px | [extracted] | Contenedor del formulario |
| `--radius-badge` | 4px | [extracted] | Etiquetas "Más popular", captions |

## Sombras

**Ninguna.** `[extracted — decisión confirmada: el tema oscuro usa bordes + tintes translúcidos en vez de sombras]`

Único efecto de profundidad: `backdrop-filter: blur(4px)` en los captions de la galería `[extracted]`.

## Bordes (patrón recurrente)

| Uso | Valor | Origen |
|-----|-------|--------|
| Borde sutil (cards, secciones, inputs) | `1px solid rgba(168,212,232, 0.08–0.15)` | [extracted] |
| Borde destacado (card popular, thumb activo) | `1px solid rgba(56,195,255, 0.30–0.35)` | [extracted] |
| Borde dashed (dropzone archivos) | `1.5px dashed rgba(168,212,232, 0.20)` | [extracted] |
