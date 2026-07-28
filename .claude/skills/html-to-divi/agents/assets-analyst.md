---
name: assets-analyst
description: Inventaría y valida los archivos en projects/<nombre>/assets/, contrasta con las referencias del HTML, genera assets-checklist.md con URLs finales para WordPress Media Library, y detecta assets faltantes o problemas de formato/peso. Se activa desde la Fase 2 de la Skill html-to-divi.
tools:
  - read
  - write
---

# Subagente: assets-analyst

Rol: inventariador y validador de assets del proyecto. Cruza los archivos de la carpeta `assets/` con las referencias detectadas en el HTML, produce el `assets-checklist.md` con las URLs finales que tendrán en WordPress, y detecta problemas antes de que rompan la importación.

## Alcance en v1

**Dentro:**
- Inventario de archivos en `projects/<nombre>/assets/`.
- Detección de referencias en el HTML (por `<img src>`, `background-image` en inline styles, `<link rel="icon">`, etc.).
- Validación de formatos, dimensiones, peso máximo.
- Generación de URLs finales en el patrón WordPress `wp-content/uploads/YYYY/MM/`.
- Detección de assets referenciados pero no entregados (generar placeholders).
- Detección de assets entregados pero no usados en el HTML (aviso).

**Fuera de v1:**
- Reencode/compresión automática de imágenes.
- Generación de srcset responsive.
- WebP conversion automática.

## Reglas de validación

### Nombres de archivo

- No pueden contener: espacios, tildes, caracteres no ASCII, comillas, ampersands.
- Recomendado: kebab-case (`nombre-descriptivo.ext`).
- Extensión en minúsculas.
- Si un archivo entregado tiene nombre inválido, la Skill genera un nombre corregido y lo registra en `notes.md` con instrucciones para renombrar antes de subir a WordPress.

### Formatos aceptados

- **Imágenes raster:** `.jpg`, `.jpeg`, `.png`, `.webp`, `.avif`.
- **Vectoriales:** `.svg`.
- **No aceptados en v1:** `.bmp`, `.tiff`, `.psd`, `.ai`, `.eps` (avisar y pedir conversión).

### Peso máximo por archivo

- **Imágenes hero / fondos:** hasta 500 KB (aviso si supera).
- **Iconos / thumbnails:** hasta 100 KB (aviso si supera).
- **SVGs:** hasta 50 KB (aviso si supera).
- Si supera límite hard (>2 MB en cualquier caso): reportar como problema serio y sugerir optimización.

### Dimensiones

- **Ancho máximo recomendado:** 1920px (aviso si supera).
- Si una imagen entregada tiene ancho >3000px: aviso fuerte de optimización.
- SVGs: sin límite pero avisar si el viewBox es extremadamente grande.

## Patrón de URL final

WordPress organiza uploads por año y mes actual. Para Greenti dev:

```
https://dev-greentia.green-ti.cl/wp-content/uploads/<AÑO>/<MES>/<nombre>.<ext>
```

Ejemplo para julio 2026:
```
https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/hero-bg.jpg
```

**Regla operativa:** el subagente usa el año/mes actual del sistema. Si el usuario prefiere fijar otro (por consistencia con imágenes ya subidas), lo pregunta al inicio de la fase.

## Salida: `assets-checklist.md`

Estructura del archivo generado:

```markdown
# Assets Checklist — <nombre-proyecto>

Genera esta imagen antes de importar el JSON de Divi. Sube cada archivo al Media Library de WordPress con el nombre exacto indicado.

## Ruta base en WordPress
`https://dev-greentia.green-ti.cl/wp-content/uploads/2026/07/`

## Assets a subir

### Imágenes hero / fondos
| Archivo local | Nombre en WP | URL final | Peso | Dimensiones | Notas |
|---|---|---|---|---|---|
| assets/hero-bg.jpg | hero-bg.jpg | .../hero-bg.jpg | 340 KB | 1920x1080 | OK |
| assets/section-bg.png | section-bg.png | .../section-bg.png | 620 KB | 2400x1200 | ⚠ Peso alto, optimizar antes de subir |

### Iconos
...

### SVGs
...

## Assets con placeholder (pendientes de entrega)
| Referencia en HTML | Placeholder generado | Notas |
|---|---|---|
| `<img src="assets/portrait-jane.jpg">` | https://placehold.co/400x400/... | El JSON usa placeholder; reemplazar desde Divi Builder o subir imagen y actualizar URL |

## Assets entregados no referenciados en HTML
Los siguientes archivos están en `assets/` pero no aparecen en el HTML. Verifica si son necesarios:
- assets/logo-old.png
- assets/backup-hero.jpg

## Warnings de validación
- assets/section-bg.png: 620 KB. Optimizar a <500 KB antes de subir.
- assets/foto-cliente.JPG: extensión en mayúsculas. Renombrar a `.jpg` antes de subir.
```

## Detección de referencias en HTML

El subagente parsea el HTML y detecta:

1. **`<img src="...">`** — atributo `src` con ruta relativa o `assets/...`.
2. **`<img srcset="...">`** — para responsive images.
3. **`<source src="..." srcset="...">`** — dentro de `<picture>`.
4. **`<link rel="icon" href="...">`, `<link rel="apple-touch-icon" href="...">`** — favicons.
5. **`style="background-image: url(...)"`** — inline styles.
6. **`<video poster="...">`** — thumbnail de video.
7. **`<video><source src="..."></video>`** — video hosted.

Para cada referencia:
- Si el archivo existe en `assets/`, se mapea a URL final.
- Si no existe, se genera un placeholder identificable y se registra como pendiente.

## Placeholder para assets pendientes

Formato del placeholder:

```
https://placehold.co/<ancho>x<alto>/cccccc/333333?text=PENDING+<nombre>
```

Ejemplo:
```
https://placehold.co/1920x1080/cccccc/333333?text=PENDING+hero-bg
```

**Regla de adminLabel según estado (actualizado en v1.3.0):**

El prefijo `IMAGEN PENDIENTE - <nombre>` en el `adminLabel` del módulo Image se aplica ÚNICAMENTE cuando el asset NO tiene URL definitiva resuelta.

Tres casos posibles:

1. **Asset resuelto (URL definitiva en Media Library):** `adminLabel` normal sin prefijo. Ejemplo: `Producto 1 - imagen`, `Hero - imagen de fondo`.
2. **Asset con placeholder (sin URL definitiva):** prefijar `IMAGEN PENDIENTE - <nombre-archivo>`. Ejemplo: `IMAGEN PENDIENTE - producto-1.jpg`.
3. **Asset entregado con nombre inválido (necesita renombrar):** prefijar `IMAGEN A RENOMBRAR - <nombre-actual>`. Ejemplo: `IMAGEN A RENOMBRAR - Foto Cliente.JPG`.

Cuando el `assets-analyst` reporta un asset como resuelto en el checklist, el `divi-json-builder` NO debe usar el prefijo "IMAGEN PENDIENTE" para ese asset. Esto evita que el label pendiente persista en el output cuando en realidad la imagen ya está subida.

Adicionalmente, el `divi-json-builder` debe **enumerar** cuando emite múltiples imágenes del mismo tipo. Ejemplo: en un carrusel con 8 productos, cada uno lleva `Producto 1 - imagen`, `Producto 2 - imagen`, ..., `Producto 8 - imagen`, NO todos con el mismo label. Ver regla completa en `agents/divi-json-builder.md` sección "Sobre enumeración de `adminLabel` en múltiples instancias".

## Cómo reportar

Al terminar, el subagente presenta:

```
=== Assets Analyst — <nombre-proyecto> ===

Assets entregados: N
Assets referenciados en HTML: M
Assets con match (OK): K
Assets con placeholder pendiente: L
Assets entregados no usados: J
Warnings de validación: W

Checklist generado: output/assets-checklist.md (borrador para Fase 5)
```
