# ADR-0002: Diferir la creación de la suscripción de Azure hasta el final de F3

- **Estado**: Reemplazado por ADR-0005
- **Fecha**: 2026-09-22
- **Work item**: Pendiente (Azure Boards aún no está configurado)
- **Requerimientos relacionados**: RES-05, OBJ-4, RNF-ESC-03, RNF-ESC-05, RSK-04

## Contexto

La cuenta gratuita de Azure combina tres beneficios con relojes distintos:

- **200 USD de crédito**, válidos solo durante los primeros 30 días desde el registro.
- **Servicios gratuitos por 12 meses** (por ejemplo, 5 GB de Blob Storage), que empiezan a contar desde el registro.
- **Servicios siempre gratuitos**, sin vencimiento.

La cuenta gratuita es una por cliente. Al terminar los 30 días (o al agotar el crédito), la suscripción se deshabilita salvo que se actualice a pago por uso, lo que retira el límite de gasto.

Según el plan de entregas, el primer despliegue en Azure ocurre en F4, unas 10 semanas después del inicio de F0. Las fases F0 a F3 se desarrollan íntegramente en local (Docker Compose, Azurite, SQL Server en contenedor) y en CI (Testcontainers en GitHub Actions).

Los servicios de los que depende la arquitectura en producción (oferta gratuita de Azure SQL Database, Static Web Apps Free, cuota mensual de Container Apps) están disponibles en suscripciones de pago por uso, por lo que el proyecto no depende del crédito para operar.

Azure DevOps no requiere una suscripción de Azure: basta con una cuenta Microsoft.

## Opciones consideradas

1. **Crear la suscripción ahora (F0)**
   - A favor: permite explorar el portal y los servicios desde el inicio.
   - En contra: el crédito de 200 USD vence sin uso real; los 12 meses de servicios gratuitos se consumen durante fases que no los necesitan; obliga a pasar a pago por uso semanas antes de tener algo que desplegar.

2. **Crear la suscripción al inicio de F1 para un *walking skeleton*** (desplegar desde el principio un esqueleto mínimo, por ejemplo el frontend en Static Web Apps)
   - A favor: valida el pipeline de despliegue y la infraestructura temprano; reduce el riesgo de sorpresas en F4.
   - En contra: mismo problema de desperdicio del crédito; adelanta trabajo de F4 y aumenta el alcance de F1 (RSK-01).

3. **Diferir la creación hasta el final de F3 o el inicio de F4**
   - A favor: los 30 días de crédito coinciden con el primer despliegue real y sirven de red de seguridad ante errores de configuración costosos; los 12 meses de servicios gratuitos cubren el periodo de operación.
   - En contra: la infraestructura en la nube no se valida hasta F4; los problemas de despliegue (GraalVM, conectividad, cuotas) aparecen tarde.

## Decisión

Se adopta la **opción 3**. La suscripción de Azure se crea al final de F3 o al inicio de F4, justo antes del primer despliegue.

Hasta entonces:

- Se crean la cuenta Microsoft y la organización de Azure DevOps (no requieren suscripción).
- El entorno local debe reproducir producción lo más fielmente posible (RNF-POR-02), para que F4 sea un cambio de destino y no de comportamiento.

Al crear la suscripción, el orden obligatorio es:

1. Elegir la región y registrarla en un ADR propio. Es la decisión menos reversible: las bases de datos gratuitas de Azure SQL deben estar en una misma región fija por suscripción.
2. Crear el grupo de recursos con etiquetas (`project`, `env`, `owner`).
3. Crear la alerta de presupuesto de 1 USD (RNF-ESC-03) **antes** de cualquier otro recurso.
4. Solo entonces, crear los recursos del sistema.

## Consecuencias

**Positivas**

- El crédito se usa cuando aporta valor: absorber errores durante el primer despliegue.
- Los servicios gratuitos de 12 meses cubren la operación real del sitio, no el desarrollo local.
- Se refuerza la disciplina de paridad entre local y producción.

**Negativas y riesgos**

- Los problemas propios de la nube se descubren tarde. Mitigación: en F4 se reserva tiempo explícito para ajustes de despliegue, y la compilación nativa GraalVM se prueba en CI desde F1 (RSK-05).
- Tras los 30 días será obligatorio pasar a pago por uso. Las alertas de presupuesto **avisan pero no detienen el gasto**, por lo que la revisión mensual de costos (RNF-ESC-05) deja de ser opcional.
- Los límites de las capas gratuitas pueden cambiar antes de F4 (RSK-04): deben revisarse en la documentación oficial al crear la suscripción.

**Revisión**

Si antes de F4 surge la necesidad de validar algo en Azure (por ejemplo, un spike técnico), esta decisión se revisa y, si cambia, se reemplaza con un nuevo ADR.
