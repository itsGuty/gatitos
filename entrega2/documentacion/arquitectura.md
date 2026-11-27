# Arquitectura y componentes — FACTURBINEH

## Vista representada

Arquitectura lógica en cuatro capas (Presentación, Seguridad y Control de Acceso, Negocio e Integración, y Persistencia) organizada en paquetes funcionales e interfaces desacopladas (`IAuth`, `IClientes`, `IFacturación`, `IGobierno`, `INotifier`, `IPayment`, `IAudit`). Representa la descomposición estructural de responsabilidades y dependencias entre componentes de software del sistema FACTURBINEH, **no** su topología física de despliegue ni infraestructura de red.

## Responsabilidades

| Componente | Responsabilidad | Relaciones principales |
| :---- | :---- | :---- |
| **Panel Web Responsivo (SPA)** | Interfaz de usuario para la operación del sistema, captura de datos y visualización de estados. | Se comunica con `IAuth` para autenticación y envía peticiones API con token JWT a `RBACGuard`. |
| **AuthController** (`IAuth`) | Gestión de los flujos de inicio y cierre de sesión, validación de credenciales y bloqueo por intentos fallidos. | Delega la creación y validación de sesiones en `SessionManager`. |
| **SessionManager** | Administración del ciclo de vida de las sesiones activas y estados de bloqueo de usuarios. | Persiste datos de sesión en `PostgreSQL` y es consultado por `AuthController` y `RBACGuard`. |
| **RBACGuard** | Control de acceso basado en roles (*Superadmin*, *Admin*, *Operativo*) sobre los endpoints de negocio. | Valida sesión y rol con `SessionManager`; enruta peticiones autorizadas hacia `IClientes`, `IFacturación`, `IGobierno`, `Gestor de Incidencias` y `Fachada Exportación Gestoría`. |
| **GlobalExceptionFilter (buscar en diagrama)** | Interceptación centralizada de excepciones no controladas, generación de identificadores de error y formateo de respuestas seguras. | Captura excepciones de `Servicio de Facturación y Presupuestos` (y capa de negocio) y registra trazas en `TechnicalLogger`. |
| **Gestor de Clientes, Asignaciones y Tarifas** (`IClientes`) | Administración del maestro de clientes, asignación de servicios, tarifas personalizadas y datos de contacto/canales. | Notifica impactos contables a `Servicio de Facturación y Presupuestos`, provee contactos a `Gestor de Notificaciones`, registra eventos en `IAudit` y persiste en `PostgreSQL`. |
| **Servicio de Facturación y Presupuestos** (`IFacturación`) | Núcleo de lógica financiera: emisión de presupuestos y facturas, cálculo de importes, aplicación de excepciones y conciliación de cobros. | Solicita PDFs a `Generador PDF / ZIP`, envíos a `Gestor de Notificaciones`, gestiona cobros vía `IPayment`, recibe eventos de `Motor Cron Recurrente` y `Webhooks`, registra auditoría en `IAudit` y persiste en `PostgreSQL`. |
| **Motor Cron Recurrente** | Ejecución programada y automatizada de ciclos de facturación periódica. | Dispara los procesos de generación periódica en `Servicio de Facturación y Presupuestos`. |
| **Gestor de Incidencias** | Registro, seguimiento y resolución de incidencias reportadas sobre facturas o servicios. | Consulta datos en `Gestor de Clientes`, vincula la factura afectada en `Servicio de Facturación y Presupuestos`, pausa recordatorios en `Gestor de Notificaciones`, audita en `IAudit` y persiste en `PostgreSQL`. |
| **Config. Global y Aprobaciones** (`IGobierno`) | Administración de entidades emisoras, periodos contables, reglas de excepción y flujos de aprobación administrativa. | Provee parámetros y excepciones a `Servicio de Facturación y Presupuestos`, registra eventos en `IAudit` y persiste en `PostgreSQL`. |
| **Generador PDF / ZIP** | Renderizado de documentos oficiales (facturas, presupuestos) en formato PDF y empaquetado masivo en archivos ZIP. | Recibe solicitudes de `Servicio de Facturación y Presupuestos` y `Fachada Exportación Gestoría`; almacena los binarios en `Almacenamiento Privado`. |
| **Fachada Exportación Gestoría** | Orquestación de paquetes documentales y contables destinados a exportación externa para gestoría. | Solicita el empaquetado masivo a `Generador PDF / ZIP`. |
| **Gestor de Notificaciones** | Selección de plantillas, programación de recordatorios de cobro y orquestación de envíos multicanal. | Obtiene contactos de `Gestor de Clientes`, recibe pausas de `Gestor de Incidencias`, despacha mensajes mediante `INotifier` y persiste estados en `PostgreSQL`. |
| **Adaptador Comunicaciones** (`INotifier`) | Implementación técnica de entrega de mensajes hacia proveedores externos de Email y WhatsApp. | Implementa la interfaz `INotifier` invocada por `Gestor de Notificaciones`. |
| **Webhooks (`/webhooks/*`)** | Recepción y validación criptográfica (HMAC) de notificaciones asíncronas de pasarelas y bancos. | Concilia movimientos de pago en `Servicio de Facturación y Presupuestos` y registra eventos de pago en `IAudit`. |
| **Adaptador Pagos y Banca** (`IPayment`) | Integración con pasarelas de pago y banca externa (Stripe, Revolut, transferencias) para cobros y consultas. | Implementa la interfaz `IPayment` consumida por `Servicio de Facturación y Presupuestos`. |
| **AuditService** (`IAudit`) | Registro inmutable de trazabilidad funcional (altas, modificaciones, emisiones, aprobaciones, pagos e incidencias). | Recibe eventos de los componentes de negocio y webhooks; persiste el log de auditoría en `PostgreSQL`. |
| **TechnicalLogger** | Registro técnico de errores de ejecución, *stack traces* e identificadores de diagnóstico. | Recibe errores desde `GlobalExceptionFilter` y escribe en `Ficheros Log Técnico`. |
| **PostgreSQL** | Persistencia relacional transaccional para entidades de negocio, usuarios, sesiones, notificaciones y auditoría funcional. | Almacén central consumido por `SessionManager`, `GestorCLI`, `SrvFact`, `GestorInc`, `ConfigApr`, `GestorNotif` y `AuditSvc`. |
| **Almacenamiento Privado** | Repositorio de archivos binarios generados (PDFs individuales y lotes ZIP) con acceso restringido. | Escrito y gestionado por `Generador PDF / ZIP`. |
| **Ficheros Log Técnico** | Almacenamiento en disco/sistema de archivos para trazas técnicas y depuración de fallos. | Escrito por `TechnicalLogger`. |

&nbsp;

&nbsp;

## Justificación y alternativas

* **Separación estricta en capas y uso de interfaces (`IPayment`, `INotifier`, `IAudit`, `IFacturación`, etc.):** Aísla el núcleo de negocio frente a cambios en proveedores externos (pasarelas de pago o canales de mensajería) y frente a la interfaz SPA. *Alternativa descartada:* acoplar directamente los SDKs de Stripe/WhatsApp en `SrvFact`, lo que dificultaría añadir nuevas entidades bancarias o realizar pruebas unitarias aisladas.  
* **Control de acceso centralizado (`RBACGuard` \+ `SessionManager`):** Garantiza que toda petición API pase por validación de sesión y permisos de rol antes de alcanzar los servicios de dominio. *Alternativa descartada:* validar permisos dentro de cada servicio de negocio, lo que duplicaría lógica de seguridad y aumentaría el riesgo de brechas.  
* **Separación de persistencia (Relacional vs. Binarios vs. Logs técnicos):** `PostgreSQL` asegura integridad ACID para datos contables y auditoría; el `Almacenamiento Privado` evita sobrecargar la base de datos con archivos PDF/ZIP pesados; y `Ficheros Log Técnico` mantiene las trazas de depuración disponibles incluso ante caídas de la base de datos.  
* **Segregación de auditoría funcional (`AuditService`) y trazas técnicas (`TechnicalLogger`):** Permite cumplir con requisitos de trazabilidad contable y fiscal en base de datos sin mezclar ruido técnico ni *stack traces*.  
* **Integración asíncrona mediante Webhooks con verificación HMAC:** Asegura la conciliación fiable de pagos externos frente a fallos de red en el cliente web, mitigando riesgos de suplantación de peticiones.

&nbsp;

## Trazabilidad

* `CU-01` (Autenticación y gestión de sesión) se realiza mediante la cooperación de `Panel Web Responsivo (SPA)`, `AuthController` (`IAuth`), `SessionManager` y `PostgreSQL`.  
* `CU-02` (Control de acceso y autorización de peticiones) se realiza mediante la cooperación de `Panel Web Responsivo (SPA)`, `RBACGuard`, `SessionManager` y `PostgreSQL`.  
* `CU-03` (Gestión de clientes, asignaciones y tarifas) se realiza mediante la cooperación de `RBACGuard`, `Gestor de Clientes, Asignaciones y Tarifas` (`IClientes`), `Servicio de Facturación y Presupuestos`, `AuditService` (`IAudit`) y `PostgreSQL`.  
* `CU-04` (Emisión manual de presupuestos y facturas) se realiza mediante la cooperación de `RBACGuard`, `Servicio de Facturación y Presupuestos` (`IFacturación`), `Gestor de Clientes, Asignaciones y Tarifas`, `Config. Global y Aprobaciones`, `Generador PDF / ZIP`, `Almacenamiento Privado`, `Gestor de Notificaciones`, `Adaptador Comunicaciones` (`INotifier`), `AuditService` (`IAudit`) y `PostgreSQL`.  
* `CU-05` (Facturación periódica automatizada) se realiza mediante la cooperación de `Motor Cron Recurrente`, `Servicio de Facturación y Presupuestos`, `Config. Global y Aprobaciones`, `Generador PDF / ZIP`, `Almacenamiento Privado`, `Gestor de Notificaciones`, `Adaptador Comunicaciones` (`INotifier`), `AuditService` (`IAudit`) y `PostgreSQL`.  
* `CU-06` (Gestión y consulta de cobros) se realiza mediante la cooperación de `Servicio de Facturación y Presupuestos`, `Adaptador Pagos y Banca` (`IPayment`) y `PostgreSQL`.  
* `CU-07` (Conciliación automática de pagos vía Webhook) se realiza mediante la cooperación de `Webhooks (/webhooks/*)`, `Servicio de Facturación y Presupuestos`, `AuditService` (`IAudit`) y `PostgreSQL`.  
* `CU-08` (Gestión de incidencias y pausa de recordatorios de cobro) se realiza mediante la cooperación de `RBACGuard`, `Gestor de Incidencias`, `Gestor de Clientes, Asignaciones y Tarifas`, `Servicio de Facturación y Presupuestos`, `Gestor de Notificaciones`, `AuditService` (`IAudit`) y `PostgreSQL`.  
* `CU-09` (Configuración global, periodos y aprobaciones) se realiza mediante la cooperación de `RBACGuard`, `Config. Global y Aprobaciones` (`IGobierno`), `Servicio de Facturación y Presupuestos`, `AuditService` (`IAudit`) y `PostgreSQL`.  
* `CU-10` (Exportación masiva de documentación para gestoría) se realiza mediante la cooperación de `RBACGuard`, `Fachada Exportación Gestoría`, `Generador PDF / ZIP` y `Almacenamiento Privado`.  
* `CU-11` (Registro centralizado de excepciones y trazas técnicas) se realiza mediante la cooperación de `Servicio de Facturación y Presupuestos`, `GlobalExceptionFilter`, `TechnicalLogger` y `Ficheros Log Técnico`.

&nbsp;