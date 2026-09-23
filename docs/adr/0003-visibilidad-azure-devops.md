# ADR-0003: Proyecto público en Azure DevOps, con GitHub como fuente de verdad de la documentación

- **Estado**: Propuesto
- **Fecha**: 2026-09-22
- **Work item**: Pendiente (Azure Boards aún no está configurado)
- **Requerimientos relacionados**: RES-06, RNF-MAN-07, RNF-MAN-09, SRS 2.2 (audiencia)

## Contexto

El SRS define como audiencia a reclutadores y líderes técnicos, y establece que el proceso de ingeniería de DevFolio forma parte del portafolio. El código ya vive en un repositorio público de GitHub.

La gestión del trabajo (backlog, sprints) y la Wiki viven en Azure DevOps (RES-06). Un proyecto de Azure DevOps puede ser privado o público; para crear proyectos públicos, primero hay que habilitar la política "Allow public projects" en la organización.

Además, la función "publicar código como wiki" de Azure DevOps requiere un repositorio Git dentro del propio proyecto (Azure Repos). Como el código está en GitHub, esa función no puede publicar directamente la carpeta `/docs`.

## Opciones consideradas

1. **Proyecto privado**
   - A favor: libertad total para escribir notas, dudas o errores en los work items.
   - En contra: el proceso de ingeniería queda invisible para la audiencia principal; habría que mostrarlo con capturas de pantalla.

2. **Proyecto público**
   - A favor: cualquier visitante puede ver el backlog, los sprints y la trazabilidad entre requerimientos, historias y código, lo que refuerza el propósito del portafolio.
   - En contra: todo lo escrito en Boards y en la Wiki es visible; exige disciplina sobre qué se escribe.

Para la documentación:

- **A. Duplicar los ADR y el SRS en la Wiki**: fácil de leer en Azure DevOps, pero dos copias que se desincronizan.
- **B. GitHub como fuente de verdad y la Wiki como índice con enlaces**: una sola copia; la Wiki solo apunta al repositorio.
- **C. Sincronización automática** de `/docs` hacia el repositorio Git de la Wiki mediante GitHub Actions: una sola fuente y lectura cómoda en ambos sitios, a cambio de automatización adicional.

## Decisión

- El proyecto de Azure DevOps será **público**.
- La documentación sigue la **opción B**: `/docs` en GitHub es la única fuente de verdad (SRS y ADR se cambian solo por Pull Request); la Wiki de Azure DevOps contiene una página índice con enlaces a cada documento.
- La **opción C** queda como mejora posible en F5.
- Se ajusta la redacción de RNF-MAN-07 a: "ADRs en `/docs/adr` (fuente de verdad), indexados en la Wiki de Azure DevOps".

Reglas de contenido para el proyecto público:

- Nunca se escriben en work items, comentarios ni Wiki: secretos, cadenas de conexión, direcciones internas, datos personales de terceros ni información de facturación.
- Los work items se redactan con el mismo cuidado que un PR público: claros, profesionales y enlazados a su requerimiento.

## Consecuencias

**Positivas**

- La audiencia puede comprobar el proceso de ingeniería directamente, no solo leer sobre él.
- Una sola copia de cada documento, versionada y revisada por PR.

**Negativas y riesgos**

- Riesgo de exponer información sensible en un work item. Mitigación: las reglas de contenido anteriores y una revisión de lo escrito al cerrar cada sprint.
- La Wiki no muestra el contenido completo de los documentos, solo enlaces; la lectura ocurre en GitHub.

**Pendientes**

- Cambiar el estado a "Aceptado" al crear el proyecto en Azure DevOps, o registrar la decisión contraria si se opta por la opción privada.
