# Changelog

Todas las mejoras notables a la Skill `html-to-divi` se documentan aquí.

Se sigue el formato de [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) y versionado [Semver](https://semver.org/lang/es/).




## [v1.2.0] - 2026-07-24

### Agregado
- **Elección de gestión de formularios**: la Skill ahora pregunta en Fase 2 si el proyecto usa Contact Form 7 o Divi Form nativo. La decisión se aplica a todos los formularios del proyecto.
- **Generación automática de configuración CF7**: cuando se elige CF7, la Skill genera dos archivos compañeros:
  - `output/cf7-form-config.md` — configuración lista para pegar en CF7 admin (Form, Mail, Messages, Additional Settings).
  - `output/cf7-form-styles.css` — CSS custom para pegar en Divi → Opciones → CSS personalizado, respetando design tokens.
- **Soporte completo para Divi Form nativo**: cuando se elige Divi Form, la Skill mapea los campos HTML al módulo `contact-form` con cobertura 80-95% + `output/divi-form-styles.css` complementario.
- **Nuevo archivo de reglas**: `rules/cf7-form-generation.md` con toda la sintaxis CF7 y plantillas.
- **Detección estructurada de formularios**: el `seo-auditor` ahora genera un inventario de formularios (ubicación, campos, tipos, botón) que alimenta la decisión CF7/Divi Form.

### Corregido
- **`group-carousel` mostraba 1 solo elemento visible** (bug crítico): el schema documentado era incompleto. Ahora el reference doc incluye 5 grupos (`module`, `arrows`, `dotNav`, `children`, `activeGroups`) y 10+ propiedades nuevas (`slidesPerView` decimal, `slidesToShow`, `centerMode`, `auto`, `speed`, `transitionSpeed`).
- Nueva regla de inferencia de `slidesPerView`: se detecta del CSS del HTML, y si no se puede inferir, la Skill pregunta al usuario cuántos elementos deben verse simultáneamente por breakpoint.
- Regla de coherencia para autoplay: `carousel.autoplay` y `advanced.auto` deben emitirse ambos con el mismo estado.

### Actualizado
- `divi5-reference.md`: schema completo del `group-carousel` con ejemplo real de configuración con 4.5 slides visibles en desktop.
- `rules/html-to-divi-mapping.md`: tabla detallada de mapeo HTML `<input>/<textarea>/<select>` → `contact-field.fieldType` de Divi Form. Sección "Carruseles" separada con reglas críticas de `slidesPerView`.
- `rules/code-module-triggers.md`: trigger de formularios ahora es condicional a la elección del usuario en Fase 2.
- `agents/divi-json-builder.md`: consulta ahora también `rules/cf7-form-generation.md`. Nuevas reglas de emisión para formularios y carruseles.




## [v1.2.0] - 2026-07-10

### Agregado
- **Elección de gestión de formularios**: la Skill ahora pregunta en Fase 2 si el proyecto usa Contact Form 7 o Divi Form nativo. La decisión se aplica a todos los formularios del proyecto.
- **Generación automática de configuración CF7**: cuando se elige CF7, la Skill genera dos archivos compañeros:
  - `output/cf7-form-config.md` — configuración lista para pegar en CF7 admin (Form, Mail, Messages, Additional Settings).
  - `output/cf7-form-styles.css` — CSS custom para pegar en Divi → Opciones → CSS personalizado, respetando design tokens.
- **Soporte completo para Divi Form nativo**: cuando se elige Divi Form, la Skill mapea los campos HTML al módulo `contact-form` con cobertura 80-95% + `output/divi-form-styles.css` complementario.
- **Nuevo archivo de reglas**: `rules/cf7-form-generation.md` con toda la sintaxis CF7 y plantillas.
- **Detección estructurada de formularios**: el `seo-auditor` ahora genera un inventario de formularios (ubicación, campos, tipos, botón) que alimenta la decisión CF7/Divi Form.

### Corregido
- **`group-carousel` mostraba 1 solo elemento visible** (bug crítico): el schema documentado era incompleto. Ahora el reference doc incluye 5 grupos (`module`, `arrows`, `dotNav`, `children`, `activeGroups`) y 10+ propiedades nuevas (`slidesPerView` decimal, `slidesToShow`, `centerMode`, `auto`, `speed`, `transitionSpeed`).
- Nueva regla de inferencia de `slidesPerView`: se detecta del CSS del HTML, y si no se puede inferir, la Skill pregunta al usuario cuántos elementos deben verse simultáneamente por breakpoint.
- Regla de coherencia para autoplay: `carousel.autoplay` y `advanced.auto` deben emitirse ambos con el mismo estado.

### Actualizado
- `divi5-reference.md`: schema completo del `group-carousel` con ejemplo real de configuración con 4.5 slides visibles en desktop.
- `rules/html-to-divi-mapping.md`: tabla detallada de mapeo HTML `<input>/<textarea>/<select>` → `contact-field.fieldType` de Divi Form. Sección "Carruseles" separada con reglas críticas de `slidesPerView`.
- `rules/code-module-triggers.md`: trigger de formularios ahora es condicional a la elección del usuario en Fase 2.
- `agents/divi-json-builder.md`: consulta ahora también `rules/cf7-form-generation.md`. Nuevas reglas de emisión para formularios y carruseles.




## [v1.1.0] - 2026-07-08

### Agregado
- Sistema de contenedores unificado como parte de los design tokens del proyecto.
  - Nuevo token `sectionPaddingHorizontal` (por breakpoint).
  - Nuevo token `sectionPaddingVertical` (por breakpoint).
  - Nuevo token `contentMaxWidth` (opcional, para pantallas ultrawide).
- Fase 2 de la Skill ahora confirma explícitamente el sistema de contenedores antes de emitir el JSON.
- `contentMaxWidth`: si el HTML no lo declara, la Skill pregunta explícitamente al usuario en vez de aplicar default silencioso.
- Nueva categoría de validación 6b en `divi-qa-validator`: verifica consistencia de padding entre sections y sizing explícito en rows.

### Corregido
- Rows ya no se emiten sin `sizing.width` y `sizing.maxWidth` (antes causaba que Divi aplicara default 1080px rompiendo la unificación visual).
- Sections del proyecto ahora respetan el mismo padding horizontal por breakpoint (antes había variaciones que causaban desalineación visual entre secciones).




## [v1.0.0] - 2026-07-06

### Agregado
- Skill `html-to-divi` inicial con orquestador de 5 fases.
- `divi5-reference.md` v1.1 con 63 módulos documentados de Divi 5.8.1.
- 4 subagentes: `seo-auditor`, `assets-analyst`, `divi-json-builder`, `divi-qa-validator`.
- 4 archivos de reglas: responsive-inference, design-tokens-inference, html-to-divi-mapping, code-module-triggers.
- Soporte para 5 breakpoints Divi 5 (`desktop`, `tabletWide`, `tablet`, `phoneWide`, `phone`).
- Cascada de inferencia de design tokens (4 pasos).
- Detección automática y separación de header/footer para Theme Builder.
- Emisión de Code Module con placeholder CF7 al detectar formularios.
- Validación WCAG AA de contraste en `seo-auditor`.

### Ajustes tras primera prueba real
- Módulo `menu`, `login`, `search`, `sidebar` emiten `background.color: "transparent"` por defecto.
- `notes.md` reestructurado para lectura rápida (resumen ejecutivo + acciones + detalles en archivo separado).

## Backlog / próximas ideas

Aquí van las ideas y mejoras propuestas que aún no se implementan.

- [ ] Ejemplo end-to-end en `examples/` con HTML + assets + output esperado.
- [ ] Templates skeleton en `templates/`.
- [ ] Versión sistema-agnóstica para compatibilidad con otras IAs (OpenCode, etc.).


## Mejoras Hechas / Aplicadas

Aquí dejaremos las tareas completadas que hayan sido desarrolladas y aplicadas a la Skill

- [x] Ejemplo Tarea completada
