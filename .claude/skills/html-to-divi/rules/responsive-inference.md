# Reglas de inferencia responsive — Divi 5.8.1

Este documento define cómo la Skill infiere breakpoints intermedios cuando el HTML de entrada no los declara explícitamente. La consulta el subagente `divi-json-builder` durante la Fase 3.

## Los 5 breakpoints de Divi 5.8.1

| Breakpoint | Rango de anchos | Uso típico |
|---|---|---|
| `desktop` | >= 1281px | Pantallas grandes. Es la línea base. |
| `tabletWide` | 981px a 1280px | Laptops pequeñas, tablets grandes. |
| `tablet` | 768px a 980px | Tablets estándar. |
| `phoneWide` | 480px a 767px | Móviles grandes, phablets. |
| `phone` | < 480px | Móviles estándar. |

## Cascada de herencia natural de Divi

Divi resuelve automáticamente los breakpoints no declarados así:

```
desktop → tabletWide → tablet → phoneWide → phone
```

Cada breakpoint hereda del inmediato superior si no se declara. Esto significa que si solo declaras `desktop` y `phone`, Divi aplica:
- `tabletWide` = desktop
- `tablet` = desktop
- `phoneWide` = phone

**Regla base:** no declarar un breakpoint es equivalente a heredar. Por lo tanto, **solo declarar breakpoints cuando el valor sea diferente al que se heredaría**.

## Escenarios comunes y cómo resolver

### Escenario A — HTML declara los 5 breakpoints explícitamente

Caso ideal. Se emiten los 5 tal como vienen. No hay inferencia.

Ejemplo (media queries en el HTML):

```css
.hero-title { font-size: 48px; }
@media (max-width: 1280px) { .hero-title { font-size: 40px; } }
@media (max-width: 980px)  { .hero-title { font-size: 32px; } }
@media (max-width: 767px)  { .hero-title { font-size: 28px; } }
@media (max-width: 479px)  { .hero-title { font-size: 24px; } }
```

Se emite:
```json
{
  "font": {
    "desktop":    { "value": { "size": "48px" } },
    "tabletWide": { "value": { "size": "40px" } },
    "tablet":     { "value": { "size": "32px" } },
    "phoneWide":  { "value": { "size": "28px" } },
    "phone":      { "value": { "size": "24px" } }
  }
}
```

### Escenario B — HTML declara solo desktop y phone

Caso más frecuente cuando el HTML viene de Figma con solo 2 vistas. La Skill infiere los intermedios.

Regla de inferencia:

1. **Calcular la ratio de cambio.** Si el valor cambia mucho (más de 40% de reducción), interpolar. Si cambia poco (menos de 20%), heredar de desktop.
2. **Interpolación tipográfica.** Usar escala modular. Ejemplo: desktop 48px → phone 24px. tabletWide ≈ 40px, tablet ≈ 34px, phoneWide ≈ 28px.
3. **Interpolación de spacing (padding, margin).** Si desktop 80px y phone 32px, tablet ≈ 56px, phoneWide ≈ 40px.
4. **Redondear a múltiplos de 4** para mantener el sistema de spacing coherente.
5. **Colores no se interpolan.** Se heredan tal cual (colores en tablet igual a desktop, salvo que se declare distinto).

Ejemplo aplicando la regla:

Entrada (solo dos breakpoints):
```css
.hero-title { font-size: 48px; padding: 80px 0; }
@media (max-width: 767px) { .hero-title { font-size: 24px; padding: 32px 0; } }
```

Salida inferida:
```json
{
  "font": {
    "desktop":    { "value": { "size": "48px" } },
    "tablet":     { "value": { "size": "32px" } },
    "phone":      { "value": { "size": "24px" } }
  },
  "spacing": {
    "desktop":    { "value": { "padding": { "top": "80px", "bottom": "80px" } } },
    "tablet":     { "value": { "padding": { "top": "56px", "bottom": "56px" } } },
    "phone":      { "value": { "padding": { "top": "32px", "bottom": "32px" } } }
  }
}
```

Nota: `tabletWide` y `phoneWide` no se emiten porque heredarían de desktop y tablet respectivamente sin cambios significativos.

### Escenario C — HTML declara solo desktop

Peor caso. La Skill infiere los otros 4 breakpoints aplicando buenas prácticas de sistemas de diseño.

Reglas conservadoras:

1. **Tamaños de heading grandes (>= 40px):** reducir 15% en tabletWide, 25% en tablet, 35% en phoneWide, 45% en phone.
2. **Tamaños de heading medianos (24-40px):** reducir 10% en tablet, 20% en phone.
3. **Tamaños de body text (14-18px):** no reducir. Se heredan tal cual.
4. **Padding vertical (top/bottom) grande (>= 80px):** reducir 30% en tablet, 60% en phone.
5. **Padding horizontal (left/right):** conservar en desktop/tabletWide/tablet, reducir 50% en phone si excede 32px.
6. **Columnas:** si en desktop hay >= 3 columnas, colapsar a 2 en tablet y 1 en phone.
7. **Colores:** no cambian.

Todas estas inferencias se registran en `notes.md` para que el usuario las revise y ajuste si no encajan con su diseño.

### Escenario D — HTML usa Tailwind con clases responsive

Detectar clases con prefijos: `sm:`, `md:`, `lg:`, `xl:`, `2xl:`.

Mapeo a los breakpoints de Divi:

| Tailwind | Ancho aproximado | Breakpoint Divi |
|---|---|---|
| base (sin prefijo) | < 640px | phone |
| `sm:` | >= 640px | phoneWide |
| `md:` | >= 768px | tablet |
| `lg:` | >= 1024px | tabletWide |
| `xl:` | >= 1280px | desktop |
| `2xl:` | >= 1536px | desktop (se agrega al override de desktop) |

**Importante:** Tailwind usa mobile-first (base = móvil, prefijos = desktop). Divi usa desktop-first (desktop = línea base, breakpoints = overrides). Al mapear, invertir la lógica:

Ejemplo Tailwind:
```html
<h1 class="text-2xl md:text-4xl lg:text-5xl xl:text-6xl">Título</h1>
```
- base (< 640px): 24px → phone
- md (>= 768px): 36px → tablet
- lg (>= 1024px): 48px → tabletWide
- xl (>= 1280px): 60px → desktop

Se emite:
```json
{
  "font": {
    "desktop":    { "value": { "size": "60px" } },
    "tabletWide": { "value": { "size": "48px" } },
    "tablet":     { "value": { "size": "36px" } },
    "phone":      { "value": { "size": "24px" } }
  }
}
```

`phoneWide` se omite porque hereda de `tablet` sin cambios claros.

## Reglas específicas de layout responsive

### Columnas colapsando en móvil

Cuando en desktop hay `1_3, 1_3, 1_3` (3 columnas), típicamente en móvil deben apilarse. La Skill emite:

```json
{
  "decoration": {
    "sizing": {
      "desktop":    { "value": { "flexType": "8_24" } },
      "tablet":     { "value": { "flexType": "12_24" } },
      "phone":      { "value": { "flexType": "24_24" } }
    }
  }
}
```

Es decir: 3 col en desktop, 2 col en tablet, 1 col en phone.

Aplica también a:
- 4 columnas desktop → 2 columnas tablet → 1 columna phone.
- 5-6 columnas desktop → 3 columnas tablet → 2 columnas phoneWide → 1 columna phone.

### `flexWrap` en rows

Al colapsar columnas, el `row` debe permitir wrap:

```json
{
  "decoration": {
    "layout": {
      "desktop":    { "value": { "flexWrap": "nowrap" } },
      "tablet":     { "value": { "flexWrap": "wrap" } },
      "phone":      { "value": { "flexWrap": "wrap" } }
    }
  }
}
```

### Direcciones de flex

Si el HTML tiene `flex-direction: row` en desktop y `column` en móvil (patrón común), emitir:

```json
{
  "decoration": {
    "layout": {
      "desktop":    { "value": { "flexDirection": "row" } },
      "phone":      { "value": { "flexDirection": "column" } }
    }
  }
}
```

## Cuándo NO inferir (preguntar al usuario)

- Cuando el HTML tiene un layout complejo tipo grid con `grid-template-areas` que cambia entre breakpoints.
- Cuando hay overrides muy específicos de posicionamiento absoluto o transform en un breakpoint intermedio.
- Cuando el usuario declaró en `design-tokens.md` reglas responsive personalizadas.

En estos casos, el subagente pregunta al usuario cómo interpretar el layout en los breakpoints no declarados antes de emitir.

## Log de inferencias

Toda inferencia aplicada se registra en el log interno del `divi-json-builder`, que la Skill copia a `notes.md`:

```
=== Responsive Inference Log ===

- Section "Hero" padding: HTML declara 80px (desktop) y 32px (phone).
  Inferido tablet: 56px. tabletWide y phoneWide heredan.

- H1 "Impulsa tu presencia" font-size: HTML declara 48px (desktop).
  Sin declaración móvil. Inferido tablet: 36px, phone: 28px.

- Row "Servicios grid": HTML tiene 3 columnas en desktop.
  Inferido tablet: 2 columnas (flexType 12_24). Phone: 1 columna (24_24).
```
