# ADR-0001: Arquitectura de microservicios con API Gateway y base de datos por servicio

- **Estado**: Aceptado
- **Fecha**: 2026-09-22
- **Work item**: Pendiente (Azure Boards aún no está configurado)
- **Requerimientos relacionados**: RES-02, RES-03, RES-04, OBJ-3, OBJ-4, RNF-ESC-01, RNF-SEG-09, RSK-01, RSK-03, RSK-07

## Contexto

DevFolio es un portafolio personal con tráfico bajo (menos de 1.000 visitas/mes, SUP-01), un único administrador y un único desarrollador con tiempo parcial (RES-07). Desde el punto de vista puramente técnico, un sistema de este tamaño no *necesita* microservicios.

Sin embargo, el producto tiene un doble propósito (SRS 2.1): además de mostrar proyectos, es un proyecto de aprendizaje que debe demostrar cómo se construye un sistema distribuido en la nube (OBJ-3: 4+ servicios desplegados con CI/CD). A la vez, el costo mensual debe mantenerse por debajo de 1 USD (OBJ-4), lo que obliga a que la arquitectura escale a cero.

El sistema tiene dominios con responsabilidades y ritmos de cambio distintos: autenticación, catálogo de proyectos, gestión de imágenes y monitoreo/estadísticas. El monitoreo, además, ejecuta procesos programados que no deben afectar al resto (RNF-DIS-03).

## Opciones consideradas

1. **Monolito Spring Boot con una sola base de datos**
   - A favor: el más simple de construir, probar, desplegar y operar; un solo arranque en frío; sin latencia de red entre módulos.
   - En contra: no cumple el objetivo de aprendizaje (OBJ-3); un proceso programado frecuente mantendría despierta la única base de datos (RSK-02).

2. **Monolito modular** (un despliegue, módulos con fronteras estrictas, por ejemplo con Spring Modulith)
   - A favor: fronteras de dominio claras con costo operativo mínimo; permite extraer servicios más adelante.
   - En contra: enseña diseño modular, pero no los problemas propios de los sistemas distribuidos (descubrimiento, gateway, trazas distribuidas, fallos parciales, despliegues independientes).

3. **Microservicios detrás de un API Gateway, con base de datos por servicio**
   - A favor: cumple OBJ-3; cada servicio escala a cero de forma independiente en Container Apps; el fallo del monitor no afecta al catálogo; cada base de datos cabe en su propia cuota gratuita de Azure SQL.
   - En contra: mayor complejidad operativa; arranques en frío encadenados (gateway + servicio + base de datos); necesidad de trazas distribuidas, contratos entre servicios y consistencia eventual.

## Decisión

Se adopta la **opción 3**: microservicios Spring Boot detrás de un único API Gateway (Spring Cloud Gateway), cada uno dueño exclusivo de su base de datos (*database per service*).

Servicios iniciales: `api-gateway`, `auth-service`, `project-service`, `media-service` y `monitor-service` (este último agrupa contacto y estadísticas para ahorrar recursos; podrán separarse con un ADR posterior).

Reglas que acompañan la decisión:

- Solo el gateway tiene ingress externo (RNF-SEG-09).
- Ningún servicio accede a la base de datos de otro; se referencian solo por ID y se comunican por REST (`/api/v1/...`) o, desde F5, por eventos.
- Los servicios son sin estado (RNF-ESC-04).
- Los servicios se incorporan de forma incremental: F1 arranca con **un solo servicio** (`project-service`, solo lectura), y cada fase agrega como máximo uno nuevo.

Se reconoce explícitamente que esta decisión está motivada por el objetivo de aprendizaje y demostración, no por la escala del sistema. Para un producto equivalente sin ese objetivo, la opción 2 sería la recomendada.

## Consecuencias

**Positivas**

- El sistema demuestra en la práctica gateway, seguridad con JWT entre servicios, trazas distribuidas, resiliencia y despliegues independientes.
- Aislamiento de fallos: una caída de GitHub o del monitor no afecta al sitio público.
- Aislamiento de costos: cada base de datos consume su propia cuota gratuita y se pausa de forma independiente.

**Negativas y riesgos**

- Riesgo de sobrecarga de alcance (RSK-01): mitigado con la incorporación incremental de servicios y el uso estricto de MoSCoW.
- Arranques en frío acumulados (RSK-03): mitigados con imágenes nativas GraalVM, estado de carga en la SPA y frontend estático siempre disponible.
- Mayor curva de aprendizaje (RSK-07): una tecnología nueva por fase.
- Se vuelven obligatorios requisitos que en un monolito serían opcionales: trazabilidad distribuida (RNF-OBS-02), circuit breakers (RNF-DIS-03) y reintentos ante la reanudación de la base de datos (RNF-DIS-06).

**Pendientes**

- Elección del motor de base de datos y su oferta gratuita: registrar en un ADR propio.
- Separar contacto y estadísticas de `monitor-service`: evaluar en F5.
