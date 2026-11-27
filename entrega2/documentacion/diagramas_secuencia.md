# Diagramas UML de secuencia

## Selección de escenarios

Para modelar el comportamiento dinámico del núcleo de **FACTURBINEH**, se han seleccionado la totalidad de los **26 casos de uso funcionales y automatizados (`CU-01` a `CU-26`)**, complementados por un subdiagrama transversal reutilizable (`SD-00`) que encapsula la validación de sesiones JWT y el control de acceso basado en roles (RBAC). Esta cobertura completa permite especificar tanto las interacciones síncronas desde la interfaz web y la API externa como los procesos batch desatendidos y webhooks asíncronos:

* **Seguridad, autenticación y gobierno (`SD-00`, `CU-01` a `CU-04`, `CU-11`, `CU-23`, `CU-24`, `CU-26`):** Flujos críticos de inicio de sesión con bloqueo por intentos fallidos, recuperación de credenciales en dos fases, administración RBAC, delegación temporal de cartera, aprobación de excepciones comerciales, parámetros globales, auditoría inmutable y cierre de ejercicios fiscales.
* **Gestión comercial y ciclo de vida del cliente (`CU-05` a `CU-10`):** Alta con validación de duplicidad de NIF e impacto contable, actualización fiscal, pausa/reactivación sincronizada con pasarela de pagos, baja con anonimización RGPD condicionada a ausencia de deudas, fusión de duplicados y revisión anual automática de tramos de facturación.
* **Operativa de facturación, documentos y cobros (`CU-12` a `CU-17`, `CU-25`):** Emisión recurrente mediante cron, emisión puntual con captura centralizada de excepciones, modificación de borradores frente a generación de facturas rectificativas inmutables, exportación tabular/PDF, conciliación automática por webhook HMAC o transferencia manual, condonación de deudas con abono opcional y consultas externas autorizadas.
* **Incidencias, notificaciones e inteligencia de negocio (`CU-18` a `CU-22`):** Apertura y cierre de incidencias con pausa automática de reclamaciones de cobro, edición de plantillas, recordatorios periódicos de impago, dashboards ejecutivos de KPIs y exportación de paquetes ZIP para gestoría.

## Descripción de SD-00

1. El cliente (`PanelWeb` o `ClienteAPI`) envía una petición protegida adjuntando el token JWT y el rol requerido hacia `RBACGuard`.
2. `RBACGuard` delega la comprobación en `SessionManager`, que consulta la vigencia del token y los permisos asociados en `PostgreSQL`.
3. Si la sesión es válida y el rol coincide con el exigido, se autoriza la ejecución del caso de uso; en caso contrario, se devuelve un error `403 Forbidden`.

## Descripción de CU-01

1. El operativo introduce sus credenciales en `PanelWeb`, que remite la solicitud de autenticación a `AuthController` y este a `SessionManager`.
2. `SessionManager` recupera el registro del usuario desde `PostgreSQL` para verificar contraseña y estado de bloqueo.
3. Si las credenciales son válidas y la cuenta está activa, se instancia `: Usuario`, se genera el token JWT, se registra la sesión en `PostgreSQL`, se almacena el token en `PanelWeb` y se muestra el dashboard (`200 OK`); de lo contrario, se incrementa el contador de intentos fallidos en `PostgreSQL` y se devuelve `401 Unauthorized`.

## Descripción de CU-02

1. **Fase 1 (Solicitud):** El usuario solicita restablecer su contraseña en `PanelWeb`; `AuthController` y `SessionManager` comprueban en `PostgreSQL` que el usuario esté activo y generan un token temporal con expiración (`200 OK`).
2. **Fase 2 (Restablecimiento):** El usuario introduce el token recibido y la nueva contraseña; `SessionManager` verifica en `PostgreSQL` la vigencia del token.
3. Si el token es válido, se actualiza el hash de la contraseña, se desbloquea la cuenta y se redirige al login (`200 OK`); si ha caducado o es incorrecto, se devuelve `400 Bad Request`.

## Descripción de CU-03

1. El administrador envía los datos de alta o edición de usuario y rol desde `PanelWeb`, validando previamente su sesión con rol `Admin` (`SD-00`).
2. `RBACGuard` invoca a `ConfigApr`, que instancia `: Usuario` y persiste la operación en `PostgreSQL`.
3. `ConfigApr` registra de forma asíncrona el evento `GESTION_USUARIO_ROL` mediante `AuditService` en `PostgreSQL` y devuelve `200 OK` a `PanelWeb`.

## Descripción de CU-04

1. El administrador selecciona el cliente, el operativo destino y la fecha de finalización en `PanelWeb`, previa validación de sesión `Admin` (`SD-00`).
2. `RBACGuard` delega en `GestorCLI`, que actualiza la asignación y delegación temporal en `PostgreSQL`.
3. `GestorCLI` envía de forma asíncrona el evento `ASIGNACION_CLIENTE` a `AuditService` y devuelve `200 OK` para actualizar la vista de cartera.

## Descripción de CU-05

1. El operativo introduce los datos del nuevo cliente en `PanelWeb`, verificándose su sesión con rol `Operativo` (`SD-00`).
2. `GestorCLI` consulta en `PostgreSQL` que el NIF no esté registrado previamente.
3. Si los datos son válidos y el NIF está disponible, se instancia `: Cliente`, se persiste en `PostgreSQL`, se notifica a `SrvFact` para inicializar su cuenta contable, se audita `ALTA_CLIENTE` en `AuditService` y se responde `201 Created`; si existe duplicidad, se devuelve `400 Bad Request`.

## Descripción de CU-06

1. El operativo envía las modificaciones de contactos y datos fiscales desde `PanelWeb`, previa validación de rol `Operativo` (`SD-00`).
2. `GestorCLI` comprueba en `PostgreSQL` que el nuevo email o NIF no entren en conflicto con otros registros.
3. Si no hay conflictos, actualiza la ficha en `PostgreSQL`, audita `ACTUALIZACION_FISCAL_CONTACTO` en `AuditService` y devuelve `200 OK`; en caso de colisión, retorna `409 Conflict`.

## Descripción de CU-07

1. El administrador solicita pausar o reactivar la suscripción de un cliente desde `PanelWeb`, validando el rol `Admin` (`SD-00`).
2. `GestorCLI` actualiza el estado en `PostgreSQL` y notifica el impacto contable a `SrvFact`, que sincroniza el cambio en la pasarela externa mediante `PayAdp`.
3. `GestorCLI` registra el evento `CAMBIO_ESTADO_SUSCRIPCION` en `AuditService` y responde `200 OK` a la interfaz.

## Descripción de CU-08

1. El administrador confirma la baja definitiva de un cliente en `PanelWeb`, previa validación de rol `Admin` (`SD-00`).
2. `GestorCLI` solicita a `SrvFact` comprobar en `PostgreSQL` si el cliente mantiene facturas impagadas.
3. Si existen deudas pendientes, se rechaza la operación con `422 Unprocessable Entity`; si no hay deudas, `SrvFact` cancela la suscripción vía `PayAdp`, `GestorCLI` anonimiza los datos personales (PII) en `PostgreSQL`, audita `BAJA_Y_ANONIMIZACION_GDPR` y responde `200 OK`.

## Descripción de CU-09

1. El administrador selecciona el cliente origen duplicado y el cliente destino en `PanelWeb`, validando permisos `Admin` (`SD-00`).
2. `GestorCLI` reasigna los contactos al cliente destino y marca el origen como fusionado en `PostgreSQL`.
3. `GestorCLI` invoca a `SrvFact` para transferir el historial de facturas y presupuestos en `PostgreSQL`, audita `FUSION_CLIENTES` en `AuditService` y devuelve `200 OK`.

## Descripción de CU-10

1. `CronJob` inicia la revisión anual invocando a `SrvFact`, que consulta en `PostgreSQL` los clientes activos y su volumen facturado para determinar cambios de tramo.
2. En bucle por cada cliente afectado, `CronJob` ordena a `GestorCLI` actualizar la tarifa en `PostgreSQL` y notificar el impacto contable a `SrvFact`, y solicita a `GestorNotif` comunicar el cambio (obteniendo los canales desde `GestorCLI`, despachando el aviso de forma asíncrona vía `NotifAdp` y registrando el envío en `PostgreSQL`).
3. Finalizado el recorrido, `CronJob` registra el evento `REVISION_ANUAL_TARIFAS` en `AuditService`.

## Descripción de CU-11

1. **Fase 1 (Solicitud):** El operativo registra una petición de excepción comercial desde `PanelWeb` (`SD-00` con rol `Operativo`); `ConfigApr` la guarda en estado pendiente en `PostgreSQL`, audita `SOLICITUD_EXCEPCION` y devuelve `201 Created`.
2. **Fase 2 (Resolución):** El administrador aprueba o rechaza la excepción en `PanelWeb` (`SD-00` con rol `Admin`); `ConfigApr` actualiza su estado en `PostgreSQL` y, si es aprobada, invoca a `SrvFact` para registrar la regla contable del cliente.
3. `ConfigApr` audita `RESOLUCION_EXCEPCION` en `AuditService` y devuelve `200 OK`.

## Descripción de CU-12

1. `CronJob` dispara el ciclo periódico de facturación en `SrvFact`, que recupera de `PostgreSQL` los clientes facturables en la fecha de ejecución.
2. Para cada cliente, `SrvFact` genera y guarda la factura en `PostgreSQL`, emite su documento PDF mediante `GenDocs` en `Storage` y vincula la ruta; si el cobro automático está habilitado, procesa el cargo a través de `PayAdp` y actualiza el estado de pago en `PostgreSQL`.
3. `SrvFact` solicita a `GestorNotif` el envío de la factura al contacto preferido consultado en `GestorCLI` (despachando vía `NotifAdp`) y, al terminar el lote, audita `LOTE_FACTURACION_RECURRENTE` en `AuditService`.

## Descripción de CU-13

1. El operativo envía los datos de una nueva factura puntual o presupuesto desde `PanelWeb`, previa validación de rol `Operativo` (`SD-00`).
2. Si los datos fiscales y líneas son válidos, `SrvFact` instancia `: Documento`, lo persiste en `PostgreSQL`, genera el PDF con `GenDocs` en `Storage`, guarda la ruta, audita `EMISION_DOCUMENTO` en `AuditService` y retorna `201 Created`.
3. Si se produce una excepción técnica o de dominio, `GlobalExceptionFilter` intercepta el fallo, `TechnicalLogger` registra la traza en `LogFiles` con un identificador único y devuelve `400 / 500 Error`.

## Descripción de CU-14

1. El administrador solicita modificar o rectificar una factura existente desde `PanelWeb`, validando permisos `Admin` (`SD-00`).
2. `SrvFact` consulta la factura en `PostgreSQL`: si se encuentra en estado `Borrador`, actualiza directamente sus campos y regenera el PDF vía `GenDocs` en `Storage`; si ya está `Emitida` o `Pagada` (inmutable), instancia `: FacturaRectificativa`, la guarda en `PostgreSQL` y emite su nuevo PDF.
3. `SrvFact` registra `MODIFICACION_O_RECTIFICACION` en `AuditService` y devuelve `200 / 201 OK`.

## Descripción de CU-15

1. **Consulta filtrada:** El operativo aplica criterios de búsqueda en `PanelWeb` (`SD-00` con rol `Operativo`); `SrvFact` ejecuta la consulta paginada en `PostgreSQL` y devuelve `200 OK` con el listado.
2. **Descarga o exportación:** Si se solicita un PDF individual, `SrvFact` lo recupera de `Storage` a través de `GenDocs`; si se pide exportación tabular (Excel/CSV), extrae el conjunto de datos de `PostgreSQL` y `GenDocs` genera el fichero temporal en `Storage`, retornando el `FileStream` (`200 OK`).

## Descripción de CU-16

1. **Flujo 1 (Webhook automático):** `PayAdp` notifica un cobro recibido a `WebhookCtrl`, que verifica la firma HMAC, ordena a `SrvFact` conciliar y marcar la factura como pagada en `PostgreSQL`, audita `PAGO_CONCILIADO_WEBHOOK` en `AuditService` y responde `200 OK`.
2. **Flujo 2 (Conciliación manual):** El administrador registra una transferencia bancaria desde `PanelWeb` (`SD-00` con rol `Admin`); `SrvFact` contrasta la referencia en `PayAdp`, registra el pago en `PostgreSQL`, audita `PAGO_MANUAL_CONCILIADO` y devuelve `200 OK`.

## Descripción de CU-17

1. El administrador tramita la condonación de una factura impagada desde `PanelWeb`, validando sesión con rol `Admin` (`SD-00`).
2. `SrvFact` marca la factura como incobrable o perdonada en `PostgreSQL`; opcionalmente (`opt`), si se marca la generación de nota de crédito, instancia `: NotaAbono`, la persiste en `PostgreSQL` y genera su PDF en `Storage` mediante `GenDocs`.
3. `SrvFact` audita `PERDON_DEUDA` en `AuditService` y devuelve `200 OK`.

## Descripción de CU-18

1. **Fase 1 (Apertura):** El operativo registra una incidencia en `PanelWeb` (`SD-00` con rol `Operativo`); `GestorInc` verifica cliente y factura mediante `GestorCLI` y `SrvFact`, instancia `: Incidencia`, la guarda en `PostgreSQL`, ordena a `GestorNotif` pausar los recordatorios de cobro de la factura y audita `APERTURA_INCIDENCIA` (`201 Created`).
2. **Fase 2 (Resolución):** El operativo cierra la incidencia en `PanelWeb` (`SD-00` con rol `Operativo`); `GestorInc` actualiza su estado en `PostgreSQL`, reactiva los recordatorios en `GestorNotif`, audita `CIERRE_INCIDENCIA` y retorna `200 OK`.

## Descripción de CU-19

1. El administrador edita el contenido de una plantilla de notificación en `PanelWeb`, previa validación de rol `Admin` (`SD-00`).
2. `ConfigApr` delega la actualización en `SrvFact` y este en `GestorNotif`, que persiste los cambios en `PostgreSQL`.
3. `ConfigApr` registra de forma asíncrona `ACTUALIZACION_PLANTILLA` en `AuditService` y responde `200 OK`.

## Descripción de CU-20

1. `CronJob` inicia el proceso de reclamación de impagos invocando a `SrvFact`, que recupera de `PostgreSQL` las facturas vencidas.
2. En bucle por cada factura impagada, `GestorNotif` verifica en `PostgreSQL` que los recordatorios no estén pausados por incidencia; si están activos, consulta el contacto preferido a `GestorCLI`, despacha el aviso de forma asíncrona mediante `NotifAdp` e incrementa el contador de envíos en `PostgreSQL`.
3. `SrvFact` registra el evento `ENVIO_RECORDATORIOS_IMPAGO` en `AuditService` y retorna el resumen de ejecución a `CronJob`.

## Descripción de CU-21

1. El administrador solicita la carga del cuadro de mando ejecutivo indicando el periodo en `PanelWeb`, previa validación de rol `Admin` (`SD-00`).
2. `RBACGuard` obtiene los indicadores de facturación y morosidad desde `SrvFact` y los indicadores de cartera desde `GestorCLI` (consultando ambos en `PostgreSQL`).
3. `PanelWeb` recibe ambos conjuntos de métricas (`200 OK`) y renderiza los gráficos del dashboard.

## Descripción de CU-22

1. El administrador solicita el paquete documental trimestral para gestoría desde `PanelWeb`, validando permisos `Admin` (`SD-00`).
2. `RBACGuard` invoca a `GestoriaFac`, que delega en `GenDocs` la recopilación y compresión de los PDFs del periodo desde `Storage`.
3. `PanelWeb` recibe el `FileStream` resultante (`200 OK`) e inicia la descarga del archivo ZIP.

## Descripción de CU-23

1. El superadministrador actualiza parámetros globales del sistema desde `PanelWeb`, previa validación de rol `Superadmin` (`SD-00`).
2. `ConfigApr` persiste la configuración en `PostgreSQL` y registra de forma asíncrona el evento `CONFIG_GLOBAL_ACTUALIZADA` en `AuditService`.
3. Se devuelve `200 OK` con la configuración actualizada a `PanelWeb`.

## Descripción de CU-24

1. El superadministrador consulta los registros de trazabilidad aplicando filtros en `PanelWeb`, validando el rol `Superadmin` (`SD-00`).
2. `ConfigApr` solicita los logs inmutables a `AuditService`, que ejecuta la consulta paginada en `PostgreSQL`.
3. `PanelWeb` recibe la página de eventos (`200 OK`) y renderiza la tabla de auditoría.

## Descripción de CU-25

1. Un sistema externo (`ClienteAPI`) envía una consulta autorizada mediante token JWT a `RBACGuard`, que valida la sesión y permisos con `SessionManager` en `PostgreSQL`.
2. `RBACGuard` invoca a `SrvFact`, que consulta el saldo y facturas del cliente en `PostgreSQL` y registra el evento `CONSULTA_EXTERNA_ESTADO_CUENTA` en `AuditService`.
3. `RBACGuard` responde `200 OK` con el estado de cuenta al sistema externo.

## Descripción de CU-26

1. El superadministrador ejecuta el cierre de un ejercicio fiscal desde `PanelWeb`, previa validación de rol `Superadmin` (`SD-00`).
2. `ConfigApr` ordena a `SrvFact` bloquear las facturas del año en `PostgreSQL` y generar el archivo consolidado de cierre mediante `GenDocs` en `Storage`.
3. `ConfigApr` marca el periodo como cerrado en `PostgreSQL`, audita `CIERRE_PERIODO_FISCAL` en `AuditService` y devuelve `200 OK`.

## Trazabilidad

| Diagrama | Caso de uso | Participantes / responsabilidades |
| :---- | :---- | :---- |
| `SD-00.puml` | Subdiagrama de seguridad JWT/RBAC | `PanelWeb` (emisor), `RBACGuard` (interceptación API), `SessionManager` (verificación token/rol), `PostgreSQL` (persistencia de sesiones). |
| `CU-01.puml` | CU-01: Iniciar Sesión en la Plataforma | `Operativo` (actor), `PanelWeb` (UI/almacenamiento JWT), `AuthController` (entrada auth), `SessionManager` (validación/bloqueo/JWT), `Usuario` (entidad), `PostgreSQL` (datos usuario/sesión). |
| `CU-02.puml` | CU-02: Recuperar y Restablecer Contraseña | `Operativo` (actor), `PanelWeb` (formularios recuperación), `AuthController` (controlador auth), `SessionManager` (gestión token temporal y hash), `PostgreSQL` (tokens y credenciales). |
| `CU-03.puml` | CU-03: Gestión de Usuarios y Roles | `Admin` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `ConfigApr` (gobierno usuarios/roles), `Usuario` (entidad), `AuditService` (trazabilidad), `PostgreSQL`. |
| `CU-04.puml` | CU-04: Asignación y Delegación Temporal de Clientes | `Admin` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `GestorCLI` (asignaciones y delegaciones), `AuditService`, `PostgreSQL`. |
| `CU-05.puml` | CU-05: Alta de Nuevo Cliente y Registro Comercial | `Operativo` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `GestorCLI` (validación NIF y alta), `Cliente` (entidad), `SrvFact` (cuenta contable), `AuditService`, `PostgreSQL`. |
| `CU-06.puml` | CU-06: Gestión de Contactos e Información Fiscal | `Operativo` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `GestorCLI` (validación conflictos y actualización), `AuditService`, `PostgreSQL`. |
| `CU-07.puml` | CU-07: Pausa Temporal y Reactivación de Suscripción | `Admin` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `GestorCLI` (estado suscripción), `SrvFact` (impacto contable), `PayAdp` (pasarela externa), `AuditService`, `PostgreSQL`. |
| `CU-08.puml` | CU-08: Tramitar Baja Comercial y Anonimización | `Admin` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `GestorCLI` (anonimización PII/baja), `SrvFact` (verificación deuda/cancelación), `PayAdp` (baja suscripción), `AuditService`, `PostgreSQL`. |
| `CU-09.puml` | CU-09: Fusión y Control de Clientes Duplicados | `Admin` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `GestorCLI` (fusión contactos), `SrvFact` (transferencia facturas/presupuestos), `AuditService`, `PostgreSQL`. |
| `CU-10.puml` | CU-10: Revisión Anual y Cambio de Rango de Facturación | `CronJob` (orquestador batch), `SrvFact` (cálculo tramos), `GestorCLI` (actualización tarifas/contactos), `GestorNotif` (gestión aviso), `NotifAdp` (envío), `AuditService`, `PostgreSQL`. |
| `CU-11.puml` | CU-11: Solicitud y Aprobación de Excepciones Comerciales | `Operativo` y `Admin` (actores), `PanelWeb`, `RBACGuard`, `SessionManager`, `ConfigApr` (flujo aprobación), `SrvFact` (aplicación regla contable), `AuditService`, `PostgreSQL`. |
| `CU-12.puml` | CU-12: Generación Automática de Facturas Recurrentes | `CronJob` (disparador), `SrvFact` (ciclo facturación), `GenDocs` (PDF), `PayAdp` (cobro automático), `GestorNotif` y `GestorCLI` (envío factura), `NotifAdp`, `AuditService`, `PostgreSQL`, `Storage`. |
| `CU-13.puml` | CU-13: Emisión de Facturas Puntuales y Presupuestos | `Operativo` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `SrvFact` (emisión), `Documento` (entidad), `GenDocs` (PDF), `AuditService`, `GlobalExceptionFilter` y `TechnicalLogger` (errores), `PostgreSQL`, `Storage`, `LogFiles`. |
| `CU-14.puml` | CU-14: Modificación y Rectificación de Facturas | `Admin` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `SrvFact` (edición borrador o rectificación), `FacturaRectificativa` (entidad), `GenDocs` (PDF), `AuditService`, `PostgreSQL`, `Storage`. |
| `CU-15.puml` | CU-15: Consulta, Descarga y Exportación de Facturas | `Operativo` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `SrvFact` (consultas/datasets), `GenDocs` (PDF/Excel/CSV), `PostgreSQL`, `Storage`. |
| `CU-16.puml` | CU-16: Integración y Conciliación de Pagos | `Admin` (actor), `PayAdp` (pasarela/webhook), `PanelWeb`, `WebhookCtrl` (validación HMAC), `RBACGuard`, `SessionManager`, `SrvFact` (conciliación), `AuditService`, `PostgreSQL`. |
| `CU-17.puml` | CU-17: Tramitar Perdón de Deudas y Gestión de Impagos | `Admin` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `SrvFact` (condonación), `NotaAbono` (entidad opcional), `GenDocs` (PDF abono), `AuditService`, `PostgreSQL`, `Storage`. |
| `CU-18.puml` | CU-18: Gestión de Incidencias de Clientes y Facturas | `Operativo` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `GestorInc` (ciclo incidencia), `GestorCLI` y `SrvFact` (vinculación), `Incidencia` (entidad), `GestorNotif` (pausa/reanudación avisos), `AuditService`, `PostgreSQL`. |
| `CU-19.puml` | CU-19: Gestión de Plantillas de Notificación | `Admin` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `ConfigApr` (gobierno plantillas), `SrvFact` (delegación), `GestorNotif` (persistencia plantilla), `AuditService`, `PostgreSQL`. |
| `CU-20.puml` | CU-20: Envío de Recordatorios de Pago e Impago | `CronJob` (disparador), `SrvFact` (facturas vencidas), `GestorNotif` (control pausa y plantillas), `GestorCLI` (contactos), `NotifAdp` (despacho), `AuditService`, `PostgreSQL`. |
| `CU-21.puml` | CU-21: Visualización de Dashboards e Indicadores | `Admin` (actor), `PanelWeb` (gráficos KPI), `RBACGuard`, `SessionManager`, `SrvFact` (KPIs facturación/morosidad), `GestorCLI` (KPIs clientes), `PostgreSQL`. |
| `CU-22.puml` | CU-22: Generación de Informes y Paquetes para Gestoría | `Admin` (actor), `PanelWeb` (descarga ZIP), `RBACGuard`, `SessionManager`, `GestoriaFac` (fachada exportación), `GenDocs` (empaquetado masivo), `Storage`. |
| `CU-23.puml` | CU-23: Configuración Parámetros Globales del Sistema | `Superadmin` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `ConfigApr` (parámetros globales), `AuditService`, `PostgreSQL`. |
| `CU-24.puml` | CU-24: Consulta de Registros de Auditoría | `Superadmin` (actor), `PanelWeb` (tabla logs), `RBACGuard`, `SessionManager`, `ConfigApr` (solicitud gobierno), `AuditService` (consulta logs inmutables), `PostgreSQL`. |
| `CU-25.puml` | CU-25: Integración API Externa (Consultas Autorizadas JWT) | `ClienteAPI` (sistema externo), `RBACGuard`, `SessionManager`, `SrvFact` (estado de cuenta), `AuditService`, `PostgreSQL`. |
| `CU-26.puml` | CU-26: Cierre de Periodos Fiscales y Archivo de Cierre | `Superadmin` (actor), `PanelWeb`, `RBACGuard`, `SessionManager`, `ConfigApr` (cierre ejercicio), `SrvFact` (bloqueo facturas), `GenDocs` (archivo cierre), `AuditService`, `PostgreSQL`, `Storage`. |

## Consistencia

Los diagramas de secuencia guardan una correspondencia directa y verificable tanto con la **arquitectura de componentes (`CODIGO_COMPONENTES_FINAL.md`)** como con el **modelo de dominio (diagrama de clases)** y las convenciones UML establecidas:

1. **Correspondencia estructural con el Diagrama de Componentes:**
   * **Capa de Presentación y Entrada (`PanelWeb`, `ClienteAPI`, `AuthController`, `RBACGuard`, `SessionManager`):** Toda interacción iniciada por usuarios humanos atraviesa `PanelWeb` hacia `AuthController` (en flujos públicos de autenticación `CU-01`/`CU-02`) o hacia `RBACGuard` (en peticiones autenticadas), delegando invariablemente la validación de sesión y roles (`Superadmin`, `Admin`, `Operativo`) en `SessionManager` mediante `SD-00`. Las integraciones máquina-a-máquina (`CU-25`) entran directamente a `RBACGuard` desde `ClienteAPI`, y las notificaciones entrantes de pasarela (`CU-16`) entran vía `WebhookCtrl`.
   * **Direccionalidad de dependencias en Núcleo de Negocio:** Se respetan estrictamente las relaciones dirigidas definidas en la arquitectura de componentes: `GestorCLI` notifica impactos contables hacia `SrvFact` (`CU-05`, `CU-07`, `CU-09`, `CU-10`), `ConfigApr` gobierna reglas y cierres sobre `SrvFact` (`CU-11`, `CU-19`, `CU-26`), `GestorInc` consulta a `GestorCLI` y vincula facturas con `SrvFact` (`CU-18`), `SrvFact` delega la generación documental en `GenDocs` (`CU-12` a `CU-15`, `CU-17`, `CU-26`) y `GestoriaFac` actúa como fachada sobre `GenDocs` sin acoplarse directamente a la base de datos (`CU-22`).
   * **Capa de Integración, Observabilidad y Persistencia:** Solo los componentes con enlace explícito a persistencia en la arquitectura (`SessionManager`, `GestorCLI`, `SrvFact`, `GestorInc`, `ConfigApr`, `GestorNotif`, `AuditService`) interactúan con `PostgreSQL`; exclusivamente `GenDocs` opera contra `Storage` y únicamente `TechnicalLogger` (invocado por `GlobalExceptionFilter` en `CU-13`) escribe sobre `LogFiles`.

2. **Justificación del nivel de abstracción frente al Diagrama de Clases:**
   * **Granularidad de componentes vs. entidades de dominio:** Mientras que el diagrama de clases detalla atributos, tipos y relaciones estáticas entre entidades (`Usuario`, `Cliente`, `Factura`, `FacturaRectificativa`, `Documento`, `NotaAbono`, `Incidencia`, `AuditEvent`), los diagramas de secuencia operan a nivel de **servicios y controladores de aplicación** (`SrvFact`, `GestorCLI`, `ConfigApr`) para mantener la legibilidad de los flujos distribuidos y evitar acoplar la vista arquitectónica a detalles internos de repositorios ORM.
   * **Instanciación explícita (`create`) y DTOs de frontera:** Para tender un puente entre ambas vistas, las operaciones de creación relevante (`CU-01`, `CU-03`, `CU-05`, `CU-13`, `CU-14`, `CU-17`, `CU-18`) explicitan la instanciación en tiempo de ejecución de las entidades del diagrama de clases (`: Usuario`, `: Cliente`, `: Documento`, `: FacturaRectificativa`, `: NotaAbono`, `: Incidencia`), mientras que los mensajes entre capas intercambian objetos de transferencia (`SesionDTO`, `ClienteDTO`, `DocumentoDTO`, `EstadoCuentaDTO`) propios de la frontera API/SPA.
   * **Semántica síncrona vs. asíncrona (`->>`):** Las operaciones transaccionales que condicionan la respuesta al usuario se modelan mediante llamadas síncronas con retorno explícito, mientras que el despacho de comunicaciones externas (`GestorNotif ->> NotifAdp`) y el registro de trazas de auditoría (`->> AuditService`) se representan como mensajes asíncronos para reflejar su naturaleza desacoplada y no bloqueante.