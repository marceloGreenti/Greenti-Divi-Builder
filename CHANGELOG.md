# Changelog

Todas las mejoras notables a la Skill `html-to-divi` se documentan aquí.

Se sigue el formato de [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) y versionado [Semver](https://semver.org/lang/es/).

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
