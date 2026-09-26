# Architecture Decision Records

Registro de las decisiones técnicas de DevFolio. Cada ADR se crea por Pull Request y, una vez aceptado, no se edita: si una decisión cambia, se escribe un ADR nuevo que la reemplaza. La plantilla está en [template.md](template.md).

| ADR | Título | Estado | Fecha |
| --- | --- | --- | --- |
| [0001](0001-arquitectura-microservicios.md) | Arquitectura de microservicios con API Gateway y base de datos por servicio | Aceptado | 2026-09-22 |
| [0002](0002-momento-suscripcion-azure.md) | Diferir la creación de la suscripción de Azure hasta el final de F3 | Reemplazado por [ADR-0005](0005-suscripcion-azure-anticipada.md) | 2026-09-22 |
| [0003](0003-visibilidad-azure-devops.md) | Proyecto privado en Azure DevOps, con GitHub como fuente de verdad y cara pública del proceso | Aceptado | 2026-09-25 |
| [0004](0004-motor-base-de-datos.md) | Azure SQL Database (oferta gratuita permanente) como motor de base de datos | Aceptado | 2026-09-22 |
| [0005](0005-suscripcion-azure-anticipada.md) | Crear la suscripción de Azure en F0, porque Azure DevOps la exige | Aceptado | 2026-09-25 |