---
name: html-to-divi
description: Convierte un HTML maquetado + assets + design tokens en un JSON de Divi 5.8.1 importable a WordPress vía Divi Library, respetando estilos exactos, generando header/footer separados para Theme Builder, y avisando de patrones que requieran Code Module. Se activa cuando el usuario menciona convertir HTML a Divi, generar JSON para Divi, procesar una maqueta a Divi, o cuando trabaja en una carpeta bajo projects/ con html/ + assets/ + design-tokens.md.
---

# Skill: html-to-divi

Convierte un HTML de maqueta en un JSON importable en Divi 5.8.1, con salidas separadas para página, header y footer, más archivos compañeros de SEO, performance, assets y notas.

## Cuándo activarse

Esta Skill se activa cuando el usuario:
- Pide "generar JSON de Divi" para una carpeta bajo `projects/`.
- Pide "convertir un HTML a Divi" o "pasar una maqueta a Divi".
- Trabaja dentro de una carpeta que contenga la estructura `html/` + `assets/` + opcionalmente `design-tokens.md`.
- Menciona explícitamente "usar la skill html-to-divi".

## Contrato de entrada

La Skill espera una carpeta de proyecto con la siguiente estructura:

```
projects/<nombre-proyecto>/
├── html/
│   └── *.html               ← al menos un archivo HTML maquetado
├── assets/                  ← carpeta con imágenes, íconos, SVGs (opcional)
│   └── ...
└── design-tokens.md         ← manifiesto de tokens (opcional)
```

**Requisitos:**
- Al menos un archivo HTML en `html/`. Si hay múltiples, se procesan de a uno o el usuario elige.
- `assets/` puede estar vacía o no existir; en ese caso se generan placeholders.
- `design-tokens.md` es opcional; si no existe, la Skill aplica la cascada de inferencia (ver `rules/design-tokens-inference.md`).

## Contrato de salida

Al finalizar, la Skill genera en `projects/<nombre-proyecto>/output/`:

1. **`divi-import-page.json`** — contenido de la página sin header/footer, importable en Divi Library.
2. **`divi-import-header.json`** — si el HTML incluye `<header>`. Para Theme Builder → Global Header.
3. **`divi-import-footer.json`** — si el HTML incluye `<footer>`. Para Theme Builder → Global Footer.
4. **`seo-meta.md`** — metadatos on-page (title, description, OG, Twitter, canonical, schema.org) para copiar a Rank Math/Yoast.
5. **`performance-checklist.md`** — checklist de Core Web Vitals validado.
6. **`assets-checklist.md`** — inventario de imágenes con URLs finales para subir al Media Library.
7. **`notes.md`** — avisos importantes, Code Modules pendientes, decisiones tomadas, design tokens usados y su origen.
8. **`html/landing-corrected.html`** (dentro de la carpeta html/) — HTML corregido por el `seo-auditor` con las correcciones aplicadas.

## Fases de ejecución

La Skill orquesta el trabajo en **5 fases secuenciales**. En cada una invoca uno o más subagentes especializados. Los subagentes viven en `.claude/skills/html-to-divi/agents/` y las reglas específicas en `.claude/skills/html-to-divi/rules/`.

### Fase 1 — Ingesta

Objetivo: recoger inputs y verificar que estén disponibles.

Pasos:
1. Leer los archivos de `projects/<nombre-proyecto>/html/`. Si hay más de un HTML, preguntar al usuario cuál procesar (o procesar todos).
2. Leer la carpeta `assets/` si existe. Guardar la lista de archivos disponibles.
3. Leer `design-tokens.md` si existe. Si no, marcar como "pendiente de inferencia".
4. Confirmar al usuario los inputs recogidos antes de continuar.

Si falta el HTML → cortar y pedir. Si faltan assets o tokens → continuar; se resolverán en fases posteriores.

### Fase 2 — Validación y corrección de entrada

Objetivo: dejar el HTML limpio, semánticamente correcto, con SEO y a11y básica cubiertas; e inventariar assets.

Se invocan dos subagentes en paralelo:

**Subagente `seo-auditor`** (`agents/seo-auditor.md`):
- Audita el HTML según SEO técnico + a11y básica.
- **Aplica correcciones automáticas** (auto-corregibles sin decisión de negocio).
- Pregunta al usuario cuando la corrección requiere decisión (ej: alt text descriptivo faltante).
- Corta y reporta si hay errores estructurales bloqueantes.
- Emite `html/landing-corrected.html` con las correcciones aplicadas.
- Prepara el borrador de `seo-meta.md` (se completa en Fase 5).

**Subagente `assets-analyst`** (`agents/assets-analyst.md`):
- Inventaría archivos de `assets/`.
- Valida formatos, dimensiones, pesos.
- Detecta assets referenciados en el HTML que faltan en la carpeta.
- Prepara el borrador de `assets-checklist.md` con URLs finales de WordPress.

Al final de esta fase, si el HTML no tiene tokens de diseño explícitos y `design-tokens.md` no existe, la Skill aplica la cascada de inferencia (ver `rules/design-tokens-inference.md`) y **presenta al usuario el manifiesto de tokens** con marca de origen (`[explicit]`, `[extracted]`, `[inferred]`) para confirmación antes de continuar.

### Fase 3 — Emisión del JSON

Objetivo: construir el JSON de Divi respetando el schema del reference doc.

Se invoca:

**Subagente `divi-json-builder`** (`agents/divi-json-builder.md`):
- Toma el HTML corregido + tokens confirmados + inventario de assets.
- Aplica las reglas de mapeo HTML → Divi (ver `rules/html-to-divi-mapping.md`).
- Detecta patrones que requieran Code Module (ver `rules/code-module-triggers.md`) y los emite como tal, marcados en `adminLabel`.
- Separa header y footer del contenido principal.
- Aplica reglas de responsive con inferencia de breakpoints intermedios (ver `rules/responsive-inference.md`).
- Emite los archivos JSON borrador (`divi-import-page.json`, y opcionalmente `divi-import-header.json` y `divi-import-footer.json`).
- Registra en un log interno las decisiones tomadas (qué patrón produjo qué módulo, qué se marcó como Code Module, qué breakpoints se infirieron).

### Fase 4 — Validación de salida

Objetivo: verificar que el JSON emitido sea correcto e importable.

Se invoca:

**Subagente `divi-qa-validator`** (`agents/divi-qa-validator.md`):
- Parsea los JSON emitidos y verifica que sean JSON válido.
- Verifica jerarquía correcta (`placeholder → section → row → column → módulo`).
- Verifica que todo `column` tenga `advanced.type` y `flexType`.
- Verifica que las imágenes referenciadas estén en el `assets-checklist.md`.
- Verifica que se respeten los design tokens confirmados (colores, fuentes, spacing).
- Verifica que header/footer estén correctamente separados.
- Verifica que `builderVersion` sea `"5.8.1"` en todos los bloques.
- Si algo falla, corta y reporta al usuario. La Skill puede pedir a `divi-json-builder` reintentar con las correcciones sugeridas.

### Fase 5 — Empaquetado

Objetivo: escribir todos los archivos de salida en `output/` y presentar el resumen ejecutivo al usuario.

Pasos:
1. Escribir en `projects/<nombre-proyecto>/output/`:
   - Los JSON emitidos.
   - `seo-meta.md` final (completado a partir del borrador de Fase 2).
   - `performance-checklist.md`.
   - `assets-checklist.md` final.
   - `notes.md` estructurado para **lectura rápida** con:
     - Resumen ejecutivo (10 líneas máximo al inicio).
     - Acciones requeridas antes de importar (lista corta accionable).
     - Design tokens aplicados con origen.
     - Code Modules emitidos (1 párrafo por cada uno).
     - Warnings a revisar.
     - Referencia a `notes-detail.md` para el log completo.
   - `notes-detail.md` (opcional, si el log excede ~40 items) con logs completos de mapeo, inferencias responsive, sanitización, etc.
2. Presentar al usuario un **resumen ejecutivo** con:
   - Archivos generados y sus rutas.
   - Cantidad de módulos emitidos.
   - Cantidad de Code Modules generados.
   - Cantidad de assets a subir manualmente.
   - Warnings importantes.
3. Recordar el siguiente paso: subir assets al Media Library, importar los JSON en Divi Library / Theme Builder, copiar SEO meta a Rank Math/Yoast.

## Reglas transversales

- **Fidelidad de estilos.** La Skill nunca inventa valores. Si un valor no viene en el HTML o los tokens, aplica la cascada de inferencia y confirma con el usuario.
- **builderVersion fijo.** Siempre `"5.8.1"` en cada bloque emitido.
- **5 breakpoints Divi 5.** desktop, tabletWide, tablet, phoneWide, phone. Ver `rules/responsive-inference.md` para inferencia.
- **Header y footer separados.** Detectar por `<header>` / `<footer>` semánticos o clases equivalentes.
- **CF7 como Code Module placeholder.** Cuando el HTML tenga un `<form>`, emitir Code Module con `[contact-form-7 id="INSERTA_ID_AQUI" title="Inserta shortcode del formulario aquí"]`.
- **Idempotencia.** Regenerar la Skill sobre el mismo input debe producir el mismo output.
- **Sanitización.** Limpiar clases/IDs del HTML que colisionen con las que Divi genera (`et_pb_*`).
- **Sección B / módulos pendientes.** Si el HTML requiere un módulo aún no catalogado en `divi5-reference.md`, cortar y pedir al usuario que exporte un ejemplo mínimo desde su Divi para completar el catálogo antes de continuar.

## Referencias internas

- `divi5-reference.md` — schema completo de Divi 5.8.1 y catálogo de 63 módulos. Fuente de verdad técnica.
- `agents/seo-auditor.md` — auditor SEO/a11y activo con corrección de HTML.
- `agents/assets-analyst.md` — inventario y validación de assets.
- `agents/divi-json-builder.md` — emisor del JSON.
- `agents/divi-qa-validator.md` — QA de salida.
- `rules/responsive-inference.md` — cascada de inferencia entre 5 breakpoints.
- `rules/design-tokens-inference.md` — cascada de 4 pasos para tokens.
- `rules/html-to-divi-mapping.md` — tabla de mapeo HTML → módulos Divi.
- `rules/code-module-triggers.md` — patrones que fuerzan Code Module.
