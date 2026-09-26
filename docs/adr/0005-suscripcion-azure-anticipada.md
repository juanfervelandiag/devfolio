# ADR-0005: Crear la suscripción de Azure en F0, porque Azure DevOps la exige

- **Estado**: Aceptado
- **Fecha**: 2026-09-25
- **Work item**: Pendiente (asignar al crear el backlog)
- **Requerimientos relacionados**: RES-05, RES-06, OBJ-4, RNF-ESC-03, RNF-ESC-05, RSK-04
- **Reemplaza a**: ADR-0002

## Contexto

ADR-0002 difería la creación de la suscripción de Azure hasta el final de F3, para que los 30 días de crédito de la cuenta gratuita (200 USD) coincidieran con el primer despliegue. Una de sus premisas era que **Azure DevOps no requiere suscripción**.

Al crear la organización de Azure DevOps (2026-09-25), esa premisa resultó falsa:

- Azure DevOps exige ahora vincular una suscripción de Azure activa para crear organizaciones nuevas. El botón "Continue" queda deshabilitado sin ella.
- Vincular la suscripción no genera cargos mientras el uso quepa en el nivel gratuito (hasta 5 usuarios Basic, sin pipelines de pago, Artifacts ≤ 2 GB, sin GitHub Advanced Security). DevFolio solo usa Boards y Wiki con un usuario.
- Había reportes de suscripciones de prueba rechazadas con el error "invalid offer code"; en este caso **la suscripción de la cuenta gratuita fue aceptada sin necesidad de actualizar a pago por uso**.

Además, el tenant de Microsoft Entra ID, necesario para la administración de Azure y para la federación OIDC con GitHub Actions en F4, solo puede crearse con una cuenta de pago o al registrar una cuenta gratuita de Azure. Crear la suscripción también lo crea.

## Opciones consideradas

1. **Mantener ADR-0002 y usar GitHub Projects hasta F3**
   - A favor: conserva el crédito para F4; todo el proceso en un solo lugar, con integración nativa con PR.
   - En contra: incumple RES-06; se pierde el aprendizaje de Azure DevOps, que es un objetivo explícito del proyecto; obliga a migrar el backlog o a cambiar de herramienta.

2. **Crear la suscripción ahora (F0)**
   - A favor: cumple RES-06; habilita Azure DevOps y el tenant de Entra ID desde el inicio; los controles de costo quedan configurados antes de que exista cualquier recurso.
   - En contra: el crédito de 200 USD vence en F0 prácticamente sin uso; los 12 meses de servicios gratuitos empiezan a correr diez semanas antes de necesitarlos; la región de Azure se vuelve una decisión de F0.

## Decisión

Se adopta la **opción 2**. La suscripción se creó el 2026-09-25 con la cuenta gratuita de Azure (crédito de 200 USD, 30 días), con la misma cuenta Microsoft que administra Azure DevOps. ADR-0002 pasa a estado "Reemplazado por ADR-0005".

Configuración realizada, en este orden:

1. Registro de la cuenta gratuita (país de facturación: Colombia).
2. Presupuesto `budget-devfolio-mensual` a nivel de **suscripción**: 1 USD/mes, con alertas al 50 %, 80 % y 100 % del gasto real y al 100 % del pronosticado, y vencimiento en 2036 (RNF-ESC-03).
3. Vinculación de la suscripción a la organización de Azure DevOps `juanfervelandiag` (región: Brazil).

Reglas vigentes desde esta decisión:

- **No se crea ningún recurso de Azure** hasta que exista el ADR de región (ADR-0006) aceptado.
- **Antes del día 30 (≈ 2026-10-25) se actualiza la suscripción a pago por uso.** Al terminar el crédito, una suscripción gratuita sin actualizar se deshabilita, y la organización de Azure DevOps depende de ella para la facturación. Con el presupuesto ya creado y sin recursos desplegados, el costo esperado tras la actualización es 0 USD.

## Consecuencias

**Positivas**

- Azure DevOps y el tenant de Entra ID disponibles desde F0, como pide RES-06.
- La barrera de costos existe antes que cualquier recurso, que es el orden correcto en una empresa.

**Negativas y riesgos**

- Se pierde el crédito como red de seguridad para el primer despliegue de F4. Mitigación: el presupuesto con alertas de pronóstico y la revisión mensual de costos (RNF-ESC-05) pasan a ser la protección principal.
- Tras la actualización a pago por uso desaparece el límite de gasto: **las alertas avisan pero no detienen el gasto**. Cualquier recurso creado por error empieza a facturarse de inmediato.
- Los 12 meses de servicios gratuitos (por ejemplo, Blob Storage) vencerán unas diez semanas antes de lo previsto. El impacto es bajo: se estima en céntimos por mes con el volumen de DevFolio.
- Olvidar la actualización antes del día 30 dejaría la suscripción deshabilitada y podría afectar a la organización de Azure DevOps. Mitigación: work item con fecha límite en el backlog.

**Pendientes**

- ADR-0006: región de Azure (fija para las bases de datos gratuitas de Azure SQL).
- Work item: "Actualizar la suscripción a pago por uso", con fecha límite 2026-10-23.
- Actualizar la tabla 9.4 del SRS: Azure DevOps sigue siendo gratuito para este uso, pero requiere una suscripción de Azure vinculada.
