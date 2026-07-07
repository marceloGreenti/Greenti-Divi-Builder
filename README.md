# Greenti Divi Builder Skill

Pipeline `html-to-divi`: convierte HTML maquetado + assets + design tokens en JSON de Divi 5.8.1 importable a WordPress.

## Contexto

Herramienta interna de Greenti para el flujo de trabajo:
Figma → HTML (Skill separada) → JSON de Divi (esta Skill) → Import manual en WordPress

## Requisitos

- Claude Code instalado (`npm install -g @anthropic-ai/claude-code`).
- Node.js >= 18.
- (Opcional) `jq` para validación de JSON: `sudo apt install jq`.

## Estructura del proyecto
greenti-divi-builder/
├── .claude/
│   └── skills/
│       └── html-to-divi/
│           ├── SKILL.md                 ← trigger de la Skill
│           ├── divi5-reference.md       ← catálogo de módulos Divi 5.8.1
│           ├── agents/                  ← 4 subagentes especializados
│           ├── rules/                   ← reglas de mapeo/inferencia
│           ├── templates/               ← JSON skeletons
│           └── examples/                ← ejemplos end-to-end
└── projects/
└── <nombre-cliente>/
├── html/                        ← HTML de entrada
├── assets/                      ← imágenes, íconos
├── design-tokens.md             ← manifiesto de tokens (opcional)
└── output/                      ← JSON emitidos + archivos compañeros

## Uso

1. Preparar la carpeta del proyecto en `projects/<nombre-cliente>/` con la estructura de arriba.
2. Arrancar Claude Code desde la raíz del repo: `claude`
3. Pedirle: "Ejecuta la skill html-to-divi sobre projects/<nombre-cliente>/"
4. Al finalizar, revisar `projects/<nombre-cliente>/output/notes.md` para las acciones requeridas antes de importar.

## Documentación

- `divi5-reference.md` — schema completo de Divi 5.8.1 y catálogo de módulos.
- `SKILL.md` — descripción del pipeline y las 5 fases.
- Cada subagente en `agents/` tiene su rol documentado.

## Contribuir / mantener

- Cambios menores (reglas, correcciones puntuales): editar el archivo relevante y commit.
- Nuevos módulos al catálogo: seguir el protocolo en `divi5-reference.md` (exportar ejemplo mínimo desde Divi, incorporarlo al reference).
- Cambios estructurales: discutir antes en Backlog / Issues.

## Versionado

Se usa Semver (`vMAJOR.MINOR.PATCH`).
- MAJOR: cambios que rompen compatibilidad de outputs previos.
- MINOR: nuevas features (nuevos módulos, nuevas reglas).
- PATCH: correcciones de bugs y ajustes menores.
