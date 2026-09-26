# ADR-0004: Azure SQL Database (oferta gratuita permanente) como motor de base de datos

- **Estado**: Aceptado
- **Fecha**: 2026-09-22
- **Work item**: Pendiente (Azure Boards aún no está configurado)
- **Requerimientos relacionados**: RES-03, RES-05, OBJ-4, RNF-DIS-04, RNF-DIS-05, RNF-DIS-06, RNF-ESC-05, RNF-ESC-06, RNF-MAN-10, RSK-02, RSK-04, RSK-10
- **Relacionado con**: ADR-0001 (base de datos por servicio), ADR-0002 (momento de creación de la suscripción)

## Contexto

ADR-0001 establece que cada microservicio es dueño exclusivo de su base de datos, lo que implica al menos 4 bases de datos (auth, project, media, monitor). El modelo de datos es relacional: relaciones muchos a muchos (proyecto–tecnología), restricciones de unicidad (slug, nombre de tecnología) y reglas de integridad (RN-01 a RN-09).

El presupuesto objetivo es menor a 1 USD/mes (OBJ-4) y el sitio debe operar indefinidamente, no solo durante un periodo de prueba. El tráfico es bajo e intermitente (SUP-01).

La versión 0.1 del SRS contemplaba PostgreSQL. Al revisar las ofertas gratuitas de Azure se identificó que:

- **Azure Database for PostgreSQL Flexible Server** es gratuito solo durante los 12 meses de la cuenta gratuita; después tiene un costo mensual fijo por servidor que supera por sí solo el presupuesto de OBJ-4.
- **Azure SQL Database** ofrece una capa gratuita permanente, disponible en cualquier tipo de suscripción: hasta 10 bases de datos General Purpose serverless por suscripción, cada una con 100.000 vCore-segundos de cómputo, 32 GB de datos y 32 GB de respaldo al mes. Al agotar la cuota se puede elegir entre pausar la base hasta el mes siguiente (sin cargos) o facturar el exceso. Las 10 bases deben estar en la misma región, fija por suscripción.

## Opciones consideradas

1. **Azure Database for PostgreSQL Flexible Server**
   - A favor: motor de código abierto, muy popular en el mercado laboral; excelente soporte en Spring, Flyway y Testcontainers; imagen local liviana.
   - En contra: gratis solo 12 meses; después su costo fijo mensual incumple OBJ-4 de forma permanente.

2. **Azure SQL Database serverless, oferta gratuita permanente**
   - A favor: gratis sin fecha de vencimiento; una base por servicio sin costo adicional (4 de 10); respaldos automáticos con restauración a un punto en el tiempo (RNF-DIS-04); al agotar la cuota se pausa sin cobrar.
   - En contra: cuota de cómputo limitada que se consume también mientras la base espera para pausarse; arranque en frío al reanudar; SQL Server en local consume más RAM que PostgreSQL (RSK-10); motor propietario.

3. **Azure Cosmos DB (capa gratuita)**
   - A favor: capa gratuita permanente.
   - En contra: no es relacional; el modelo (muchos a muchos, unicidad, integridad) tendría que resolverse en la aplicación; se pierde el aprendizaje de JPA y Flyway.

4. **PostgreSQL autogestionado en un contenedor de Container Apps**
   - A favor: sin costo de licencia de base de datos.
   - En contra: escalar a cero apagaría la base; persistencia sobre Azure Files no es adecuada para bases de datos; respaldos, parches y alta disponibilidad quedarían a cargo propio.

5. **PostgreSQL serverless de un proveedor externo a Azure**
   - A favor: algunos proveedores ofrecen capas gratuitas con escala a cero.
   - En contra: incumple RES-05 (despliegue en Azure), añade latencia entre nubes y otra dependencia externa con términos propios.

## Decisión

Se adopta la **opción 2**: Azure SQL Database serverless con la oferta gratuita permanente, una base por microservicio (`dvf-auth`, `dvf-project`, `dvf-media`, `dvf-monitor`).

Configuración obligatoria de cada base:

- General Purpose serverless, 0,5 vCore mínimo, tamaño máximo ≤ 32 GB (RNF-ESC-06).
- Comportamiento al agotar la cuota: **pausar hasta el mes siguiente**, nunca facturar el exceso.
- **Retraso de autopausa en el mínimo permitido (15 minutos)**, no en el valor por defecto de 60 (ver "Presupuesto de reanudaciones").
- Alerta sobre la métrica de cuota gratuita restante.

Para preservar la portabilidad (RSK-04, RNF-MAN-10):

- Acceso a datos solo mediante Spring Data JPA y JPQL; sin T-SQL en el código de aplicación.
- El SQL específico del motor queda confinado a las migraciones Flyway.
- En local se usa el contenedor oficial de SQL Server; en CI, Testcontainers con el módulo MSSQL.

## Consecuencias

### Presupuesto de reanudaciones (hallazgo principal)

Cada vez que una base se reanuda, consume cómputo durante todo el retraso de autopausa aunque no haya actividad, facturado al mínimo configurado de vCore y memoria. Estimación, a verificar con la métrica de cuota consumida:

| Retraso de autopausa | Costo aproximado por reanudación | Reanudaciones posibles al mes | Por día |
| --- | --- | --- | --- |
| 60 min (por defecto) | ~1.800–2.400 vCore-s | ~40–55 | ~1,5 |
| 15 min (mínimo) | ~450–600 vCore-s | ~165–220 | ~5–7 |

Con 1.000 visitas al mes (SUP-01), si las visitas están separadas por más de 15 minutos, cada una reanudaría la base de `project-service`. Ese volumen puede agotar la cuota antes de fin de mes, y la base quedaría pausada, dejando el catálogo público sin datos hasta el mes siguiente.

Esto convierte la estrategia de lectura del sitio público en una decisión de arquitectura propia: el tráfico público no debería depender de que la base esté despierta. Se registrará en **ADR-0007** (por ejemplo, servir el catálogo público desde una instantánea estática generada al publicar, dejando la base solo para el panel de administración).

### Otras consecuencias

**Positivas**

- Costo de base de datos de 0 USD de forma permanente, con aislamiento de cuota por servicio.
- Respaldos automáticos y restauración a un punto en el tiempo incluidos.
- El mecanismo de pausa al agotar la cuota elimina el riesgo de cargos inesperados por base de datos.

**Negativas y riesgos**

- Arranque en frío al reanudar, y la primera conexión puede fallar mientras la base despierta: obliga a reintentos con backoff (RNF-DIS-06).
- Las sondas de salud no deben tocar la base: el indicador de base de datos de Spring Actuator se excluye de los grupos de liveness y readiness que consulta Container Apps; de lo contrario, las sondas impedirían la autopausa (RSK-02, RNF-DIS-05).
- Ninguna herramienta de consulta (SSMS, Azure Data Studio, extensiones del IDE) debe quedar conectada a producción: una conexión abierta impide la autopausa.
- SQL Server en local requiere unos 2 GB de RAM (RSK-10): una sola instancia local con las 4 bases y perfiles de Docker Compose.
- La compatibilidad del driver JDBC de SQL Server con GraalVM Native Image debe validarse en el spike de compilación nativa (RSK-05).
- La elección de región queda atada a esta decisión, porque las 10 bases gratuitas comparten una región fija por suscripción (ver ADR-0002).
- El motor es propietario; se mitiga con la disciplina de portabilidad descrita y con el hecho de que migrar implicaría reescribir solo las migraciones Flyway.

**Pendientes**

- ADR-0007: estrategia de lectura del sitio público frente a la cuota de cómputo.
- Actualizar RNF-ESC-05 (retraso de autopausa de 15 minutos) y RNF-DIS-05 (sondas sin acceso a la base) en el SRS.
