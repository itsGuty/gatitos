# Requisitos 

## Requisitos de negocio 
El cliente necesita un sistema que reúna toda la información de facturación de la compañía y convertirla en respuestas, avisos e informes. Turbineh necesita un control claro y sencillo de pagos y facturas de clientes aumentando así la velocidad del control de facturación.


## Requisitos de usuario 
- UR1. El usuario debe poder hacer login usando su cuenta interna de la empresa (email y contraseña)
- UR2. El usuario lector debe poder gestionar las facturas de los clientes que le asoció el usuario admin.
- UR3. El usuario admin debe poder asignar clientes a los lectores.
- UR3. El usuario admin debe poder cambiar los parámetros de configuración del sistema.
- UR4. Para el usuario deben estar automatizados todos los procesos que no requieran intervención humana: generación de facturas, envío y notificaciones.

## Requisitos del sistema 

### Requisitos Funcionales 

- FR1. Generar facturas (formato dado)
- FR2. Facturas simples / recurrentes
- FR3. Control facturas existentes
- FR4. Página de registro a la plataforma (log in)
- FR5. Analíticas de pagos / impagos (1 semana, 1 mes y 1 año)
- FR6. Notificaciones de impago al cliente por WhatsApp/ email (a partir del día 5 cada 2-3 días) (antes de ser impago notif dia 3 y 5 de pago en plazo)
- FR7. Control de roles (admin y lector). Admin: acceso a leer y editar todo. Da acceso a lector.
- FR8. Conexión a pasarela de pago para verificar / modificar pagos pendientes.
- FR9. Para el administrador la posibilidad de tener una copia de seguridad del registro de movimientos y (los accesos cedidos a los lectores?)
- FR10. Evaluación anual para posible cambio de plan según ingresos anuales.
- FR11. Usuarios tienen que poder leer y descargar facturas (lectores)
- FR12. Factura el día que contratan BrainOS de lo equivalente a lo que queda de mes, y luego el día 1 una factura completa normal.
- FR13. Factura modificable por trimestre, si ya ha pasado el trimestre se bloquea, debido a que ya ha sido entregado a hacienda.
- FR14. Cliente se puede dar de baja, si le falta un pago envía un aviso, se mantendrán los datos del cliente  el máximo tiempo posible legalmente, se realizará un borrado lógico.(No sé si esto hay que añadirlo) (al servicio de BrainOS)



### Requisitos No-Funcionales 

- NFR1. Facturar en menos de un minuto. Minimizar clicks. Interfaz intuitiva.
- NFR2. Protección de datos fiscales.
- NFR3. Que funcione desde el móvil, aplicación web responsiva (multidispositivo).
- NFR4. Las facturas han de ser estándar y válidas.
- NFR5. Completamente independiente de BrainOS (no es para clientes).
- NFR6. Login solo Usuario (Email de Turbineh) y contraseña (para saber si soy lector o admin), previamente registrado por la empresa
- NFR7. Interfaz intuitiva. Intentar minimizar el número de clicks.
- NFR8. Aplicación web responsiva.
