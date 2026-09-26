# ADR-0003: Proyecto privado en Azure DevOps, con GitHub como fuente de verdad y cara pública del proceso

- **Estado**: Aceptado
- **Fecha**: 2026-09-25 (propuesto el 2026-09-22; revisado antes de su aceptación)
- **Work item**: Pendiente (asignar al crear el backlog)
- **Requerimientos relacionados**: RES-06, RNF-MAN-07, RNF-MAN-09, SRS 2.2 (audiencia), SRS 10.3 (ceremonias)

## Contexto

El SRS define como audiencia a reclutadores y líderes técnicos, y establece que el proceso de ingeniería de DevFolio forma parte del portafolio. El código vive en un repositorio público de GitHub; la gestión del trabajo, en Azure DevOps (RES-06).

La versión propuesta de este ADR (2026-09-22) planteaba un proyecto **público** en Azure DevOps. Al crear la organización (2026-09-25) se verificó que esa opción ya no existe:

- Microsoft retiró los proyectos públicos de Azure DevOps. Solo las organizaciones que ya tenían activa la política "Allow public projects" pueden seguir usándola; para organizaciones nuevas la política no está disponible.
- A partir de 2027, los proyectos públicos existentes se convertirán en privados.
- Microsoft recomienda GitHub para cualquier necesidad de proyecto público.

Además, la función "publicar código como wiki" de Azure DevOps requiere un repositorio en Azure Repos, por lo que no puede publicar la carpeta `/docs` de GitHub.

## Opciones consideradas

Visibilidad del proyecto:

1. **Proyecto público**: descartada; no está disponible para organizaciones nuevas.
2. **Proyecto privado**: única opción disponible.

Cómo mostrar el proceso a la audiencia:

- **A. Solo describirlo en el README**: mínimo esfuerzo, pero sin evidencia verificable.
- **B. GitHub como cara pública del proceso**: el repositorio muestra ADR, SRS, historial de PR con Conventional Commits, retrospectivas y evidencias de cada sprint (capturas del tablero, notas de la revisión).
- **C. Invitar a revisores concretos con acceso Stakeholder**: gratuito y con acceso de lectura a los work items; útil para un proceso de selección puntual.

Documentación:

- **Duplicar SRS y ADR en la Wiki**: dos copias que se desincronizan.
- **GitHub como fuente de verdad y la Wiki como índice con enlaces**: una sola copia.

## Decisión

- El proyecto de Azure DevOps es **privado**.
- La documentación vive en `/docs` de GitHub como **única fuente de verdad** (SRS, ADR, retrospectivas) y se modifica solo por Pull Request. La Wiki de Azure DevOps contiene un índice con enlaces a esos documentos.
- **GitHub es la cara pública del proceso (opción B)**: al cerrar cada sprint se agregan en `/docs/retros` las notas de revisión y retrospectiva, con capturas del tablero cuando aporten evidencia (SRS 10.3).
- Se usa la **opción C** bajo demanda, cuando un revisor quiera ver el tablero en vivo.
- Se ajusta la redacción de RNF-MAN-07 a: "ADRs en `/docs/adr` (fuente de verdad), indexados en la Wiki de Azure DevOps".

## Consecuencias

**Positivas**

- Libertad para registrar dudas, errores y riesgos en los work items sin exponerlos públicamente.
- La evidencia pública del proceso queda en un solo lugar, junto al código, y es permanente: no depende de la política de visibilidad de Azure DevOps.

**Negativas y riesgos**

- Los enlaces `AB#id` de los PR públicos llevan a una página de inicio de sesión para los visitantes: la trazabilidad funciona para el autor, pero no es verificable desde fuera. Mitigación: las retrospectivas y capturas en `/docs/retros`.
- Mostrar el proceso requiere disciplina al cierre de cada sprint.
- Las capturas del tablero deben revisarse antes de publicarse, para no exponer información escrita pensando en un proyecto privado.

**Pendientes**

- Actualizar RNF-MAN-07 en el SRS.
- Crear `/docs/retros` con la primera retrospectiva al cerrar el primer sprint.
