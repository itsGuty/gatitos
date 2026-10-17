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
| BR-03 | Poder diagnosticar los fallos técnicos y entender, en términos de negocio, qué operación falló, sin exponer información sensible a los usuarios | Cliente / entrevista |
| BR-04 | Mantener datos de clientes fiables y suficientes para facturar, con un historial comprensible y acceso ágil a los clientes habituales | Cliente / entrevista |
| BR-05 | Proteger la información de clientes, facturas y cobros frente a accesos no autorizados | Cliente / entrevista |
| BR-06 | Que las reglas económicas se mantengan correctas, verificables y comprensibles con el tiempo, y que los importes sean exactos | Cliente / entrevista |
| BR-07 | Que Turbine pueda desplegar, mantener y recuperar el sistema con tecnología libre o accesible, y que funcione en los navegadores, móviles y formatos habituales | Cliente / entrevista |
| BR-08 | (Ampliación parcial) Poder crecer sin rehacer el sistema: varias sociedades emisoras, clasificación por proyecto y otras monedas | Cliente / entrevista |
| BR-09 | Seguir el progreso de objetivos y detectar a tiempo situaciones que requieren acción, controlando quién puede cambiar las reglas | Cliente / entrevista |
| BR-10 | Que los cambios de estado del cliente, las recurrencias y las operaciones sensibles tengan consecuencias claras y estén controlados | Cliente / entrevista |

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
| UR-06 | Desarrollador | Poder consultar los errores técnicos con un identificador y su traza (stack trace) en el servidor | Cliente / entrevista |
| UR-07 | Admin | Poder saber en términos de negocio qué envío falló y a qué destinatario (p. ej. «falló el envío al correo X») | Cliente / entrevista |
| UR-08 | Lector | Ante un error, recibir un código de incidencia que permita reportarlo, sin ver información técnica sensible | Cliente / entrevista |
| UR-09 | Lector | Crear clientes con ayuda del sistema para evitar duplicados y datos incorrectos, pudiendo dejar pendientes datos no esenciales | Cliente / entrevista |
| UR-10 | Lector / Admin | Entender rápidamente la historia de un cliente (línea temporal) y acceder rápido a los clientes habituales (favoritos y recientes) | Cliente / entrevista |
| UR-11 | Lector | Recibir alertas útiles y configurar sus preferencias personales de notificación (p. ej. un resumen por correo) | Cliente / entrevista |
| UR-12 | Admin | Configurar las reglas corporativas que cambian el comportamiento del sistema (alertas, objetivos del dashboard, umbral de aprobaciones, festivos), según su impacto | Cliente / entrevista |
| UR-13 | Lector con permiso | (Ampliación) Seleccionar la entidad emisora de una factura cuando haya varias | Cliente / entrevista |
| UR-14 | Lector con permiso explícito | Pausar los recordatorios de impago de un caso concreto indicando el motivo | Cliente / entrevista |
| UR-15 | Lector | Gestionar el estado de un cliente y sus recurrencias recibiendo avisos o bloqueos claros y explicados | Cliente / entrevista |
| UR-16 | Solicitante / Aprobador | Consultar el estado de sus solicitudes y el comentario del aprobador (solicitante) y disponer de una bandeja de pendientes (aprobador) | Cliente / entrevista |

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
| FR-14 | El sistema deberá registrar en el servidor cada error técnico con un identificador único de incidencia, la fecha y su traza técnica | UR-06 |
| FR-15 | El sistema deberá registrar en una auditoría de negocio, separada de los logs técnicos, los fallos y eventos relevantes con un mensaje comprensible para personal no técnico (p. ej. «falló el envío al correo X») | UR-07 |
| FR-16 | Ante un error, el sistema deberá mostrar al usuario solo un mensaje genérico y el código de incidencia, sin información técnica sensible, de modo que el código permita localizar el error en el log técnico (FR-14) | UR-08, UR-06 |
| FR-17 | Antes de crear un cliente, el sistema deberá buscar posibles duplicados (p. ej. por correo, identificador fiscal y nombre) y avisar al usuario | UR-09 |
| FR-18 | El sistema deberá validar los datos del cliente: formato del correo, identificador fiscal según el país cuando este se conozca, campos obligatorios y normalización de espacios | UR-09 |
| FR-19 | El sistema deberá distinguir los datos obligatorios para emitir una factura de los datos convenientes para la ficha comercial: permitirá guardar un cliente sin estos últimos (p. ej. teléfono pendiente) pero no emitirle facturas mientras falten los primeros, indicando cuáles | UR-09 |
| FR-20 | El sistema deberá mostrar para cada cliente una línea temporal con los eventos de negocio relevantes: alta, cambios de responsable, cambios de tarifa, facturas emitidas, pagos, incidencias, pausas, baja y reactivación, sin registrar cada acción de interfaz | UR-10 |
| FR-21 | El sistema deberá permitir marcar clientes como favoritos y mostrar los recientes, limitados a los clientes a los que el usuario tiene acceso | UR-10 |
| FR-22 | El sistema deberá cerrar la sesión tras un periodo de inactividad, exigir autenticarse de nuevo y permitir también el cierre de sesión manual | BR-05 |
| FR-23 | El sistema deberá invalidar de inmediato las sesiones activas de un usuario cuando un Admin lo desactive | BR-05 |
| FR-24 | El sistema deberá aplicar una protección básica ante intentos de login fallidos repetidos, sin causar bloqueos prolongados a usuarios legítimos | BR-05 |
| FR-25 | El sistema deberá tratar los datos fiscales del emisor como configuración, no fijos en el código | UR-12 |
| FR-26 | (Ampliación) El sistema deberá permitir varias entidades emisoras con series de facturación y cuentas independientes, y que un usuario con permiso seleccione la entidad de cada factura | UR-13 |
| FR-27 | (Ampliación) El sistema deberá permitir clasificar facturas en un proyecto o centro de coste a efectos de informes y administrar esas categorías (crear, renombrar, desactivar) | UR-12 |
| FR-28 | (Ampliación) El sistema deberá conservar la clasificación histórica de las facturas al renombrar o desactivar una categoría | UR-12 |
| FR-29 | El sistema deberá permitir al Admin configurar un objetivo mensual o anual de facturación o cobro y mostrar su progreso en el dashboard | UR-12, FR-04 |
| FR-30 | El sistema deberá conservar el objetivo vigente en cada periodo y dejar constancia de sus cambios, para evitar comparaciones engañosas | UR-12 |
| FR-31 | El sistema deberá permitir configurar alertas mediante parámetros sobre un conjunto cerrado de condiciones (p. ej. factura vencida más de N días, cliente con más de X euros pendientes, N fallos de envío, revisión anual sin resolver tras N días), además de las notificaciones de impago ya definidas en FR-05 | UR-11, UR-12 |
| FR-32 | El sistema deberá permitir a cada usuario configurar preferencias personales de notificación (p. ej. resumen por correo) sin afectar al comportamiento general del sistema | UR-11 |
| FR-33 | El sistema deberá permitir modificar las reglas corporativas solo a Admin o, si se crea ese rol, a Superadmin (ver Q-02), según el impacto del cambio | UR-12 |
| FR-34 | El sistema no permitirá a un Lector desactivar los recordatorios de impago de sus clientes; solo podrá pausarlos en un caso concreto si tiene permiso explícito, registrando el motivo | UR-14 |
| FR-35 | El sistema deberá disponer de un calendario laboral configurable para España (fines de semana y festivos administrables) y trasladar al siguiente día laborable las comunicaciones y recordatorios que caigan en día no laborable | UR-11, UR-12 |
| FR-36 | (Ampliación) El sistema deberá permitir emitir una factura en una moneda distinta de la base y conservarla en esa moneda | BR-08 |
| FR-37 | (Ampliación) El sistema deberá registrar un tipo de cambio para informes y convertir a moneda base los importes del dashboard, distinguiendo el importe original del convertido | BR-08 |
| FR-38 | El sistema deberá gestionar los estados del cliente que afectan a la facturación: pendiente de activación, activo, pausado, baja solicitada, inactivo y bloqueado administrativamente | UR-15 |
| FR-39 | El sistema deberá permitir cambiar el estado de un cliente (incluido el bloqueo administrativo) y registrar el cambio en su línea temporal | UR-15 |
| FR-40 | El sistema deberá aplicar solo las consecuencias definidas para cada cambio de estado e informar de ellas al usuario antes de confirmarlo | UR-15 |
| FR-41 | El sistema deberá bloquear la activación de una recurrencia si faltan datos fiscales, explicando qué datos faltan | UR-15 |
| FR-42 | El sistema deberá permitir dar de baja una recurrencia aunque existan facturas pendientes, manteniendo activo el recobro de esas facturas | UR-15, FR-13 |
| FR-43 | El sistema deberá advertir al usuario cuando intente eliminar el único contacto de facturación de un cliente activo | UR-15 |
| FR-44 | El sistema deberá ofrecer una bandeja con las solicitudes pendientes de aprobación | UR-16 |
| FR-45 | El sistema deberá permitir al solicitante consultar el estado de sus solicitudes (pendiente, aprobada, rechazada) y el comentario del aprobador | UR-16 |
| FR-46 | El sistema deberá permitir configurar el umbral económico a partir del cual se exige separación de funciones y, cuando aplique, impedir que el aprobador apruebe su propia solicitud | UR-12, UR-16 |



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
| NFR-08 | Seguridad (acceso) | El servidor deberá comprobar la autorización en cada acceso a un cliente o factura, sea cual sea la vía (URL directa, favoritos, recientes o menú) | Al abrir con otro usuario una URL copiada de un cliente o factura que no tiene asignada, el sistema deniega el acceso sin mostrar datos en el 100 % de los casos probados |
| NFR-09 | Seguridad / Configuración | La configuración y los secretos (credenciales, claves) deberán mantenerse fuera del código fuente | Ningún secreto está en el código fuente y cambiar de entorno solo requiere cambiar la configuración |
| NFR-10 | Calidad (pruebas) | Las reglas económicas importantes deberán tener pruebas automáticas | Hay al menos una prueba automática por regla: prorrateo, generación mensual idempotente, vencimientos, cálculo de impuestos, permisos, conciliación y cambios de estado; todas se superan antes de cada despliegue |
| NFR-11 | Calidad (aceptación) | Las reglas económicas deberán tener escenarios de aceptación comprensibles para personal de administración | Cada regla económica clave tiene al menos un escenario en lenguaje de negocio (dado / cuando / entonces) que una persona de administración puede leer y validar |
| NFR-12 | Documentación | Deberá existir documentación breve y útil que cubra: uso, despliegue y mantenimiento, modelo de datos y API, decisiones importantes, configuración, restauración de copias y reglas de negocio | Existen los siete apartados; una persona ajena al desarrollo puede desplegar el sistema siguiendo solo la documentación; la API está documentada en formato estructurado legible por máquina (ver A-07) |
| NFR-13 | Restricción tecnológica | Las tecnologías deberán ser software libre o razonablemente accesible y desplegables por Turbine | Cada tecnología y librería tiene licencia libre o coste de acceso razonable, documentado |
| NFR-14 | Despliegue | Deberá existir al menos un entorno de desarrollo/pruebas y una forma reproducible de poner en marcha producción | Producción puede levantarse desde cero en una máquina limpia siguiendo un procedimiento documentado o automatizado, sin depender del equipo personal de nadie |
| NFR-15 | Fiabilidad | Deberá ser posible hacer copias de seguridad y restaurarlas (ver A-08 y FR-07) | Un procedimiento documentado permite restaurar una copia en un entorno de pruebas y recuperar los datos |
| NFR-16 | Compatibilidad | La aplicación deberá funcionar en navegadores modernos habituales de escritorio y en los navegadores actuales de Android e iOS | Las funcionalidades principales se verifican en las versiones actuales de Chrome, Firefox, Edge y Safari y en los navegadores móviles de Android e iOS; no se exige Internet Explorer |
| NFR-17 | Interoperabilidad | Los PDF deberán abrirse de forma estándar y los CSV exportados no deberán mostrar caracteres rotos | Los PDF se abren en visores estándar; un CSV con tildes y nombres extranjeros (p. ej. «José Núñez», «Müller») se muestra correctamente al abrirlo con herramientas habituales |
| NFR-18 | Exactitud y consistencia | Los importes deberán manejarse con precisión decimal y ser coherentes: los almacenados, los del PDF y los del dashboard derivan de los mismos datos | Ningún cálculo pierde ni crea céntimos (el total coincide con la suma de líneas e impuestos); el importe del PDF coincide con el almacenado y la suma del dashboard con la suma de las facturas, verificado con pruebas |

## 5. Reglas de negocio

Reglas del dominio o de la organización que condicionan el comportamiento del sistema
pero que no representan por sí mismas una funcionalidad.

| ID | Regla |
|---|---|
| RB-01 | Favoritos y recientes son solo atajos de navegación: no conceden ni sustituyen permisos de acceso |
| RB-02 | Un Lector solo puede acceder a los clientes y facturas que tiene asignados |
| RB-03 | Una factura se emite en una moneda y se conserva en ella; el total legal no varía con el tipo de cambio y el importe convertido solo se usa para informes |
| RB-04 | La regla de redondeo debe estar definida, documentada y aplicarse de forma uniforme |
| RB-05 | Cuando aplica la separación de funciones, el aprobador no puede ser el solicitante; se aplica a operaciones sensibles por encima del umbral y por debajo se admite más flexibilidad |
| RB-06 | Los objetivos del dashboard son informativos, no alteran facturas y solo los gestiona el Admin |

## 6. Suposiciones y decisiones de análisis

En esta sección se documentarán las decisiones tomadas por el equipo cuando la
información proporcionada por el cliente no sea suficiente para determinar unívocamente
un requisito.

| ID | Suposición o decisión | Justificación | Requisitos afectados |
|---|---|---|---|
| A-01 | «Usuario» es cualquier trabajador de Turbine que usa la aplicación (Admin o Lector). «Cliente» es una empresa externa a la que Turbine factura: aparece en el sistema como un registro, pero no accede a la aplicación. Por tanto, el código de incidencia y los mensajes de error se muestran a trabajadores de Turbine, a quienes tampoco se les enseña información técnica sensible | La aplicación es interna; solo la usan trabajadores de Turbine | UR-08, FR-16 |
| A-02 | El login con cuenta interna, los dos roles (Admin y Lector), la asignación de clientes a lectores y la web responsive ya están recogidos (UR-01, UR-03, FR-03, FR-06, NFR-02, NFR-07) y no se repiten aquí | Evitar duplicados al unir las partes del equipo | FR-23, NFR-16 |
| A-03 | Los duplicados se detectan por correo, identificador fiscal o nombre y se resuelven con un aviso, sin bloqueo automático | El cliente pide «duplicados razonables» sin concretar | FR-17 |
| A-04 | La línea temporal es una vista única desde la ficha del cliente; la auditoría completa queda aparte | El cliente dice que la auditoría completa «puede estar aparte» y no fija el número de pasos | FR-15, FR-20 |
| A-05 | Se propone un tiempo de inactividad por defecto de 30 minutos, configurable | «Razonable» no es verificable; propuesta pendiente de confirmar | FR-22 |
| A-06 | La protección ante logins fallidos se interpreta como medidas poco intrusivas (aviso a un Admin y/o retardo corto progresivo), no bloqueos prolongados | El cliente pide protección básica sin «infierno de bloqueos» | FR-24 |
| A-07 | «Documentación corta» se interpreta como guías de tareas habituales, sin manual exhaustivo; la documentación estructurada de la API se prevé por un posible agente de IA | El cliente rechaza un manual de 200 páginas | NFR-12 |
| A-08 | Se asume que el sistema permite copias de seguridad y su restauración, en línea con FR-07 (copia del registro de movimientos y accesos) | El cliente pide documentar cómo restaurar un backup | NFR-15, FR-07 |
| A-09 | Varias entidades emisoras, centros de coste y moneda extranjera son ampliaciones: no obligatorias en el primer despliegue, pero el diseño no debe impedirlas | El cliente no pide multiempresa completa ahora | FR-26, FR-27, FR-28, FR-36, FR-37 |
| A-10 | Las alertas configurables se limitan al conjunto cerrado de condiciones citadas, con parámetros (días, importes, número de fallos) | El cliente descarta un motor BPMN | FR-31 |
| A-11 | El calendario laboral desplaza la comunicación o recordatorio, no el vencimiento legal de la factura | El cliente habla de mover «la comunicación»; ver Q-03 | FR-35 |
| A-12 | Los prospectos importados quedan fuera (pertenecen al CRM); solo se representan los estados que afectan a la facturación | El cliente dice que lo mínimo son los estados que afectan a la facturación | FR-38 |
| A-13 | Eliminar el único contacto de facturación genera una advertencia, no un bloqueo; por debajo del umbral de aprobaciones no se exige separación de funciones | El cliente dice «avisad» y «menos rígidos para cosas pequeñas» | FR-43, FR-46 |
| A-14 | Los objetivos se guardan por periodo (valor vigente) con registro de sus cambios | El cliente pide guardar el vigente o dejar constancia del cambio | FR-30 |

## 7. Dudas, ambigüedades y cuestiones pendientes

Se incluirán aquí aspectos de la información proporcionada por el cliente que sean
ambiguos, incompletos, contradictorios o que requieran aclaración.

| ID | Cuestión | Origen | Impacto | Estado / resolución |
|---|---|---|---|---|
| Q-01 | ¿Qué tiempo de inactividad es «razonable» y qué protección concreta se espera ante logins fallidos? | Entrevista | FR-22, FR-24 | Pendiente (propuestas en A-05 y A-06) |
| Q-02 | El FR-06 solo define Admin y Lector: ¿se añade un rol Superadmin («según impacto») o lo gestiona todo el Admin? ¿Qué cambios requieren cada nivel y qué rol puede pausar recordatorios de un caso? | Entrevista | FR-06, FR-33, FR-34 | Pendiente |
| Q-03 | Si el vencimiento o recordatorio cae en festivo, ¿se mueve solo la comunicación o también el vencimiento de la factura? | Entrevista | FR-35 | Pendiente (supuesto en A-11) |
| Q-04 | ¿Por qué canal y a quién se envían las alertas? | Entrevista | FR-31 | Pendiente |
| Q-05 | ¿Qué es una «revisión anual» y cuándo se considera resuelta? | Entrevista | FR-31 | Pendiente (puede estar en páginas anteriores) |
| Q-06 | ¿Quién registra el tipo de cambio (manual o fuente externa) y cuál es la moneda base? | Entrevista | FR-37 | Pendiente (ampliación) |
| Q-07 | ¿Qué regla de redondeo concreta se aplica (por línea, por total)? | Entrevista | NFR-18, RB-04 | Pendiente |
| Q-08 | ¿Qué consecuencias tiene cada estado del cliente (p. ej. si un cliente pausado sigue generando facturas)? | Entrevista | FR-40 | Pendiente |
| Q-09 | ¿Qué significa «según lo acordado» en la baja efectiva de una recurrencia con facturas pendientes? ¿Cómo encaja con FR-13? | Entrevista | FR-42 | Pendiente |
| Q-10 | ¿Qué operaciones requieren aprobación y cuáles separación de funciones? ¿Cuál es el umbral inicial y quién lo configura? | Entrevista | FR-46, RB-05 | Pendiente |
| Q-11 | ¿El objetivo del dashboard es de facturación, de cobro o de ambos? | Entrevista | FR-29 | Pendiente (se contemplan ambos) |
| Q-12 | ¿Entra en el primer despliegue la documentación estructurada para un futuro agente de IA? | Entrevista | NFR-12 | Pendiente |
| Q-13 | ¿Se incluyen los prospectos importados en los estados del cliente? | Entrevista | FR-38 | Resuelta: no, pertenecen al CRM (A-12) |


## 8. Trazabilidad

La tabla permitirá comprobar la relación entre las necesidades de negocio,
los requisitos de usuario y los requisitos del sistema.

| Requisito de negocio | Requisito de usuario | Requisito funcional / no funcional |
|---|---|---|
| BR-01 | UR-01 | FR-01, FR-02 |
| BR-02 | UR-03 | FR-07, NFR-02 |
| BR-03 | UR-06, UR-07, UR-08 | FR-14, FR-15, FR-16 |
| BR-04 | UR-09, UR-10 | FR-17, FR-18, FR-19, FR-20, FR-21 |
| BR-05 | — | FR-22, FR-23, FR-24, NFR-08 |
| BR-06 | — | NFR-10, NFR-11, NFR-12, NFR-18 |
| BR-07 | — | NFR-09, NFR-13, NFR-14, NFR-15, NFR-16, NFR-17 |
| BR-08 | UR-12, UR-13 | FR-25, FR-26, FR-27, FR-28, FR-36, FR-37 |
| BR-09 | UR-11, UR-12, UR-14 | FR-29, FR-30, FR-31, FR-32, FR-33, FR-34, FR-35 |
| BR-10 | UR-15, UR-16 | FR-38, FR-39, FR-40, FR-41, FR-42, FR-43, FR-44, FR-45, FR-46 |

## 9. Glosario

| Término | Definición |
|---|---|
| Usuario | Trabajador de Turbine que utiliza el sistema (Admin o Lector) |
| Cliente | Empresa externa a la que Turbine presta servicios y factura; no accede a la aplicación |
| Admin | Usuario con permisos de configuración, asignación de clientes y gestión de usuarios |
| Lector | Usuario que gestiona las facturas de los clientes que le ha asignado un Admin |
| Código de incidencia | Identificador único que se muestra al usuario ante un error para localizarlo en el log técnico |
| Stack trace | Traza técnica del error que indica dónde se produjo en el código; solo debe verse en el servidor |
| Auditoría de negocio | Registro de eventos y fallos en términos comprensibles para personal administrativo, distinto del log técnico |
| Línea temporal | Vista del historial del cliente con sus eventos relevantes de negocio |
| Recurrencia | Facturación periódica automática de un cliente |
| Recobro | Proceso de reclamación y seguimiento de facturas pendientes de cobro |
| Entidad emisora | Sociedad de Turbine que emite las facturas, con sus datos fiscales, series y cuentas |
| Centro de coste / proyecto | Etiqueta para clasificar facturas a efectos de informes; no es contabilidad completa |
| Moneda base | Moneda en la que se consolidan los importes del dashboard |
| Prorrateo | Reparto proporcional de un importe según el tiempo de servicio efectivo |
| Generación mensual idempotente | Generación de las facturas del mes que, si se repite, no crea duplicados |
| Conciliación | Contraste de los cobros recibidos con las facturas emitidas para saber cuáles están pagadas |
| Separación de funciones | Regla por la que quien solicita una operación no puede aprobarla |
| Escenario de aceptación | Descripción en lenguaje de negocio de una situación y su resultado esperado, que permite validar una regla |
| Secretos | Credenciales, claves y otros datos sensibles que no deben estar en el código |
