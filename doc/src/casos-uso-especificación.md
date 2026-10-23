### CU-01: Iniciar Sesión en la Plataforma

* **Identificador:** CU-01
* **Objetivo:** Autenticar a los usuarios corporativos en la plataforma FACTURBINEH mediante sus credenciales de empresa.
* **Actor o actores participantes:** Usuario (Superadministrador, Administrador, Usuario Operativo).
* **Precondiciones:** El usuario debe haber sido registrado previamente por la empresa con su correo corporativo y poseer una contraseña activa.
* **Flujo principal:**
1. El usuario accede a la página de inicio de sesión de la aplicación web.
2. El usuario introduce su correo corporativo y contraseña.
3. El usuario pulsa el botón "Iniciar Sesión".
4. El sistema valida el formato de las credenciales y verifica la autenticidad en la base de datos.
5. El sistema determina el rol (Superadministrador, Administrador o Usuario Operativo) y las asignaciones del usuario.
6. El sistema redirige al usuario a su Dashboard de trabajo según sus permisos y genera un token de sesión seguro.


* **Flujos alternativos:**
* *A1: Cierre de sesiones activas en caso de desactivación:* Si el usuario tiene una sesión previa abierta y su cuenta es desactivada por un administrador, la sesión previa se invalida inmediatamente (A1.1).


* **Flujos de excepción:**
* *E1: Credenciales incorrectas:* Si el usuario ingresa datos erróneos, el sistema muestra un mensaje claro ("Credenciales incorrectas") sin revelar cuál campo falló.
* *E2: Exceso de intentos fallidos:* Tras $N$ intentos fallidos consecutivos, el sistema bloquea temporalmente las solicitudes de esa IP/cuenta e informa al administrador sin realizar un bloqueo permanente del sistema.
* *E3: Dominio de correo no permitido:* Si el correo introducido no pertenece al dominio corporativo de Turbineh, el sistema deniega el acceso.


* **Postcondiciones:** El usuario inicia sesión de forma autenticada y cifrada; sus acciones quedan registradas en el sistema de auditoría.

---

### CU-02: Recuperar y Restablecer Contraseña

* **Identificador:** CU-02
* **Objetivo:** Permitir a un usuario restablecer su contraseña olvidada de forma segura mediante un enlace temporal por correo electrónico.
* **Actor o actores participantes:** Usuario, Sistema de Notificaciones por Correo.
* **Precondiciones:** El usuario debe estar previamente registrado en el sistema.
* **Flujo principal:**
1. El usuario hace clic en "¿Olvidaste tu contraseña?" en la pantalla de login.
2. El usuario ingresa su correo electrónico corporativo.
3. El sistema valida que el correo pertenece a una cuenta activa.
4. El sistema genera un enlace único de un solo uso con token cifrado y tiempo de caducidad restringido.
5. El sistema envía un correo electrónico al usuario con el enlace de recuperación.
6. El usuario hace clic en el enlace, accede a la pantalla de nuevo password, introduce la nueva contraseña y confirma.
7. El sistema actualiza la contraseña (almacenada mediante hashing seguro) e invalida el token.


* **Flujos alternativos:**
* *A1: Reenvío de token:* El usuario solicita un nuevo token si el anterior expiró antes de ser utilizado.


* **Flujos de excepción:**
* *E1: Token caducado o inválido:* El sistema notifica que el enlace ha expirado o ya fue utilizado y solicita iniciar nuevamente el proceso.
* *E2: Correo no registrado:* Por motivos de seguridad, el sistema muestra un mensaje neutro notificando el envío del correo de recuperación si el usuario existe.


* **Postcondiciones:** La contraseña del usuario es actualizada en la base de datos de forma cifrada y se invalidan las sesiones activas anteriores.

---

### CU-03: Gestión de Usuarios y Roles

* **Identificador:** CU-03
* **Objetivo:** Permitir a los Administradores y Superadministradores dar de alta, modificar y desactivar usuarios, especificando su rol operativo.
* **Actor o actores participantes:** Superadministrador, Administrador.
* **Precondiciones:** El usuario que ejecuta la acción debe tener rol de Administrador o Superadministrador y haber iniciado sesión.
* **Flujo principal:**
1. El Administrador accede a la sección "Gestión de Usuarios".
2. El Administrador pulsa "Crear Usuario".
3. El Administrador introduce el correo corporativo, nombre, apellidos y selecciona el rol (Superadministrador, Administrador o Usuario Operativo/Lector).
4. El sistema valida la unicidad del correo electrónico.
5. El Administrador guarda los datos.
6. El sistema envía un correo de bienvenida con las instrucciones para la asignación inicial de contraseña.


* **Flujos alternativos:**
* *A1: Desactivar usuario:* El Administrador selecciona un usuario existente y pulsa "Desactivar". El sistema cierra inmediatamente todas las sesiones abiertas de ese usuario y revoca sus accesos, manteniendo su histórico de acciones intacto.
* *A2: Editar permisos/rol:* El Administrador modifica el rol o datos del usuario sin alterar sus registros históricos de auditoría.


* **Flujos de excepción:**
* *E1: Intento de auto-desactivación o modificación de Superadministrador por un Administrador estándar:* El sistema deniega la acción por falta de jerarquía.


* **Postcondiciones:** El usuario queda registrado o actualizado en el sistema; todos los cambios se guardan en el registro de auditoría.

---

### CU-04: Asignación y Delegación Temporal de Clientes

* **Identificador:** CU-04
* **Objetivo:** Asignar clientes a responsables internos (Usuarios Operativos) o crear delegaciones temporales entre ellos durante ausencias o vacaciones.
* **Actor o actores participantes:** Administrador, Usuario Operativo (titular), Usuario Operativo (delegado).
* **Precondiciones:** Los usuarios y clientes deben existir y estar activos en la plataforma.
* **Flujo principal:**
1. El Administrador accede al perfil del cliente o a la matriz de asignación de clientes.
2. El Administrador selecciona el responsable interno principal del cliente.
3. El Administrador activa la opción "Delegación Temporal" si se requiere.
4. El Administrador define el usuario delegado y establece un rango de fechas (fecha inicio y fecha fin).
5. El sistema guarda la configuración y transfiere los permisos de gestión operativa al usuario delegado durante el intervalo estipulado, conservando al responsable principal como titular histórico.
6. Al llegar la fecha fin, la delegación expira automáticamente y el acceso regresa de forma exclusiva al titular principal.


* **Flujos alternativos:**
* *A1: Reasignación definitiva:* El Administrador cambia el responsable principal de un cliente. El nuevo responsable hereda todas las tareas pendientes e incidencias abiertas del cliente.


* **Flujos de excepción:**
* *E1: Intento de auto-asignación por parte de un Usuario Operativo:* El sistema bloquea la solicitud, ya que los Usuarios Operativos no pueden reasignarse clientes a sí mismos.


* **Postcondiciones:** Las tareas pendientes se redirigen al responsable activo/delegado y se audita el cambio en el historial del cliente.

---

### CU-05: Alta de Nuevo Cliente y Registro Comercial

* **Identificador:** CU-05
* **Objetivo:** Dar de alta a un cliente (empresa) en la plataforma con la menor cantidad de clics posible, minimizando errores y asegurando la integridad fiscal.
* **Actor o actores participantes:** Administrador, Usuario Operativo.
* **Precondiciones:** El usuario tiene permisos para crear clientes y los parámetros globales (monedas, impuestos, tarifas) están configurados.
* **Flujo principal:**
1. El usuario pulsa "Alta de Cliente".
2. El usuario introduce la Razón Social, Nombre Comercial e Identificador Fiscal (NIF/CIF/Tax ID).
3. El sistema realiza una comprobación automática de duplicados en tiempo real basada en el identificador fiscal.
4. El usuario indica el Domicilio Fiscal, País habitual, Moneda habitual (EUR, USD, etc.), Vencimiento por defecto, Sector, Etiquetas (ej.: Estratégico, Beta, Partner, Riesgo) y Tarifa/Plan de suscripción.
5. El usuario añade al menos un contacto principal de facturación.
6. El sistema muestra la vista previa del cálculo del prorrateo correspondiente al primer mes desde la fecha actual hasta fin de mes.
7. El usuario confirma el alta.
8. El sistema guarda la empresa y programa su facturación recurrente.


* **Flujos alternativos:**
* *A1: Guardado en borrador:* Si el usuario no completa el formulario, el sistema guarda un borrador automático para evitar la pérdida de datos ingresados.
* *A2: Seleccionar cobro al mes siguiente:* El usuario selecciona no cobrar prorrateo en el alta y comenzar la facturación recurrente estándar a partir del día 1 del mes siguiente.


* **Flujos de excepción:**
* *E1: Identificador Fiscal duplicado detectado:* El sistema alerta al usuario de que el cliente ya existe y bloquea la creación directa para evitar registros duplicados.
* *E2: Datos fiscales obligatorios faltantes:* Si faltan datos clave para la emisión de facturas, el cliente se guarda en estado "Pendiente de activación" y se notifica la imposibilidad de activar la recurrencia hasta completarlos.


* **Postcondiciones:** El cliente queda registrado en el sistema con su estado comercial activo y la regla de prorrateo fijada.

---

### CU-06: Gestión de Contactos e Información Fiscal

* **Identificador:** CU-06
* **Objetivo:** Administrar múltiples personas de contacto asociadas a un cliente y actualizar sus datos fiscales conservando la trazabilidad de cambios.
* **Actor o actores participantes:** Administrador, Usuario Operativo.
* **Precondiciones:** El cliente debe estar registrado previamente.
* **Flujo principal:**
1. El usuario accede a la ficha del cliente.
2. El usuario selecciona la pestaña "Contactos y Fiscalidad".
3. El usuario añade un nuevo contacto (Nombre, Cargo, Email, Teléfono, Canales autorizados: Email / WhatsApp) o edita los datos fiscales (Razón Social, NIF, Dirección).
4. El usuario marca al menos un contacto como "Contacto Principal de Facturación".
5. El usuario guarda los cambios.
6. El sistema valida la consistencia de los datos e incrementa el historial de auditoría de la ficha del cliente.


* **Flujos alternativos:**
* *A1: Desactivar contacto:* El usuario marca un contacto como inactivo. El sistema realiza un borrado lógico para mantener el histórico de envíos de facturas pasadas asociados a dicho contacto.


* **Flujos de excepción:**
* *E1: Borrado del único contacto de facturación activo:* El sistema detiene la operación e informa que un cliente activo debe poseer obligatoriamente al menos un contacto asignado para recibir facturas.


* **Postcondiciones:** Se actualiza la lista de contactos o datos fiscales; las facturas ya emitidas previamente no sufren alteraciones retroactivas.

---

### CU-07: Pausa Temporal y Reactivación de Suscripción

* **Identificador:** CU-07
* **Objetivo:** Pausar temporalmente la suscripción de un cliente por motivos justificados o reactivar un cliente pausado.
* **Actor o actores participantes:** Administrador, Usuario Operativo, Sistema.
* **Precondiciones:** El cliente debe estar en estado activo o pausado.
* **Flujo principal:**
1. El usuario accede a la suscripción del cliente y selecciona "Solicitar Pausa Temporal".
2. El usuario indica la Fecha de Inicio de Pausa, Fecha Estimada de Reactivación y el Motivo de la pausa.
3. El usuario confirma la operación.
4. El sistema cambia el estado del cliente a "Pausado", suspende la generación de facturas recurrentes durante dicho periodo y notifica al responsable interno.
5. Llegada la fecha de reactivación (o de forma manual), el sistema o el usuario reactivan la cuenta, reanudando la generación mensual e informando al responsable.


* **Flujos alternativos:**
* *A1: Pausa por incidencia:* Si la pausa deriva de una incidencia de cobro, el usuario puede marcar la opción de "Pausar notificaciones de impago relacionadas" para no agobiar al cliente durante la resolución.


* **Flujos de excepción:**
* *E1: Existencia de facturas pendientes previas a la pausa:* El sistema permite la pausa de la suscripción futura, pero mantiene activos los procesos de cobro de las facturas vencidas u omitidas previamente a la fecha de pausa.


* **Postcondiciones:** La suscripción no genera nuevas facturas recurrentes mientras dura la pausa y se registra la trazabilidad del motivo y fechas.

---

### CU-08: Tramitar Baja Comercial y Anonimización / Purgado Lógico

* **Identificador:** CU-08
* **Objetivo:** Procesar la cancelación de un cliente salvaguardando los datos requeridos por normativas fiscales y aplicando borrado lógico / anonimización cuando proceda.
* **Actor o actores participantes:** Administrador, Superadministrador, Sistema / API Externa.
* **Precondiciones:** El cliente debe estar registrado en el sistema.
* **Flujo principal:**
1. El Administrador selecciona "Tramitar Baja Comercial" (o la solicitud entra vía API externa de BrainOS).
2. El sistema verifica si existen facturas vencidas o saldo pendiente asociado al cliente.
3. El sistema muestra el desglose del saldo: si tiene deudas, marca la baja comercial cancelando las recurrencias futuras, pero conserva las facturas impagadas para su cobro.
4. El sistema ejecuta un borrado lógico del cliente de la vista operativa.
5. Pasado el tiempo legalmente requerido de conservación de datos contables (o a petición justificada), el sistema ejecuta el proceso de anonimización de datos personales en registros activos e históricos sin destruir las facturas ni la trazabilidad contable.


* **Flujos alternativos:**
* *A1: Cliente sin deudas:* Se cancelan las recurrencias en la fecha efectiva y el cliente pasa a estado "Inactivo por Baja".


* **Flujos de excepción:**
* *E1: Fallo en la comunicación por API externa:* Si la baja se solicita externamente pero el token es inválido, el sistema retorna una respuesta de error con los detalles autorizados sin modificar la BD.


* **Postcondiciones:** La suscripción futura queda cancelada; las deudas pasadas se mantienen en cobro y se anonimizan los datos personales cumpliendo con la política de conservación.

---

### CU-09: Fusión y Control de Clientes Duplicados

* **Identificador:** CU-09
* **Objetivo:** Detectar, revisar y consolidar fichas de clientes duplicadas en un procedimiento administrativo completamente auditado.
* **Actor o actores participantes:** Administrador, Superadministrador.
* **Precondiciones:** Existen dos o más fichas de clientes identificadas como potencialmente duplicadas por NIF, Razón Social o Email.
* **Flujo principal:**
1. El Administrador accede al módulo de "Detección de Duplicados".
2. El sistema muestra las coincidencias encontradas por identificador fiscal o contacto.
3. El Administrador revisa las fichas y selecciona la "Ficha Principal / Conservada" y la "Ficha Secundaria / A Consolidar".
4. El sistema muestra la vista previa de la consolidación (histórico de facturas, contactos, cobros y comentarios que se unificarán).
5. El Administrador confirma la fusión.
6. El sistema vincula el historial de la ficha secundaria a la principal y marca la secundaria como "Inactiva por Fusión", registrando quién ejecutó y autorizó el proceso.


* **Flujos alternativos:**
* *A1: Descarte de duplicado:* El Administrador marca los registros como "Duplicado Falso", evitando que el sistema vuelva a sugerir la fusión de ambas empresas.


* **Flujos de excepción:**
* *E1: Intentos de fusión automática por el sistema:* El sistema tiene estrictamente prohibido fusionar empresas automáticamente sin validación humana explícita.


* **Postcondiciones:** Las facturas y transacciones quedan consolidadas bajo un único identificador fiscal activo, conservando el rastro del cliente de origen.

---

### CU-10: Revisión Anual y Cambio de Rango de Facturación

* **Identificador:** CU-10
* **Objetivo:** Analizar los ingresos anuales del cliente, proponer un nuevo rango de tarifa y aplicar el cambio previa aprobación humana.
* **Actor o actores participantes:** Administrador, Superadministrador, Sistema.
* **Precondiciones:** Es principio de año o se dispone del dato de facturación anual verificada del cliente (vía entrada manual o fuente simulada).
* **Flujo principal:**
1. El sistema ejecuta a inicio de año la evaluación de rangos de facturación comparando la facturación del cliente con las tablas de tarifas vigentes.
2. El sistema genera una lista de "Clientes pendientes de cambio de rango" con el precio actual y la tarifa sugerida.
3. El Administrador revisa la ficha del cliente, introduce/verifica la facturación declarada y presiona "Proponer Cambio de Rango".
4. El sistema calcula el impacto económico e ingresa la propuesta en la Bandeja de Aprobaciones.
5. El Superadministrador aprueba el cambio de tarifa.
6. El sistema fija la nueva tarifa con su respectiva fecha de entrada en vigor.


* **Flujos alternativos:**
* *A1: Modificación directa por debajo del umbral de aprobación:* Si el cambio está dentro de los márgenes automáticos de configuración, el Administrador puede confirmar sin requerir nivel adicional.


* **Flujos de excepción:**
* *E1: Falta de datos de facturación anual:* El cliente queda marcado como "Revisión anual pendiente" y mantiene su tarifa anterior hasta que se introduzcan los datos requeridos.


* **Postcondiciones:** El nuevo rango aplica únicamente a las facturas futuras emitidas tras la fecha de efecto, manteniendo intacto el histórico de precios de facturas pasadas.

---

### CU-11: Solicitud y Aprobación de Excepciones Comerciales

* **Identificador:** CU-11
* **Objetivo:** Tramitar y validar condiciones especiales (descuentos fuera de norma, meses de carencia o precios pactados) garantizando la separación de funciones.
* **Actor o actores participantes:** Usuario Operativo (Solicitante), Administrador / Superadministrador (Aprobador).
* **Precondiciones:** La regla de negocio exige separación de funciones si el descuento o excepción excede un umbral económico determinado.
* **Flujo principal:**
1. El Usuario Operativo crea una solicitud de excepción comercial para un cliente, detallando el porcentaje de descuento o meses de carencia y justificando el motivo.
2. El sistema evalúa el umbral parametrizado en la base de datos.
3. Si la solicitud requiere aprobación, el sistema la envía a la "Bandeja de Aprobaciones Pendientes" y notifica al aprobador.
4. El Superadministrador/Administrador accede a la bandeja, examina la justificación y hace clic en "Aprobar" o "Rechazar" (incluyendo un comentario).
5. El sistema notifica la resolución al solicitante y actualiza las condiciones económicas del cliente si fue aprobada.


* **Flujos alternativos:**
* *A1: Autoconcesión rechazada:* Si un Administrador intenta aprobar una excepción que él mismo solicitó y la norma exige separación de funciones, el sistema impide la confirmación (A1.1).


* **Flujos de excepción:**
* *E1: Solicitud rechazada:* La propuesta queda en estado "Rechazada" con el motivo registrado; las facturas del cliente continúan según la tarifa estándar.


* **Postcondiciones:** Queda registrado el historial completo (quién solicitó, quién aprobó/rechazó, fechas, motivo, valor anterior y nuevo valor).

---

### CU-12: Generación Automática de Facturas Recurrentes

* **Identificador:** CU-12
* **Objetivo:** Generar de forma masiva y programada las facturas periódicas de los clientes activos, asignando series y numeración oficial sin duplicados.
* **Actor o actores participantes:** Sistema (Proceso Cron en segundo plano).
* **Precondiciones:** Es el día de facturación configurado (ej.: día 1 de mes), la suscripción está activa y los parámetros fiscales e impuestos están vigentes.
* **Flujo principal:**
1. El generador automático se activa en segundo plano en la fecha/hora programada.
2. El sistema identifica todas las suscripciones activas que deben facturarse.
3. Para cada cliente, el sistema calcula los conceptos, aplica los impuestos predefinidos, descuentos y la moneda correspondiente.
4. El sistema asigna la serie y el número correlativo único oficial a la factura únicamente en el instante de su emisión oficial.
5. El sistema genera la factura en estado "Emitida", crea el documento PDF oficial basado en la plantilla vigente y prepara el lote de notificaciones.
6. El sistema registra el resultado del proceso (clientes procesados, facturas creadas, fallos si los hubiera).


* **Flujos alternativos:**
* *A1: Recuperación tras caída de proceso:* Al reanudarse, el proceso mensual automático detecta qué facturas del ciclo no se generaron y las emite sin duplicar números ni facturas previas.


* **Flujos de excepción:**
* *E1: Error en datos fiscales de un cliente:* El sistema aísla el fallo para ese cliente en concreto, notifica la incidencia al responsable interno en el dashboard y continúa procesando el resto de los clientes del lote.


* **Postcondiciones:** Se emiten las facturas recurrentes con correlatividad garantizada en transacciones concurrentes; los estados pasan a "Emitida/Enviada".

---

### CU-13: Emisión de Facturas Puntuales y Presupuestos

* **Identificador:** CU-13
* **Objetivo:** Crear y emitir facturas no recurrentes o presupuestos previos para servicios adicionales o ventas directas.
* **Actor o actores participantes:** Administrador, Usuario Operativo.
* **Precondiciones:** El cliente debe estar registrado en el sistema.
* **Flujo principal:**
1. El usuario hace clic en "Nueva Factura Puntual" (o "Nuevo Presupuesto").
2. El usuario selecciona el cliente, moneda y líneas de detalle (concepto del catálogo o personalizado, cantidad, precio unitario, descuento aplicable e impuestos).
3. El sistema valida los importes, calcula subtotales y muestra el borrador en pantalla.
4. Si es una factura, el usuario presiona "Emitir Factura"; el sistema asigna serie/número correlativo oficial y genera el PDF.


* **Flujos alternativos:**
* *A1: Convertir Presupuesto Aceptado en Factura:* El usuario genera un presupuesto, lo envía al cliente y, tras su aceptación, presiona "Convertir en Factura", reutilizando automáticamente todas las líneas de detalle para emitir el documento.


* **Flujos de excepción:**
* *E1: Alerta de variación de precio inusual (Price Checker):* Si el precio ingresado fluctúa drásticamente de la media o del catálogo, el sistema muestra una advertencia de verificación antes de confirmar.


* **Postcondiciones:** La factura puntual o presupuesto queda registrada; el balance del cliente se actualiza con los nuevos importes emitidos.

---

### CU-14: Modificación y Rectificación de Facturas

* **Identificador:** CU-14
* **Objetivo:** Corregir errores en facturas emitidas mediante la creación de facturas rectificativas o modificaciones dentro del plazo legal trimestral.
* **Actor o actores participantes:** Administrador, Superadministrador.
* **Precondiciones:** La factura original existe y se encuentra en un estado que requiere rectificación o ajuste.
* **Flujo principal:**
1. El usuario localiza la factura original en el sistema.
2. Si el trimestre fiscal NO ha cerrado y la normativa lo admite, el usuario puede realizar ajustes autorizados registrando el motivo.
3. Si la factura está emitida y el trimestre ya fue entregado a Hacienda (o se requiere devolución/corrección formal), el usuario selecciona "Crear Factura Rectificativa".
4. El sistema vincula la rectificativa a la factura original, propone los datos a corregir, aplica la serie de rectificativas configurada por el Superadministrador y solicita la justificación.
5. Si el importe supera el umbral de aprobación, la rectificativa se envía a la bandeja de aprobaciones.
6. Tras aprobarse, el sistema emite el documento rectificativo sin reescribir ni borrar la factura original.


* **Flujos alternativos:**
* *A1: Corrección de ficha por dato erróneo:* Si la rectificación se debe a un dato de ficha de cliente mal introducido, se corrige la ficha con trazabilidad completa.


* **Flujos de excepción:**
* *E1: Intento de edición directa de factura bloqueada por cierre trimestral:* El sistema bloquea los campos y exige obligatoriamente el flujo de factura rectificativa.


* **Postcondiciones:** Se emite la factura rectificativa con su serie independiente, conservando la trazabilidad matemática e inmutabilidad de la factura original.

---

### CU-15: Consulta, Descarga y Exportación de Facturas y Documentos

* **Identificador:** CU-15
* **Objetivo:** Permitir a los usuarios autorizados visualizar, filtrar, previsualizar y descargar facturas en formato PDF oficial y listados en CSV.
* **Actor o actores participantes:** Usuario Operativo, Administrador, Superadministrador.
* **Precondiciones:** El usuario tiene permisos para consultar el cliente o factura correspondiente.
* **Flujo principal:**
1. El usuario accede al listado de facturas.
2. El usuario aplica filtros (por rango de fechas, cliente, estado, país, moneda, responsable, o búsqueda inteligente por texto).
3. El sistema muestra la lista paginada de facturas que cumplen con los criterios.
4. El usuario selecciona una factura y hace clic en "Descargar PDF" o "Ver Vista Previa".
5. El sistema genera o sirve el PDF garantizando que utilice la plantilla de diseño vigente en la fecha de emisión original de la factura.


* **Flujos alternativos:**
* *A1: Exportación masiva a CSV/Zip:* El usuario selecciona un conjunto de facturas y presiona "Exportar a CSV" o "Exportar Paquete Gestoría (ZIP)". El sistema procesa en segundo plano y notifica cuando el archivo está listo para descarga.
* *A2: Adjuntar/Consultar Documentos Anexos:* El usuario puede consultar o subir pedidos de cliente, presupuestos firmados o justificantes asociados a la factura, respetando las restricciones de formato e inoperatividad de archivos peligrosos.


* **Flujos de excepción:**
* *E1: Intento de acceso mediante URL adivinable sin permisos:* El sistema intercepta la petición, verifica los permisos del token y deniega el acceso con un error de autorización.


* **Postcondiciones:** Los documentos son entregados en el formato correcto sin alterar los datos del sistema ni exponer información no autorizada.

---

### CU-16: Integración y Conciliación de Pagos

* **Identificador:** CU-16
* **Objetivo:** Registrar, verificar y conciliar movimientos bancarios y pagos procedentes de pasarelas de pago (Stripe/Revolut) o transferencias manuales con las facturas emitidas.
* **Actor o actores participantes:** Administrador, Usuario Operativo, Sistema / Pasarela Externa (Stripe simulado).
* **Precondiciones:** Existen facturas emitidas en estado "Pendiente" y se reciben notificaciones o extractos de cobro.
* **Flujo principal:**
1. El sistema recibe una notificación automática de pago de la pasarela (webhook) o el usuario introduce un movimiento bancario manual.
2. El sistema busca la factura correspondiente mediante la referencia de pago o identificador de factura.
3. Si la referencia y el importe coinciden exactamente, el sistema marca la factura como "Pagada", actualiza el saldo del cliente y registra la fecha de cobro.
4. Si se recibe la confirmación vía Stripe, el sistema actualiza de inmediato el estado del cobro.


* **Flujos alternativos:**
* *A1: Conciliación Parcial / Discrepancia de Importe:* Si el movimiento bancario difiere del total de la factura (pago menor o mayor), el sistema asigna el estado "Parcialmente Conciliada" o alerta de sobrepago, generando una tarea en el Dashboard para conciliación manual.
* *A2: Intervención por referencia no clara:* El usuario busca la factura manualmente desde el módulo de conciliación y vincula el movimiento bancario tras verificar el origen del dinero.


* **Flujos de excepción:**
* *E1: Detección de dobles pagos:* El sistema detecta un segundo cobro recibido para una misma factura ya pagada, frena el procesamiento automático y levanta una alerta para revisión administrativa.


* **Postcondiciones:** La factura pasa a estado "Pagada" o "Parcialmente conciliada"; se actualiza el desglose del saldo del cliente (facturado, cobrado y pendiente) con trazabilidad auditada.

---

### CU-17: Tramitar Perdón de Deudas y Gestión de Impagos

* **Identificador:** CU-17
* **Objetivo:** Ejecutar la condonación total o parcial de una deuda vencida mediante un proceso administrativo estrictamente controlado con estado propio y justificación obligatoria.
* **Actor o actores participantes:** Superadministrador, Administrador.
* **Precondiciones:** La factura se encuentra en estado "Vencida" o "Impagada" y se han agotado o pausado los cauces habituales de recobro.
* **Flujo principal:**
1. El Administrador accede a la factura impagada y selecciona "Solicitar Condonación / Perdonar Deuda".
2. El usuario introduce obligatoriamente el motivo formal del perdón de la deuda y adjunta la documentación de soporte si existe.
3. La solicitud se envía a la Bandeja de Aprobaciones del Superadministrador.
4. El Superadministrador evalúa el impacto financiero y autoriza la condonación.
5. El sistema cambia el estado de la factura a "Condonada / Deuda Perdonada", detiene los recordatorios automáticos de impago e incrementa la trazabilidad contable.


* **Flujos alternativos:**
* *A1: Condonación parcial:* Se perdona una parte de la deuda ajustando la suma pendiente y manteniendo el resto bajo gestión de cobro.


* **Flujos de excepción:**
* *E1: Solicitud sin justificación:* El sistema impide el envío de la solicitud si el campo de motivo se encuentra vacío.


* **Postcondiciones:** Se cancela el saldo pendiente de la factura sin borrar su registro histórico y se audita formalmente la acción.

---

### CU-18: Gestión de Incidencias de Clientes y Facturas

* **Identificador:** CU-18
* **Objetivo:** Registrar, hacer seguimiento y resolver incidencias operativas o disputas asociadas a clientes y facturas específicas.
* **Actor o actores participantes:** Usuario Operativo, Administrador, Sistema.
* **Precondiciones:** El cliente o la factura relevante existen en la plataforma.
* **Flujo principal:**
1. El usuario pulsa "Abrir Incidencia" desde la ficha del cliente o desde una factura en concreto.
2. El usuario indica la prioridad (Baja, Media, Alta, Crítica), el responsable asignado, la descripción del problema y adjunta archivos opcionales.
3. El usuario decide si la incidencia debe "Pausar notificaciones automáticas de cobro" para evitar molestias al cliente durante la disputa.
4. Los usuarios participantes agregan comentarios internos con posibilidad de mencionar a otros compañeros (`@usuario`), lo que genera una notificación interna al usuario mencionado.
5. Una vez resuelto el problema, el responsable ingresa la resolución obligatoria y cambia el estado a "Cerrada".
6. Si las notificaciones estaban pausadas, se reanudan o ajustan según la resolución.


* **Flujos alternativos:**
* *A1: Comentarios externos:* Un usuario redacta explícitamente una nota derivada de la incidencia para ser enviada al cliente en una comunicación formal.


* **Flujos de excepción:**
* *E1: Cierre de incidencia sin texto de resolución:* El sistema bloquea el cambio de estado hasta que se especifique cómo se resolvió el caso.


* **Postcondiciones:** Queda registrada la resolución de la incidencia en la ficha del cliente y en el log de auditoría.

---

### CU-19: Gestión de Plantillas de Notificación

* **Identificador:** CU-19
* **Objetivo:** Personalizar y mantener las plantillas de correo y mensajes de WhatsApp simulados que utiliza el sistema para notificaciones automáticas.
* **Actor o actores participantes:** Administrador, Superadministrador.
* **Precondiciones:** El usuario cuenta con los permisos de configuración corporativa.
* **Flujo principal:**
1. El Administrador accede al menú "Configuración -> Plantillas de Notificación".
2. Selecciona la plantilla a editar (Factura Emitida, Recordatorio Previo, Impago, Disputa Resuelta, etc.) y el idioma (Español / Inglés).
3. El Administrador edita el cuerpo del mensaje insertando variables dinámicas (`{empresa}`, `{numero_factura}`, `{importe}`, `{vencimiento}`, `{responsable}`, `{enlace_soporte}`).
4. El sistema ejecuta una verificación automática de variables para confirmar que el sintaxis es correcto y no faltan parámetros requeridos.
5. El Administrador utiliza la opción "Enviar prueba" a su correo para validar la previsualización.
6. El Administrador guarda los cambios; el sistema genera una nueva versión en el historial de plantillas.


* **Flujos alternativos:**
* *A1: Restaurar versión anterior:* El Administrador selecciona una versión antigua de la plantilla y la reestablece como activa.


* **Flujos de excepción:**
* *E1: Variable dinámica inválida o mal escrita:* El sistema detecta la discrepancia, marca la sintaxis errónea en rojo e impide guardar hasta corregirla.


* **Postcondiciones:** Los futuros envíos automatizados utilizarán la versión actualizada de la plantilla.

---

### CU-20: Envío de Recordatorios de Pago e Impago

* **Identificador:** CU-20
* **Objetivo:** Ejecutar de forma automatizada las notificaciones de vencimiento previo y recordatorios periódicos de impago a través de Email y WhatsApp simulado.
* **Actor o actores participantes:** Sistema (Cron / Scheduler), Cliente (Receptor).
* **Precondiciones:** Existen facturas en estado "Emitida" próximas a vencer o en estado "Vencida", con las reglas de cadencia parametrizadas.
* **Flujo principal:**
1. El tarea programada revisa diariamente las facturas vigentes.
2. Para facturas a vencer: En los días configurados previo al vencimiento (ej.: días 3 y 5 de plazo), el sistema evalúa si cae en día hábil (según calendario laboral); si cae en festivo/fin de semana, programa el envío para el siguiente día hábil.
3. Para facturas impagadas: A partir del día 5 posterior al vencimiento, el sistema dispara avisos cada 2-3 días.
4. El sistema selecciona la plantilla e idioma del cliente, sustituye las variables dinámicas y envía la notificación por los canales autorizados (Email / WhatsApp simulado).
5. El sistema registra el intento de entrega y su estado (Enviado, Entregado, Rebote).


* **Flujos alternativos:**
* *A1: Rebote de correo (Fallo de entrega):* Si la entrega por email rebota, el sistema notifica inmediatamente al responsable interno y reintenta el envío hacia el contacto de facturación secundario del cliente.
* *A2: Notificaciones pausadas por incidencia/superposición:* Si el cliente tiene una incidencia activa con pausa de avisos o vencimientos superpuestos específicos, el sistema salta el envío.


* **Flujos de excepción:**
* *E1: Error general en el proveedor de comunicaciones:* El proceso aísla las fallas, las almacena en la cola de errores de integración e intenta el reenvío en el siguiente ciclo sin congelar el sistema.


* **Postcondiciones:** Se envían y registran las notificaciones de cobranza manteniendo el historial de entrega en la ficha de la factura.

---

### CU-21: Visualización de Dashboards y Indicadores

* **Identificador:** CU-21
* **Objetivo:** Presentar paneles de información operativa y financiera consolidada con filtros avanzados y accesos directos a la gestión diaria.
* **Actor o actores participantes:** Superadministrador, Administrador, Usuario Operativo.
* **Precondiciones:** El usuario está autenticado y posee asignaciones/permisos para consultar métricas.
* **Flujo principal:**
1. El usuario accede al Dashboard principal de la aplicación.
2. El sistema identifica el rol y asignaciones del usuario para estructurar los datos:
* **Dashboard de Trabajo Diario:** Presenta facturas que vencen pronto, facturas vencidas, cobros por conciliar, problemas de entrega/rebotes, aprobaciones pendientes, revisiones anuales e incidencias del usuario.
* **Dashboard Financiero (Admin/Superadmin):** Presenta facturación (semana, mes, trimestre, año), totales cobrados, pendientes y vencidos, tiempo medio de cobro, previsiones de ingresos recurrentes (MRR), evolución y clientes con mayor deuda.


3. El sistema aplica las conversiones de moneda extranjera a la moneda base de informe según los tipos de cambio registrados sin alterar los datos legales.
4. El usuario hace clic en cualquier indicador o cifra para navegar directamente al listado filtrado detallado.


* **Flujos alternativos:**
* *A1: Configurar Objetivos:* El Administrador/Superadministrador establece un objetivo o presupuesto mensual/anual de facturación dentro del Dashboard. El sistema almacena el progreso sin sobreescribir históricos de ejercicios anteriores.


* **Flujos de excepción:**
* *E1: Intento de visualización de Dashboard financiero global por un Usuario Operativo:* El sistema oculta las métricas financieras globales y muestra únicamente el panel operativo del catálogo de clientes que el usuario tiene asignados.


* **Postcondiciones:** El usuario obtiene una radiografía precisa en tiempo real de la situación comercial y operativa de la empresa.

---

### CU-22: Generación de Informes y Paquetes para Gestoría

* **Identificador:** CU-22
* **Objetivo:** Elaborar informes financieros, analíticas de cobros/impagos e integrar paquetes exportables para la gestoría contable.
* **Actor o actores participantes:** Administrador, Superadministrador, Sistema.
* **Precondiciones:** Existen registros de operaciones y facturación en el período objeto de consulta.
* **Flujo principal:**
1. El Administrador accede a la sección "Informes y Gestoría".
2. Selecciona el tipo de informe (Analítica de pagos/impagos a 1 semana, 1 mes, 1 año; Informe de prorrateos; Ventas por país/sector; Paquete Gestoría Trimestral).
3. Establece los filtros requeridos (Periodo, Cliente, Responsable interno, País, Sector, Estado de factura, Tipo recurrente o puntual, Moneda).
4. El usuario hace clic en "Generar Informe".
5. Si el informe requiere procesamiento pesado, el sistema ejecuta la tarea en segundo plano e informa con una notificación cuando el reporte está listo.
6. El usuario descarga el informe formateado en CSV legible o en un archivo ZIP con todos los documentos comprimidos.


* **Flujos alternativos:**
* *A1: Envío programado de informes:* El Administrador configura el envío automático semanal o mensual de informes a usuarios autorizados mediante enlace temporal autenticado.


* **Flujos de excepción:**
* *E1: Intento de exportación en periodo no cerrado sin permisos:* Si se solicita un reporte de gestoría de un periodo abierto, el sistema incluye una marca de agua explícita ("Borrador / Periodo No Cerrado").


* **Postcondiciones:** Se descarga el informe o paquete contable solicitados y se registra la acción en la trazabilidad de descargas de datos.

---

### CU-23: Configuración Parámetros Globales del Sistema

* **Identificador:** CU-23
* **Objetivo:** Gestionar las variables de negocio, impuestos, divisas, umbrales de aprobación y datos fiscales corporativos almacenados en base de datos.
* **Actor o actores participantes:** Superadministrador.
* **Precondiciones:** Autenticación con el rol de Superadministrador.
* **Flujo principal:**
1. El Superadministrador accede a "Configuración del Sistema".
2. Modifica parámetros globales como:
* Datos fiscales de Turbineh y logotipo oficial.
* Plazo de pago por defecto y reglas de vencimiento.
* Monedas soportadas y registro de tipos de cambio.
* Catálogo de impuestos aplicables (IVA, retenciones, impuestos extranjeros).
* Umbrales económicos para aprobación de descuentos y excepciones.
* Series de facturación independientes (Ordinarias, Rectificativas, etc.).
* Dominio de correo permitido e integración con WhatsApp/Pasarelas.
* Calendario laboral y días festivos.


3. El Superadministrador ejecuta una simulación previa si el cambio afecta a automatizaciones globales.
4. El Superadministrador guarda los parámetros.
5. El sistema valida los datos e incrementa el historial de cambios de configuración.


* **Flujos alternativos:**
* *A1: Gestión de Categorías / Centros de Coste:* El Superadministrador crea, renombra o desactiva categorías de proyectos o centros de coste sin romper el historial de las facturas previamente vinculadas.


* **Flujos de excepción:**
* *E1: Modificación de configuración con efecto retroactivo:* Las nuevas reglas aplican únicamente a las operaciones creadas con posterioridad a la modificación, garantizando la inmutabilidad retroactiva.


* **Postcondiciones:** Las variables globales quedan actualizadas en la base de datos sin necesidad de recompilar o modificar el código fuente de la aplicación.

---

### CU-24: Consulta de Registros de Auditoría (Audit Log)

* **Identificador:** CU-24
* **Objetivo:** Inspeccionar el historial inmutable de acciones, cambios de configuración, emisiones de facturas y eventos del sistema.
* **Actor o actores participantes:** Superadministrador.
* **Precondiciones:** El rol del usuario debe ser Superadministrador.
* **Flujo principal:**
1. El Superadministrador entra al módulo de "Auditoría de Sistema".
2. El sistema presenta el buscador y filtros de eventos auditados (por Usuario, Cliente, Tipo de Acción: Alta, Cambio fiscal, Emisión, Descuento, Condonación, Reasignación, Conciliación, Rango de Fechas).
3. El Superadministrador realiza la búsqueda.
4. El sistema presenta los registros indicando: Fecha/Hora exacta, Usuario responsable, Acción realizada, Valor Anterior y Valor Nuevo.
5. El Superadministrador puede exportar los resultados para revisiones de cumplimiento o seguridad.


* **Flujos alternativos:**
* *A1: Localización de datos personales (Derecho de acceso):* El Superadministrador atiende una petición interna para localizar todos los datos personales asociados a un contacto en el registro del sistema.


* **Flujos de excepción:**
* *E1: Intento de edición o borrado de registros de auditoría:* El sistema carece de interfaces o métodos para alterar o eliminar el historial pasado, incluso para el Superadministrador.


* **Postcondiciones:** Se consulta el registro de auditoría de forma de solo lectura, manteniendo la integridad inmutable de la información.

---

### CU-25: Integración API para Consultas y Operaciones Externas

* **Identificador:** CU-25
* **Objetivo:** Proporcionar una interfaz API segura para que sistemas autorizados (como BrainOS) consulten el estado comercial, facturas pendientes y soliciten operaciones controladas.
* **Actor o actores participantes:** Sistema Externo (BrainOS / API Client), Sistema FACTURBINEH.
* **Precondiciones:** El sistema externo se autentica mediante una API Key / Token con permisos específicos y control de límites de tasa (rate limiting).
* **Flujo principal:**
1. El sistema externo envía una petición HTTPS cifrada al endpoint de la API de FACTURBINEH (ej.: consultar facturas pendientes de un cliente mediante su ID).
2. El sistema valida las credenciales de la API y los permisos del token.
3. El sistema procesa la solicitud y recupera la información solicitada sin exponer la interfaz de usuario.
4. El sistema devuelve la respuesta estructurada en formato JSON con los datos pertinentes (ej.: lista de facturas pendientes, importe total adeudado, estado de recurrencia).
5. El sistema registra la llamada a la API en el log de auditoría técnica.


* **Flujos alternativos:**
* *A1: Petición de Baja Comercial externa via API:* El sistema externo solicita tramitar la baja de una suscripción. FACTURBINEH evalúa las deudas pendientes, ejecuta el flujo definido en el CU-08 y retorna el resultado detallado de la operación.


* **Flujos de excepción:**
* *E1: Autenticación de API fallida:* El sistema retorna un código HTTP 401/403 con un mensaje estructurado de error e inhabilita las peticiones repetidas que excedan la tasa límite.
* *E2: Intento de ejecutar acciones no autorizadas:* Si el contrato de API no otorga permisos de escritura para un parámetro, el sistema rechaza la solicitud.


* **Postcondiciones:** La operación externa se ejecuta o consulta de forma aislada, segura y totalmente rastreable.

---

### CU-26: Cierre de Periodos Fiscales y Copias de Seguridad

* **Identificador:** CU-26
* **Objetivo:** Bloquear las modificaciones sobre periodos contables cerrados y gestionar la ejecución y prueba de restauración de copias de seguridad.
* **Actor o actores participantes:** Superadministrador, Administrador, Sistema.
* **Precondiciones:** El periodo a cerrar ha finalizado y las facturas/conciliaciones del mismo están completas.
* **Flujo principal:**
1. El Administrador accede al menú "Cierre de Periodos".
2. Selecciona el mes o trimestre a cerrar y presiona "Marcar Periodo como Cerrado".
3. El sistema valida que no existan facturas en estado "Borrador" o "Pendiente de aprobación" en dicho rango.
4. El sistema bloquea el periodo: no se permite la emisión directa, edición ni borrado de facturas con fecha dentro del rango cerrado.
5. De forma paralela, el sistema automático ejecuta backups periódicos de la base de datos y documentos PDF adjuntos según la política definida.
6. El Superadministrador puede acceder a la "Pantalla de Salud del Sistema" para verificar la última ejecución del backup, el estado de las colas de correo y lanzar pruebas de restauración simuladas.


* **Flujos alternativos:**
* *A1: Descarga de paquete de respaldo:* El Superadministrador descarga una copia cifrada del registro de movimientos y accesos para almacenamiento fuera del sitio.


* **Flujos de excepción:**
* *E1: Intento de agregar o modificar facturas en periodo cerrado:* El sistema rechaza la transacción e indica que el periodo fiscal se encuentra bloqueado por la administración.


* **Postcondiciones:** El periodo fiscal queda formalmente inmutable y se verifica la integridad de las copias de seguridad del sistema.