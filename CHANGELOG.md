# Changelog

Todas las mejoras notables a la Skill `html-to-divi` se documentan aquí.

Se sigue el formato de [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) y versionado [Semver](https://semver.org/lang/es/).


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
