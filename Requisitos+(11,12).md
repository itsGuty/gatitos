# Requisitos

## 1. Contexto y alcance

### 1.1. Objetivo del sistema

El sistema resolverá el problema de la gestión de facturación de la empresa Turbineh. Centralizando los datos de sus clientes, sus facturas, sus plazos y los estados de sus contratos lograremos que un usuario interno de la empresa pueda gestionar y auditar en un par de clicks.

### 1.2. Alcance

El cliente necesita un sistema que reúna toda la información de facturación de la compañía y convertirla en respuestas, avisos e informes. Turbineh necesita un control claro y sencillo de pagos y facturas de clientes aumentando así la velocidad del control de facturación.

### 1.3. Actores y partes interesadas

Identificar las personas, organizaciones o sistemas que interactúan con el sistema
o tienen algún interés relevante en su funcionamiento.

<!--
EJEMPLO:

| Actor / stakeholder | Descripción | Intereses principales |
|---|---|---|
| Personal administrativo | Personal de Turbine encargado de gestionar clientes, facturas y cobros | Realizar las tareas administrativas de forma rápida y disponer de información actualizada |
| Administrador | Usuario con permisos sobre la gestión del sistema y sus usuarios | Gestionar usuarios, permisos y configuración |
| Cliente de Turbine | Empresa a la que Turbine presta sus servicios y factura periódicamente | Recibir correctamente sus facturas y comunicaciones relacionadas con los pagos |
| Sistema de pagos | Sistema externo en el que se reflejan los cobros | Proporcionar información que permita determinar si una factura ha sido pagada |

Los elementos anteriores son únicamente un ejemplo de cómo documentar esta sección.
El equipo debe identificar y justificar los actores y stakeholders que considere
adecuados a partir de la información del caso.
-->

| Actor / stakeholder | Descripción | Intereses principales |
|---|---|---|
| Lector | Trabajador de Turbine que gestiona las facturas de los clientes que le ha asignado un Admin | Trabajar rápido, ver el historial del cliente y recibir avisos útiles |
| Admin | Trabajador de Turbine con permisos sobre usuarios, asignación de clientes, configuración, objetivos y reglas corporativas | Controlar la configuración y recibir información de incidencias y fallos |
| Superadmin (posible rol, ver Q-02) | Perfil para cambios de mayor impacto en las reglas corporativas | Que los cambios críticos solo los haga el perfil adecuado |
| Solicitante / Aprobador | Roles dentro del flujo de aprobaciones (Lector o Admin según la operación) | Conocer el estado de sus solicitudes; gestionar las pendientes sin aprobar las propias |
| Desarrollador / equipo técnico | Personas que mantienen y despliegan el sistema | Diagnosticar errores y desplegar de forma reproducible |
| Alejandro Ruiz (Turbine) | Responsable de negocio que define las necesidades | Reglas económicas correctas y comprensibles; sistema mantenible y desplegable |
| Cliente de Turbine | Empresa externa a la que se factura; no accede a la aplicación | Recibir correctamente facturas y comunicaciones |
| Servicios de correo y mensajería (email, WhatsApp) | Servicios externos para enviar facturas, notificaciones y resúmenes | Entrega fiable; sus fallos deben quedar registrados |
| Sistema de pagos / banca | Sistema externo donde se reflejan los cobros | Permitir la conciliación de cobros con facturas |
| Agente de IA (futuro, ampliación) | Posible agente que opere contra la API | Disponer de documentación estructurada de la API |

## 2. Requisitos de negocio

Los requisitos de negocio describen los objetivos, necesidades o resultados que
la organización pretende conseguir mediante el desarrollo del sistema. No describen
todavía funcionalidades concretas del software.

<!--
EJEMPLO:

| ID | Requisito | Fuente |
|---|---|---|
| BR-01 | Reducir el trabajo manual asociado a la generación de facturas y al seguimiento de los cobros | Cliente / entrevista |
| BR-02 | Hacer más eficiente la gestión administrativa a medida que aumente el número de clientes de Turbine | Cliente / entrevista |

Observa que estos requisitos expresan POR QUÉ se necesita el sistema y qué pretende
conseguir la organización. No indican todavía cómo debe hacerlo el software.
-->

| ID | Requisito | Fuente |
|---|---|---|
| BR-01 | El cliente necesita un sistema que reúna toda la información de facturación de la compañía y convertirla en respuestas, avisos e informes.| Cliente / entrevista  |
| BR-02 | Turbineh necesita un control claro y sencillo de pagos y facturas de clientes aumentando así la velocidad del control de facturación. | Cliente / entrevista |


## 3. Requisitos de usuario

Los requisitos de usuario describen las necesidades, objetivos o servicios que los
usuarios esperan obtener del sistema. Deben expresarse desde el punto de vista del
usuario, evitando entrar todavía en detalles internos de implementación.

<!--
EJEMPLO:

| ID | Actor | Requisito | Fuente |
|---|---|---|---|
| UR-01 | Personal administrativo | Poder generar y gestionar facturas de los clientes sin tener que realizar manualmente cada facturación mensual | Cliente / entrevista |
| UR-02 | Personal administrativo | Poder conocer qué facturas están pagadas y cuáles siguen pendientes | Cliente / entrevista |
| UR-03 | Administrador | Poder gestionar los usuarios del sistema y determinar a qué clientes puede acceder cada uno | Cliente / entrevista |

Un requisito de usuario expresa QUÉ necesita conseguir un usuario con el sistema,
pero normalmente no especifica todavía todos los comportamientos detallados que
serán necesarios para proporcionarle ese servicio.
-->

| ID | Actor | Requisito | Fuente |
|---|---|---|---|
| UR-01 | Lector / Admin | Poder hacer login usando su cuenta interna de la empresa (email y contraseña) | Cliente / entrevista |
| UR-02 | Lector | Poder gestionar las facturas de los clientes que le asoció el usuario admin | Cliente / entrevista |
| UR-03 | Admin | Poder asignar clientes a los lectores | Cliente / entrevista |
| UR-04 | Admin | Poder cambiar los parámetros de configuración del sistema | Cliente / entrevista |
| UR-05 | Lector / Admin | Automatización de todos los procesos que no requieran intervención humana: generación de facturas, envío y notificaciones | Cliente / entrevista |


## 4. Requisitos del sistema

### 4.1. Requisitos funcionales

Los requisitos funcionales describen comportamientos, servicios u operaciones que
debe proporcionar el sistema. Siempre que sea posible, deben formularse de manera
clara, precisa y verificable.

<!--
EJEMPLO:

| ID | Requisito | Relacionado con |
|---|---|---|
| FR-01 | El sistema deberá permitir crear y almacenar una factura asociada a un cliente | UR-01 |
| FR-02 | El sistema deberá permitir indicar si la facturación de un cliente es puntual o recurrente | UR-01 |
| FR-03 | El sistema deberá generar automáticamente las nuevas facturas correspondientes a una facturación recurrente | UR-01 |
| FR-04 | El sistema deberá registrar si una factura se encuentra pagada o pendiente de pago | UR-02 |
| FR-05 | El sistema deberá permitir al administrador asignar clientes a los usuarios del sistema | UR-03 |

Evita requisitos como "El sistema gestionará las facturas correctamente", porque
no permiten saber con precisión qué comportamiento se espera.

También deben evitarse decisiones de diseño innecesarias, por ejemplo:
"El sistema utilizará una tabla SQL denominada FACTURAS".
Eso corresponde al diseño, no a los requisitos.
-->

| ID | Requisito | Relacionado con |
|---|---|---|
| FR-01 | El sistema deberá generar facturas simples y recurrentes (formato dado) | UR-05 |
| FR-02 | El sistema deberá poder llevar el control de las facturas existentes | UR-02 |
| FR-03 | El sistema deberá permitir hacer login en la plataforma | UR-01 |
| FR-04 | El sistema deberá realizar analíticas de pagos e impagos tras 1 semana, 1 mes y 1 año | UR-05 |
| FR-05 | El sistema deberá enviar notificaciones de impago al cliente por vías como email o Whatsapp (a partir del día 5 cada 2-3 días) (antes de ser impago notif dia 3 y 5 de pago en plazo) | UR-05 |
| FR-06 | El sistema deberá tener dos roles (admin y lector) | UR-03 |
| FR-07 | El sistema deberá tener una copia de seguridad del registro de movimientos y los accesos cedidos a los lectores para el admin | UR-03 |
| FR-09 | El sistema deberá evaluar anualmente posible cambio de plan según ingresos anuales | UR-05 |
| FR-10 | El sistema deberá permitir que los lectores puedan leer y descargar facturas | UR-02 |
| FR-11 | El sistema deberá crear la factura el día que contratan BrainOS de lo equivalente a lo que queda de mes, y luego el día 1 una factura completa normal | UR-05 |
| FR-12 | El sistema deberá permitir modificar factura en trimestre, si ya ha pasado el trimestre se bloquea, debido a que ya ha sido entregado a hacienda | UR-05 |
| FR-13 | El sistema deberá al darse Cliente de baja, si le falta un pago envía un aviso, se mantendrán los datos del cliente el máximo tiempo posible legalmente, se realizará un borrado lógico | UR-05 |




### 4.2. Requisitos no funcionales

Los requisitos no funcionales describen propiedades de calidad, restricciones o
condiciones que debe satisfacer el sistema. Cuando sea posible, deberán incluir
algún criterio que permita determinar si el requisito se cumple.

<!--
EJEMPLO:

| ID | Categoría | Requisito | Criterio verificable |
|---|---|---|---|
| NFR-01 | Usabilidad | Las operaciones administrativas habituales deberán poder realizarse de forma rápida y sencilla | Una operación habitual como generar o consultar una factura debería poder completarse aproximadamente en menos de un minuto |
| NFR-02 | Portabilidad / Usabilidad | La aplicación deberá poder utilizarse desde distintos tipos de dispositivo | Las principales funcionalidades deberán ser utilizables desde ordenador, tableta y teléfono móvil |
| NFR-03 | Escalabilidad | El sistema deberá poder gestionar un crecimiento significativo del número de clientes | El diseño no deberá asumir un número pequeño o fijo de clientes y deberá permitir gestionar potencialmente miles de ellos |

Evita expresiones difíciles de comprobar como:
"La aplicación deberá ser moderna", "muy rápida" o "fácil de usar".

Cuando el cliente emplee expresiones de este tipo, el equipo deberá intentar
transformarlas en criterios más concretos y verificables o documentar la
interpretación realizada.
-->

| ID | Categoría | Requisito | Criterio verificable |
|---|---|---|---|
| NFR-01 | Usabilidad | Las operaciones administrativas habituales deberán poder realizarse de forma rápida y sencilla | Una operación habitual como generar o consultar una factura debería poder completarse aproximadamente en menos de un minuto |
| NFR-02 | Portabilidad / Usabilidad | La aplicación deberá poder utilizarse desde distintos tipos de dispositivo y debe ser web responsiva | Las principales funcionalidades deberán ser utilizables desde ordenador, tableta y teléfono móvil |
| NFR-03 | Escalabilidad | El sistema deberá poder gestionar un crecimiento significativo del número de clientes | El diseño no deberá asumir un número pequeño o fijo de clientes y deberá permitir gestionar potencialmente miles de ellos |
| NFR-04 | Privacidad | El sistema deberá cumplir con la protección de datos fiscales | El diseño deberá seguir la ley de protección y privacidad de datos almacenando información en los tiempos permitidos |
| NFR-05 | Estandarización | El sistema deberá generar facturas estándar y válidas | El diseño de las facturas deberá seguir el estándar de la empresa |
| NFR-06 | Usabilidad | El sistema deberá ser completamente independiente de BrainOS | El usuario final de esta aplicación es quien trabaja en la empresa, no quien usa sus servicios |
| NFR-07 | Seguridad | El sistema deberá hacer login con email interno de la empresa y contraseña | Los usuarios tanto lectores como administradores previamente registrados hacen login con sus datos internos de la empresa |


## 5. Reglas de negocio

Reglas del dominio o de la organización que condicionan el comportamiento del sistema
pero que no representan por sí mismas una funcionalidad.

| ID | Regla |
|---|---|
| RB-01 | ... |
| RB-02 | ... |


## 6. Suposiciones y decisiones de análisis

En esta sección se documentarán las decisiones tomadas por el equipo cuando la
información proporcionada por el cliente no sea suficiente para determinar unívocamente
un requisito.

| ID | Suposición o decisión | Justificación | Requisitos afectados |
|---|---|---|---|
| A-01 | ... | ... | FR-... |
| A-02 | ... | ... | ... |


## 7. Dudas, ambigüedades y cuestiones pendientes

Se incluirán aquí aspectos de la información proporcionada por el cliente que sean
ambiguos, incompletos, contradictorios o que requieran aclaración.

| ID | Cuestión | Origen | Impacto | Estado / resolución |
|---|---|---|---|---|
| Q-01 | ... | Entrevista | ... | Pendiente |
| Q-02 | ... | ... | ... | Resuelta: ... |


## 8. Trazabilidad

La tabla permitirá comprobar la relación entre las necesidades de negocio,
los requisitos de usuario y los requisitos del sistema.

| Requisito de negocio | Requisito de usuario | Requisito funcional / no funcional |
|---|---|---|
| BR-01 | UR-01 | FR-01, FR-02 |
| BR-02 | UR-03 | FR-07, NFR-02 |


## 9. Glosario

| Término | Definición |
|---|---|
| ... | ... |
