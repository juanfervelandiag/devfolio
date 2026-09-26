# ADR-0006: Región de Azure para los recursos de DevFolio

- **Estado**: Propuesto
- **Fecha**: 2026-09-25
- **Work item**: Pendiente (asignar al crear el backlog)
- **Requerimientos relacionados**: RES-05, OBJ-4, RNF-EFI (latencia del sitio público), RSK-03, RSK-04
- **Relacionado con**: ADR-0004 (Azure SQL Database), ADR-0005 (suscripción anticipada)

## Contexto

Con la suscripción creada en F0 (ADR-0005), la región deja de ser una decisión de F4: debe fijarse antes de crear el primer recurso.

Es una de las decisiones menos reversibles del proyecto:

- La oferta gratuita de Azure SQL Database permite hasta 10 bases por suscripción, pero todas deben estar en **la misma región, que queda fija por suscripción** (ADR-0004).
- Mover recursos con datos (bases de datos, Blob Storage) entre regiones implica recrearlos y migrar los datos.

La región debe ofrecer todos los servicios de la arquitectura: Azure SQL Database serverless (oferta gratuita), Container Apps, Blob Storage, Application Insights y Log Analytics. Static Web Apps sirve el contenido estático desde una red de distribución global, por lo que su región influye poco en la latencia del sitio público.

La audiencia principal y el autor están en Colombia. No existe una región de Azure en Colombia.

## Opciones consideradas

1. **East US 2 (Virginia)**
   - A favor: región grande con disponibilidad amplia de servicios y capacidad; suele estar entre las de menor precio; latencia razonable desde Colombia.
   - En contra: no es la región geográficamente más cercana.

2. **South Central US (Texas)**
   - A favor: disponibilidad amplia de servicios; latencia comparable desde Colombia.
   - En contra: sin ventaja clara frente a East US 2.

3. **Brazil South (São Paulo)**
   - A favor: misma región que la organización de Azure DevOps; residencia de datos en Latinoamérica.
   - En contra: suele tener precios más altos; la ruta de red desde Colombia a Brasil no siempre es más corta que hacia Estados Unidos; algunos servicios nuevos llegan después.

4. **Mexico Central**
   - A favor: la región más cercana geográficamente.
   - En contra: región reciente; hay que verificar que ofrezca todos los servicios y la oferta gratuita de Azure SQL.

## Decisión (propuesta)

Se propone **East US 2** para todos los recursos regionales de DevFolio.

Antes de aceptar este ADR se debe verificar en el portal:

- Que East US 2 permite crear una base de Azure SQL con la oferta gratuita (el banner "Apply offer" en la creación de la base).
- Que Container Apps con el plan de consumo está disponible en la región.
- La latencia real desde Colombia hacia las regiones candidatas (por ejemplo, con una herramienta de medición de latencia de Azure).

La región de la organización de Azure DevOps (Brazil) es independiente de esta decisión: solo determina dónde se guardan los work items, no dónde corren los servicios.

## Consecuencias

**Positivas**

- Todos los servicios en una sola región: sin tráfico ni latencia entre regiones.
- Precios bajos, que ayudan a mantener el presupuesto de OBJ-4 si se supera alguna cuota gratuita.

**Negativas y riesgos**

- Cambiar de región después obliga a recrear las bases de datos y migrar los datos.
- Latencia algo mayor que la de una región hipotética en Colombia; su efecto es menor frente al arranque en frío de la escala a cero (RSK-03).

**Pendientes**

- Completar las verificaciones y cambiar el estado a "Aceptado" antes de crear el primer recurso.
