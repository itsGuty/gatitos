@startuml

title SD-00: Validación de Sesión JWT y Autorización RBAC

autonumber

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

database ": PostgreSQL" as DB

&nbsp;

FE \-\> RBAC : peticionAPI(jwt: String, rolRequerido: Rol, payload: Object)

activate RBAC

&nbsp;

RBAC \-\> SessionMgr : validarSesionYRol(jwt: String, rolRequerido: Rol)

activate SessionMgr

SessionMgr \-\> DB : verificarTokenYPermisos(jwt: String)

activate DB

DB \--\> SessionMgr : sesion: SesionActivaDTO

deactivate DB

&nbsp;

alt Token válido y rol autorizado (Superadmin / Admin / Operativo)

&nbsp;&nbsp;&nbsp;&nbsp;SessionMgr \--\> RBAC : autorizado: Boolean \= true

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SessionMgr

&nbsp;&nbsp;&nbsp;&nbsp;note right of RBAC : Continúa la ejecución del Caso de Uso

&nbsp;&nbsp;&nbsp;&nbsp;deactivate RBAC

else Token inválido, expirado o rol insuficiente

&nbsp;&nbsp;&nbsp;&nbsp;SessionMgr \--\> RBAC : errorAcceso: ForbiddenException

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SessionMgr

&nbsp;&nbsp;&nbsp;&nbsp;RBAC \--\> FE : 403 Forbidden (mensaje: String)

&nbsp;&nbsp;&nbsp;&nbsp;deactivate RBAC

end

@enduml

&nbsp;

@startuml

title CU-01: Iniciar Sesión en la Plataforma

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Operativo" as U

participant ": PanelWeb" as FE

participant ": AuthController" as AuthCtrl

participant ": SessionManager" as SessionMgr

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

U \-\> FE : iniciarSesion(email: String, password: String)

activate FE

FE \-\> AuthCtrl : login(email: String, password: String)

activate AuthCtrl

&nbsp;

&nbsp;

AuthCtrl \-\> SessionMgr : validarCredencialesYBloqueo(email: String, password: String)

activate SessionMgr

SessionMgr \-\> DB : obtenerUsuarioPorEmail(email: String)

activate DB

DB \--\> SessionMgr : datosUsuario: UsuarioRecord

deactivate DB

&nbsp;

&nbsp;

alt Credenciales válidas y cuenta no bloqueada

&nbsp;&nbsp;&nbsp;&nbsp;create participant ": Usuario" as Usr

&nbsp;&nbsp;&nbsp;&nbsp;SessionMgr \--\>\> Usr : Usuario(datosUsuario)

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SessionMgr \-\> SessionMgr : generarTokenJWT(Usr)

&nbsp;&nbsp;&nbsp;&nbsp;SessionMgr \-\> DB : registrarSesionActiva(Usr.id, tokenJWT: String)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SessionMgr : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SessionMgr \--\> AuthCtrl : sesion: SesionDTO

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SessionMgr

&nbsp;&nbsp;&nbsp;&nbsp;AuthCtrl \--\> FE : 200 OK (sesion: SesionDTO)

&nbsp;&nbsp;&nbsp;&nbsp;deactivate AuthCtrl

&nbsp;&nbsp;&nbsp;&nbsp;FE \-\> FE : almacenarTokenJWT(sesion.token)

&nbsp;&nbsp;&nbsp;&nbsp;FE \--\> U : mostrarDashboard()

&nbsp;&nbsp;&nbsp;&nbsp;deactivate FE

else Credenciales incorrectas o cuenta bloqueada

&nbsp;&nbsp;&nbsp;&nbsp;SessionMgr \-\> DB : actualizarIntentosYBloqueo(email: String)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SessionMgr : bloqueado: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SessionMgr \--\> AuthCtrl : errorAutenticacion: AuthException

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SessionMgr

&nbsp;&nbsp;&nbsp;&nbsp;AuthCtrl \--\> FE : 401 Unauthorized (mensaje: String)

&nbsp;&nbsp;&nbsp;&nbsp;deactivate AuthCtrl

&nbsp;&nbsp;&nbsp;&nbsp;FE \--\> U : mostrarError("Credenciales inválidas o cuenta bloqueada")

&nbsp;&nbsp;&nbsp;&nbsp;deactivate FE

end

@enduml

&nbsp;

@startuml

title CU-02: Recuperar y Restablecer Contraseña

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Operativo" as U

participant ": PanelWeb" as FE

participant ": AuthController" as AuthCtrl

participant ": SessionManager" as SessionMgr

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

\== Fase 1: Solicitud de Token de Recuperación \==

U \-\> FE : solicitarRecuperacion(email: String)

activate FE

FE \-\> AuthCtrl : iniciarRecuperacion(email: String)

activate AuthCtrl

AuthCtrl \-\> SessionMgr : generarTokenRecuperacion(email: String)

activate SessionMgr

SessionMgr \-\> DB : verificarUsuarioActivo(email: String)

activate DB

DB \--\> SessionMgr : usuarioId: UUID

deactivate DB

SessionMgr \-\> DB : guardarTokenTemporal(usuarioId: UUID, expiracion: DateTime)

activate DB

DB \--\> SessionMgr : tokenRecuperacion: String

deactivate DB

SessionMgr \--\> AuthCtrl : resultado: Boolean

deactivate SessionMgr

AuthCtrl \--\> FE : 200 OK (confirmacion: String)

deactivate AuthCtrl

FE \--\> U : mostrarConfirmacion("Solicitud procesada")

deactivate FE

&nbsp;

&nbsp;

\== Fase 2: Restablecimiento de Credenciales \==

U \-\> FE : introducirNuevaPassword(token: String, nuevaPassword: String)

activate FE

FE \-\> AuthCtrl : restablecerCredenciales(token: String, nuevaPassword: String)

activate AuthCtrl

AuthCtrl \-\> SessionMgr : validarYActualizarPassword(token: String, nuevaPassword: String)

activate SessionMgr

SessionMgr \-\> DB : validarTokenTemporal(token: String)

activate DB

DB \--\> SessionMgr : estadoToken: Boolean

deactivate DB

&nbsp;

&nbsp;

alt Token válido y vigente

&nbsp;&nbsp;&nbsp;&nbsp;SessionMgr \-\> DB : actualizarHashYDesbloquear(token: String, hash: String)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SessionMgr : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;SessionMgr \--\> AuthCtrl : actualizado: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SessionMgr

&nbsp;&nbsp;&nbsp;&nbsp;AuthCtrl \--\> FE : 200 OK

&nbsp;&nbsp;&nbsp;&nbsp;deactivate AuthCtrl

&nbsp;&nbsp;&nbsp;&nbsp;FE \--\> U : redirigirALogin()

&nbsp;&nbsp;&nbsp;&nbsp;deactivate FE

else Token inválido o expirado

&nbsp;&nbsp;&nbsp;&nbsp;SessionMgr \--\> AuthCtrl : errorToken: AuthException

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SessionMgr

&nbsp;&nbsp;&nbsp;&nbsp;AuthCtrl \--\> FE : 400 Bad Request

&nbsp;&nbsp;&nbsp;&nbsp;deactivate AuthCtrl

&nbsp;&nbsp;&nbsp;&nbsp;FE \--\> U : mostrarError("Token inválido o caducado")

&nbsp;&nbsp;&nbsp;&nbsp;deactivate FE

end

@enduml

&nbsp;

@startuml

title CU-03: Gestión de Usuarios y Roles

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Admin" as Admin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": ConfigApr" as ConfigApr

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Admin \-\> FE : guardarUsuarioYRol(jwt: String, datosUsuario: UsuarioDTO)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> ConfigApr : gestionarUsuarioYRol(datosUsuario: UsuarioDTO)

activate ConfigApr

&nbsp;

&nbsp;

create participant ": Usuario" as Usr

ConfigApr \--\>\> Usr : Usuario(datosUsuario)

&nbsp;

&nbsp;

ConfigApr \-\> DB : insertarOActualizarUsuario(Usr)

activate DB

DB \--\> ConfigApr : usuarioGuardado: Usuario

deactivate DB

&nbsp;

&nbsp;

ConfigApr \-\>\> AuditSvc : registrarEventoConfig(evento: "GESTION\_USUARIO\_ROL", usuarioGuardado.id)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

ConfigApr \--\> RBAC : resultado: UsuarioDTO

deactivate ConfigApr

RBAC \--\> FE : 200 OK (resultado: UsuarioDTO)

deactivate RBAC

FE \--\> Admin : mostrarConfirmacionUsuario()

deactivate FE

@enduml

&nbsp;

@startuml

title CU-04: Asignación y Delegación Temporal de Clientes

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Admin" as Admin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": GestorCLI" as GestorCLI

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Admin \-\> FE : asignarCliente(jwt: String, clienteId: UUID, operativoId: UUID, fechaFin: Date)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> GestorCLI : asignarCliente(clienteId: UUID, operativoId: UUID, fechaFin: Date)

activate GestorCLI

GestorCLI \-\> DB : actualizarAsignacionYDelegacion(clienteId: UUID, operativoId: UUID, fechaFin: Date)

activate DB

DB \--\> GestorCLI : ok: Boolean

deactivate DB

&nbsp;

&nbsp;

GestorCLI \-\>\> AuditSvc : registrarEventoCliente(evento: "ASIGNACION\_CLIENTE", clienteId: UUID)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

GestorCLI \--\> RBAC : asignacionConfirmada: Boolean

deactivate GestorCLI

RBAC \--\> FE : 200 OK

deactivate RBAC

FE \--\> Admin : actualizarCarteraClientes()

deactivate FE

@enduml

&nbsp;

@startuml

title CU-05: Alta de Nuevo Cliente y Registro Comercial

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Operativo" as Op

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": GestorCLI" as GestorCLI

participant ": SrvFact" as SrvFact

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Op \-\> FE : registrarCliente(jwt: String, datosCliente: ClienteDTO)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Operativo")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> GestorCLI : crearCliente(datosCliente: ClienteDTO)

activate GestorCLI

GestorCLI \-\> DB : validarDuplicidadNIF(datosCliente.nif)

activate DB

DB \--\> GestorCLI : disponible: Boolean

deactivate DB

&nbsp;

&nbsp;

alt Datos válidos y NIF disponible

&nbsp;&nbsp;&nbsp;&nbsp;create participant ": Cliente" as Cli

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \--\>\> Cli : Cliente(datosCliente)

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \-\> DB : insertarClienteTarifaYAsignacion(Cli)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> GestorCLI : clienteCreado: Cliente

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \-\> SrvFact : notificarImpactoContable(clienteCreado.id, datosCliente.tarifa)

&nbsp;&nbsp;&nbsp;&nbsp;activate SrvFact

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> DB : inicializarCuentaContableCliente(clienteCreado.id)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SrvFact : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \--\> GestorCLI : impactoRegistrado: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SrvFact

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \-\>\> AuditSvc : registrarEventoCliente(evento: "ALTA\_CLIENTE", clienteCreado.id)

&nbsp;&nbsp;&nbsp;&nbsp;activate AuditSvc

&nbsp;&nbsp;&nbsp;&nbsp;AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> AuditSvc : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;deactivate AuditSvc

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \--\> RBAC : clienteCreado: Cliente

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GestorCLI

&nbsp;&nbsp;&nbsp;&nbsp;RBAC \--\> FE : 201 Created (clienteCreado: Cliente)

&nbsp;&nbsp;&nbsp;&nbsp;deactivate RBAC

&nbsp;&nbsp;&nbsp;&nbsp;FE \--\> Op : mostrarFichaCliente(clienteCreado)

&nbsp;&nbsp;&nbsp;&nbsp;deactivate FE

else NIF duplicado o datos inválidos

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \--\> RBAC : errorValidacion: ValidationException

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GestorCLI

&nbsp;&nbsp;&nbsp;&nbsp;RBAC \--\> FE : 400 Bad Request

&nbsp;&nbsp;&nbsp;&nbsp;deactivate RBAC

&nbsp;&nbsp;&nbsp;&nbsp;FE \--\> Op : mostrarErroresFormulario()

&nbsp;&nbsp;&nbsp;&nbsp;deactivate FE

end

@enduml

&nbsp;

@startuml

title CU-06: Gestión de Contactos e Información Fiscal

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Operativo" as Op

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": GestorCLI" as GestorCLI

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Op \-\> FE : actualizarDatosFiscalesYContactos(jwt: String, clienteId: UUID, datos: ContactoFiscalDTO)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Operativo")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> GestorCLI : actualizarContactoYFiscalidad(clienteId: UUID, datos: ContactoFiscalDTO)

activate GestorCLI

GestorCLI \-\> DB : comprobarConflictosEmailYNIF(clienteId: UUID, datos: ContactoFiscalDTO)

activate DB

DB \--\> GestorCLI : sinConflictos: Boolean

deactivate DB

&nbsp;

&nbsp;

alt Datos válidos y sin conflictos

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \-\> DB : actualizarClienteYContactos(clienteId: UUID, datos: ContactoFiscalDTO)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> GestorCLI : actualizado: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \-\>\> AuditSvc : registrarEventoCliente(evento: "ACTUALIZACION\_FISCAL\_CONTACTO", clienteId: UUID)

&nbsp;&nbsp;&nbsp;&nbsp;activate AuditSvc

&nbsp;&nbsp;&nbsp;&nbsp;AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> AuditSvc : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;deactivate AuditSvc

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \--\> RBAC : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GestorCLI

&nbsp;&nbsp;&nbsp;&nbsp;RBAC \--\> FE : 200 OK

&nbsp;&nbsp;&nbsp;&nbsp;deactivate RBAC

&nbsp;&nbsp;&nbsp;&nbsp;FE \--\> Op : mostrarNotificacionExito()

&nbsp;&nbsp;&nbsp;&nbsp;deactivate FE

else Conflicto de email o NIF

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \--\> RBAC : errorConflicto: ConflictException

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GestorCLI

&nbsp;&nbsp;&nbsp;&nbsp;RBAC \--\> FE : 409 Conflict

&nbsp;&nbsp;&nbsp;&nbsp;deactivate RBAC

&nbsp;&nbsp;&nbsp;&nbsp;FE \--\> Op : mostrarErrorConflicto()

&nbsp;&nbsp;&nbsp;&nbsp;deactivate FE

end

@enduml

&nbsp;

@startuml

title CU-07: Pausa Temporal y Reactivación de Suscripción

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Admin" as Admin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": GestorCLI" as GestorCLI

participant ": SrvFact" as SrvFact

participant ": PayAdp" as PayAdp

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Admin \-\> FE : cambiarEstadoSuscripcion(jwt: String, clienteId: UUID, nuevoEstado: EstadoSuscripcion)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> GestorCLI : cambiarEstadoSuscripcion(clienteId: UUID, nuevoEstado: EstadoSuscripcion)

activate GestorCLI

GestorCLI \-\> DB : actualizarEstadoCliente(clienteId: UUID, nuevoEstado: EstadoSuscripcion)

activate DB

DB \--\> GestorCLI : suscripcionExternaId: String

deactivate DB

&nbsp;

&nbsp;

GestorCLI \-\> SrvFact : aplicarImpactoContableSuscripcion(clienteId: UUID, nuevoEstado: EstadoSuscripcion)

activate SrvFact

SrvFact \-\> PayAdp : gestionarSuscripcionExterna(suscripcionExternaId: String, nuevoEstado: EstadoSuscripcion)

activate PayAdp

PayAdp \--\> SrvFact : estadoPasarela: String

deactivate PayAdp

SrvFact \--\> GestorCLI : sincronizado: Boolean

deactivate SrvFact

&nbsp;

&nbsp;

GestorCLI \-\>\> AuditSvc : registrarEventoCliente(evento: "CAMBIO\_ESTADO\_SUSCRIPCION", clienteId: UUID)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

GestorCLI \--\> RBAC : estadoActualizado: EstadoSuscripcion

deactivate GestorCLI

RBAC \--\> FE : 200 OK (estadoActualizado: EstadoSuscripcion)

deactivate RBAC

FE \--\> Admin : actualizarEstadoEnUI()

deactivate FE

@enduml

&nbsp;

@startuml

title CU-08: Tramitar Baja Comercial y Anonimización

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Admin" as Admin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": GestorCLI" as GestorCLI

participant ": SrvFact" as SrvFact

participant ": PayAdp" as PayAdp

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Admin \-\> FE : confirmarBajaDefinitiva(jwt: String, clienteId: UUID)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> GestorCLI : tramitarBajaYAnonimizacion(clienteId: UUID)

activate GestorCLI

GestorCLI \-\> SrvFact : verificarDeudasPendientes(clienteId: UUID)

activate SrvFact

SrvFact \-\> DB : consultarFacturasImpagadas(clienteId: UUID)

activate DB

DB \--\> SrvFact : totalDeuda: Decimal

deactivate DB

SrvFact \--\> GestorCLI : tieneDeudas: Boolean

deactivate SrvFact

&nbsp;

&nbsp;

alt Existen deudas pendientes

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \--\> RBAC : errorDeudaPendiente: BusinessException

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GestorCLI

&nbsp;&nbsp;&nbsp;&nbsp;RBAC \--\> FE : 422 Unprocessable Entity

&nbsp;&nbsp;&nbsp;&nbsp;deactivate RBAC

&nbsp;&nbsp;&nbsp;&nbsp;FE \--\> Admin : mostrarError("No se puede tramitar baja con deudas pendientes")

&nbsp;&nbsp;&nbsp;&nbsp;deactivate FE

else Sin deudas pendientes

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \-\> SrvFact : cancelarSuscripcionPorBaja(clienteId: UUID)

&nbsp;&nbsp;&nbsp;&nbsp;activate SrvFact

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> PayAdp : cancelarSuscripcion(clienteId: UUID)

&nbsp;&nbsp;&nbsp;&nbsp;activate PayAdp

&nbsp;&nbsp;&nbsp;&nbsp;PayAdp \--\> SrvFact : cancelada: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate PayAdp

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \--\> GestorCLI : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SrvFact

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \-\> DB : anonimizarPIIYBajaLogica(clienteId: UUID)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> GestorCLI : bajaCompletada: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \-\>\> AuditSvc : registrarEventoCliente(evento: "BAJA\_Y\_ANONIMIZACION\_GDPR", clienteId: UUID)

&nbsp;&nbsp;&nbsp;&nbsp;activate AuditSvc

&nbsp;&nbsp;&nbsp;&nbsp;AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> AuditSvc : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;deactivate AuditSvc

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \--\> RBAC : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GestorCLI

&nbsp;&nbsp;&nbsp;&nbsp;RBAC \--\> FE : 200 OK

&nbsp;&nbsp;&nbsp;&nbsp;deactivate RBAC

&nbsp;&nbsp;&nbsp;&nbsp;FE \--\> Admin : redirigirAListadoClientes()

&nbsp;&nbsp;&nbsp;&nbsp;deactivate FE

end

@enduml

&nbsp;

@startuml

title CU-09: Fusión y Control de Clientes Duplicados

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Admin" as Admin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": GestorCLI" as GestorCLI

participant ": SrvFact" as SrvFact

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Admin \-\> FE : fusionarClientes(jwt: String, origenId: UUID, destinoId: UUID)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> GestorCLI : fusionarClientes(origenId: UUID, destinoId: UUID)

activate GestorCLI

GestorCLI \-\> DB : reasignarContactosYMarcarOrigenFusionado(origenId: UUID, destinoId: UUID)

activate DB

DB \--\> GestorCLI : ok: Boolean

deactivate DB

&nbsp;

&nbsp;

GestorCLI \-\> SrvFact : reasignarFacturasPorFusion(origenId: UUID, destinoId: UUID)

activate SrvFact

SrvFact \-\> DB : transferirFacturasYPresupuestos(origenId: UUID, destinoId: UUID)

activate DB

DB \--\> SrvFact : ok: Boolean

deactivate DB

SrvFact \--\> GestorCLI : impactoContableAplicado: Boolean

deactivate SrvFact

&nbsp;

&nbsp;

GestorCLI \-\>\> AuditSvc : registrarEventoCliente(evento: "FUSION\_CLIENTES", origenId: UUID, destinoId: UUID)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

GestorCLI \--\> RBAC : fusionCompletada: Boolean

deactivate GestorCLI

RBAC \--\> FE : 200 OK

deactivate RBAC

FE \--\> Admin : recargarFichaClienteDestino(destinoId)

deactivate FE

@enduml

&nbsp;

@startuml

title CU-10: Revisión Anual y Cambio de Rango de Facturación

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

participant ": CronJob" as Cron

participant ": SrvFact" as SrvFact

participant ": GestorNotif" as GestorNotif

participant ": GestorCLI" as GestorCLI

participant ": NotifAdp" as NotifAdp

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Cron \-\> SrvFact : ejecutarRevisionAnualTarifas(anio: Integer)

activate SrvFact

SrvFact \-\> DB : obtenerClientesActivosYFacturacionAnual(anio: Integer)

activate DB

DB \--\> SrvFact : candidatosCambio: List\<CambioTramoDTO\>

deactivate DB

&nbsp;

&nbsp;

loop Por cada cliente con cambio de tramo

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> DB : actualizarTramoCuotaCliente(candidato.clienteId, candidato.nuevoTramo)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SrvFact : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> GestorNotif : notificarCambioCuotaAnual(candidato.clienteId, candidato.nuevoTramo)

&nbsp;&nbsp;&nbsp;&nbsp;activate GestorNotif

&nbsp;&nbsp;&nbsp;&nbsp;GestorNotif \-\> GestorCLI : obtenerContactoYCanales(candidato.clienteId)

&nbsp;&nbsp;&nbsp;&nbsp;activate GestorCLI

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \-\> DB : consultarContactosPreferidos(candidato.clienteId)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> GestorCLI : contacto: ContactoDTO

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \--\> GestorNotif : contacto: ContactoDTO

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GestorCLI

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;GestorNotif \-\>\> NotifAdp : despacharMensaje(contacto: ContactoDTO, plantilla: "CAMBIO\_TRAMO")

&nbsp;&nbsp;&nbsp;&nbsp;GestorNotif \-\> DB : registrarEnvioNotificacion(candidato.clienteId, "CAMBIO\_TRAMO")

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> GestorNotif : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;GestorNotif \--\> SrvFact : notificado: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GestorNotif

end

&nbsp;

&nbsp;

SrvFact \-\>\> AuditSvc : registrarEventoFactura(evento: "REVISION\_ANUAL\_TARIFAS", anio: Integer)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

SrvFact \--\> Cron : procesoCompletado: Boolean

deactivate SrvFact

@enduml

&nbsp;

@startuml

title CU-11: Solicitud y Aprobación de Excepciones Comerciales

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Operativo" as Op

actor ": Admin" as Admin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": ConfigApr" as ConfigApr

participant ": SrvFact" as SrvFact

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

\== Fase 1: Solicitud de Excepción \==

Op \-\> FE : solicitarExcepcion(jwt: String, clienteId: UUID, datosExcepcion: ExcepcionDTO)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Operativo")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> ConfigApr : crearSolicitudExcepcion(clienteId: UUID, datosExcepcion: ExcepcionDTO)

activate ConfigApr

ConfigApr \-\> DB : insertarExcepcionPendiente(datosExcepcion: ExcepcionDTO)

activate DB

DB \--\> ConfigApr : excepcionId: UUID

deactivate DB

&nbsp;

&nbsp;

ConfigApr \-\>\> AuditSvc : registrarEventoSolicitud(evento: "SOLICITUD\_EXCEPCION", excepcionId: UUID)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

ConfigApr \--\> RBAC : excepcionId: UUID

deactivate ConfigApr

RBAC \--\> FE : 201 Created (excepcionId: UUID)

deactivate RBAC

FE \--\> Op : mostrarConfirmacionEnvio()

deactivate FE

&nbsp;

&nbsp;

\== Fase 2: Resolución de Excepción \==

Admin \-\> FE : resolverExcepcion(jwt: String, excepcionId: UUID, aprobada: Boolean)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> ConfigApr : resolverExcepcion(excepcionId: UUID, aprobada: Boolean)

activate ConfigApr

ConfigApr \-\> DB : actualizarEstadoExcepcion(excepcionId: UUID, aprobada: Boolean)

activate DB

DB \--\> ConfigApr : excepcion: ExcepcionDTO

deactivate DB

&nbsp;

&nbsp;

alt Solicitud Aprobada

&nbsp;&nbsp;&nbsp;&nbsp;ConfigApr \-\> SrvFact : aplicarExcepcionFacturacion(excepcion: ExcepcionDTO)

&nbsp;&nbsp;&nbsp;&nbsp;activate SrvFact

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> DB : guardarReglaExcepcionCliente(excepcion: ExcepcionDTO)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SrvFact : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \--\> ConfigApr : aplicada: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SrvFact

end

&nbsp;

&nbsp;

ConfigApr \-\>\> AuditSvc : registrarEventoSolicitud(evento: "RESOLUCION\_EXCEPCION", excepcionId: UUID)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

ConfigApr \--\> RBAC : resolucionConfirmada: Boolean

deactivate ConfigApr

RBAC \--\> FE : 200 OK

deactivate RBAC

FE \--\> Admin : mostrarConfirmacionResolucion()

deactivate FE

@enduml

&nbsp;

@startuml

title CU-12: Generación Automática de Facturas Recurrentes

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

participant ": CronJob" as Cron

participant ": SrvFact" as SrvFact

participant ": GenDocs" as GenDocs

participant ": PayAdp" as PayAdp

participant ": GestorNotif" as GestorNotif

participant ": GestorCLI" as GestorCLI

participant ": NotifAdp" as NotifAdp

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

database ": Storage" as Storage

&nbsp;

&nbsp;

Cron \-\> SrvFact : ejecutarCicloRecurrente(fecha: Date)

activate SrvFact

SrvFact \-\> DB : obtenerClientesFacturables(fecha: Date)

activate DB

DB \--\> SrvFact : clientes: List\<ClienteFacturableDTO\>

deactivate DB

&nbsp;

&nbsp;

loop Por cada cliente facturable

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> DB : generarYGuardarFactura(cliente: ClienteFacturableDTO)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SrvFact : facturaGuardada: Factura

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> GenDocs : emitirDocumentoPDF(facturaGuardada)

&nbsp;&nbsp;&nbsp;&nbsp;activate GenDocs

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \-\> Storage : guardarArchivoPDF(facturaGuardada.id, contenido: Byte\[\])

&nbsp;&nbsp;&nbsp;&nbsp;activate Storage

&nbsp;&nbsp;&nbsp;&nbsp;Storage \--\> GenDocs : rutaPDF: String

&nbsp;&nbsp;&nbsp;&nbsp;deactivate Storage

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \--\> SrvFact : rutaPDF: String

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GenDocs

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> DB : asociarRutaPDF(facturaGuardada.id, rutaPDF: String)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SrvFact : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;alt Cobro automático habilitado

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> PayAdp : gestionarCobro(facturaGuardada)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;activate PayAdp

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PayAdp \--\> SrvFact : resultadoCobro: EstadoPago

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;deactivate PayAdp

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> DB : actualizarEstadoFactura(facturaGuardada.id, resultadoCobro: EstadoPago)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SrvFact : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;end

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> GestorNotif : solicitarEnvioFactura(facturaGuardada.id, cliente.id, rutaPDF: String)

&nbsp;&nbsp;&nbsp;&nbsp;activate GestorNotif

&nbsp;&nbsp;&nbsp;&nbsp;GestorNotif \-\> GestorCLI : obtenerContactoYCanales(cliente.id)

&nbsp;&nbsp;&nbsp;&nbsp;activate GestorCLI

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \-\> DB : consultarContactosPreferidos(cliente.id)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> GestorCLI : contacto: ContactoDTO

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \--\> GestorNotif : contacto: ContactoDTO

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GestorCLI

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;GestorNotif \-\>\> NotifAdp : despacharMensaje(contacto: ContactoDTO, rutaPDF: String)

&nbsp;&nbsp;&nbsp;&nbsp;GestorNotif \-\> DB : registrarEnvio(facturaGuardada.id, contacto: ContactoDTO)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> GestorNotif : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;GestorNotif \--\> SrvFact : envioEncolado: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GestorNotif

end

&nbsp;

&nbsp;

SrvFact \-\>\> AuditSvc : registrarEventoFactura(evento: "LOTE\_FACTURACION\_RECURRENTE", fecha: Date)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

SrvFact \--\> Cron : loteFinalizado: Boolean

deactivate SrvFact

@enduml

&nbsp;

@startuml

title CU-13: Emisión de Facturas Puntuales y Presupuestos

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Operativo" as Op

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": SrvFact" as SrvFact

participant ": GenDocs" as GenDocs

participant ": AuditService" as AuditSvc

participant ": GlobalExceptionFilter" as ExFilter

participant ": TechnicalLogger" as TechLog

database ": PostgreSQL" as DB

database ": Storage" as Storage

database ": LogFiles" as LogFiles

&nbsp;

&nbsp;

Op \-\> FE : emitirDocumento(jwt: String, datosDoc: DocumentoDTO)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Operativo")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> SrvFact : crearYEmitirDocumento(datosDoc: DocumentoDTO)

activate SrvFact

&nbsp;

&nbsp;

alt Datos fiscales y líneas válidos

&nbsp;&nbsp;&nbsp;&nbsp;create participant ": Documento" as Doc

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \--\>\> Doc : Documento(datosDoc)

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> DB : insertarDocumentoYLineas(Doc)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SrvFact : documento: Documento

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> GenDocs : emitirDocumento(documento: Documento)

&nbsp;&nbsp;&nbsp;&nbsp;activate GenDocs

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \-\> Storage : guardarPDF(documento.id, binario: Byte\[\])

&nbsp;&nbsp;&nbsp;&nbsp;activate Storage

&nbsp;&nbsp;&nbsp;&nbsp;Storage \--\> GenDocs : rutaPDF: String

&nbsp;&nbsp;&nbsp;&nbsp;deactivate Storage

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \--\> SrvFact : rutaPDF: String

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GenDocs

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> DB : actualizarRutaPDF(documento.id, rutaPDF: String)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SrvFact : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\>\> AuditSvc : registrarEventoFactura(evento: "EMISION\_DOCUMENTO", documento.id)

&nbsp;&nbsp;&nbsp;&nbsp;activate AuditSvc

&nbsp;&nbsp;&nbsp;&nbsp;AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> AuditSvc : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;deactivate AuditSvc

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \--\> RBAC : documentoEmitido: DocumentoDTO

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SrvFact

&nbsp;&nbsp;&nbsp;&nbsp;RBAC \--\> FE : 201 Created (documentoEmitido: DocumentoDTO)

&nbsp;&nbsp;&nbsp;&nbsp;deactivate RBAC

&nbsp;&nbsp;&nbsp;&nbsp;FE \--\> Op : mostrarPrevisualizacionPDF(documentoEmitido.rutaPDF)

&nbsp;&nbsp;&nbsp;&nbsp;deactivate FE

else Excepción técnica o de dominio

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \--\> ExFilter : lanzarExcepcion(excepcion: Exception)

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SrvFact

&nbsp;&nbsp;&nbsp;&nbsp;activate ExFilter

&nbsp;&nbsp;&nbsp;&nbsp;ExFilter \-\> TechLog : registrarTrazaError(stackTrace: String, errorId: UUID)

&nbsp;&nbsp;&nbsp;&nbsp;activate TechLog

&nbsp;&nbsp;&nbsp;&nbsp;TechLog \-\> LogFiles : escribirTraza(lineaLog: String)

&nbsp;&nbsp;&nbsp;&nbsp;activate LogFiles

&nbsp;&nbsp;&nbsp;&nbsp;LogFiles \--\> TechLog : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate LogFiles

&nbsp;&nbsp;&nbsp;&nbsp;TechLog \--\> ExFilter : logged: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate TechLog

&nbsp;&nbsp;&nbsp;&nbsp;ExFilter \--\> RBAC : errorControlado(errorId: UUID)

&nbsp;&nbsp;&nbsp;&nbsp;deactivate ExFilter

&nbsp;&nbsp;&nbsp;&nbsp;RBAC \--\> FE : 400 / 500 Error (errorId: UUID)

&nbsp;&nbsp;&nbsp;&nbsp;deactivate RBAC

&nbsp;&nbsp;&nbsp;&nbsp;FE \--\> Op : mostrarMensajeError(errorId)

&nbsp;&nbsp;&nbsp;&nbsp;deactivate FE

end

@enduml

&nbsp;

@startuml

title CU-14: Modificación y Rectificación de Facturas

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Admin" as Admin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": SrvFact" as SrvFact

participant ": GenDocs" as GenDocs

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

database ": Storage" as Storage

&nbsp;

&nbsp;

Admin \-\> FE : modificarORectificarFactura(jwt: String, facturaId: UUID, cambios: FacturaDTO)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> SrvFact : procesarAlteracionFactura(facturaId: UUID, cambios: FacturaDTO)

activate SrvFact

SrvFact \-\> DB : consultarFactura(facturaId: UUID)

activate DB

DB \--\> SrvFact : facturaActual: Factura

deactivate DB

&nbsp;

&nbsp;

alt Factura en estado "Borrador"

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> DB : actualizarBorrador(facturaId: UUID, cambios: FacturaDTO)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SrvFact : facturaActualizada: Factura

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> GenDocs : emitirDocumento(facturaActualizada: Factura)

&nbsp;&nbsp;&nbsp;&nbsp;activate GenDocs

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \-\> Storage : guardarPDF(facturaActualizada.id, binario: Byte\[\])

&nbsp;&nbsp;&nbsp;&nbsp;activate Storage

&nbsp;&nbsp;&nbsp;&nbsp;Storage \--\> GenDocs : rutaPDF: String

&nbsp;&nbsp;&nbsp;&nbsp;deactivate Storage

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \--\> SrvFact : rutaPDF: String

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GenDocs

else Factura "Emitida" o "Pagada" (Inmutable)

&nbsp;&nbsp;&nbsp;&nbsp;create participant ": FacturaRectificativa" as Rect

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \--\>\> Rect : FacturaRectificativa(facturaActual, cambios)

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> DB : guardarFacturaRectificativa(Rect)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SrvFact : rectificativa: FacturaRectificativa

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> GenDocs : emitirDocumento(rectificativa)

&nbsp;&nbsp;&nbsp;&nbsp;activate GenDocs

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \-\> Storage : guardarPDF(rectificativa.id, binario: Byte\[\])

&nbsp;&nbsp;&nbsp;&nbsp;activate Storage

&nbsp;&nbsp;&nbsp;&nbsp;Storage \--\> GenDocs : rutaPDF: String

&nbsp;&nbsp;&nbsp;&nbsp;deactivate Storage

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \--\> SrvFact : rutaPDF: String

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GenDocs

end

&nbsp;

&nbsp;

SrvFact \-\>\> AuditSvc : registrarEventoFactura(evento: "MODIFICACION\_O\_RECTIFICACION", facturaId: UUID)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

SrvFact \--\> RBAC : resultado: FacturaDTO

deactivate SrvFact

RBAC \--\> FE : 200 OK (resultado: FacturaDTO)

deactivate RBAC

FE \--\> Admin : mostrarConfirmacionFactura()

deactivate FE

@enduml

&nbsp;

@startuml

title CU-15: Consulta, Descarga y Exportación de Facturas

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Operativo" as Op

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": SrvFact" as SrvFact

participant ": GenDocs" as GenDocs

database ": PostgreSQL" as DB

database ": Storage" as Storage

&nbsp;

&nbsp;

\== Consulta Filtrada \==

Op \-\> FE : filtrarFacturas(jwt: String, filtros: FiltroFacturaDTO)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Operativo")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> SrvFact : consultarFacturas(filtros: FiltroFacturaDTO)

activate SrvFact

SrvFact \-\> DB : ejecutarConsultaPaginada(filtros: FiltroFacturaDTO)

activate DB

DB \--\> SrvFact : listado: Page\<FacturaDTO\>

deactivate DB

SrvFact \--\> RBAC : listado: Page\<FacturaDTO\>

deactivate SrvFact

RBAC \--\> FE : 200 OK (listado: Page\<FacturaDTO\>)

deactivate RBAC

FE \--\> Op : mostrarTablaFacturas(listado)

deactivate FE

&nbsp;

&nbsp;

\== Descarga PDF o Exportación \==

Op \-\> FE : descargarOExportar(jwt: String, formato: FormatoExportacion, filtros: FiltroFacturaDTO)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Operativo")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> SrvFact : exportarDocumentos(formato: FormatoExportacion, filtros: FiltroFacturaDTO)

activate SrvFact

&nbsp;

&nbsp;

alt Formato \== PDF individual

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> GenDocs : recuperarDocumentoPDF(facturaId: UUID)

&nbsp;&nbsp;&nbsp;&nbsp;activate GenDocs

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \-\> Storage : leerArchivoPDF(facturaId: UUID)

&nbsp;&nbsp;&nbsp;&nbsp;activate Storage

&nbsp;&nbsp;&nbsp;&nbsp;Storage \--\> GenDocs : archivoPDF: FileStream

&nbsp;&nbsp;&nbsp;&nbsp;deactivate Storage

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \--\> SrvFact : archivoPDF: FileStream

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GenDocs

else Formato \== Excel / CSV

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> DB : obtenerDatasetFacturas(filtros: FiltroFacturaDTO)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SrvFact : dataset: List\<FacturaDTO\>

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> GenDocs : generarExportacionTabular(dataset: List\<FacturaDTO\>, formato: FormatoExportacion)

&nbsp;&nbsp;&nbsp;&nbsp;activate GenDocs

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \-\> Storage : guardarTemporal(archivo: FileStream)

&nbsp;&nbsp;&nbsp;&nbsp;activate Storage

&nbsp;&nbsp;&nbsp;&nbsp;Storage \--\> GenDocs : archivoExportado: FileStream

&nbsp;&nbsp;&nbsp;&nbsp;deactivate Storage

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \--\> SrvFact : archivoExportado: FileStream

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GenDocs

end

&nbsp;

&nbsp;

SrvFact \--\> RBAC : stream: FileStream

deactivate SrvFact

RBAC \--\> FE : 200 OK (stream: FileStream)

deactivate RBAC

FE \--\> Op : iniciarDescargaNavegador()

deactivate FE

@enduml

&nbsp;

@startuml

title CU-16: Integración y Conciliación de Pagos

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Admin" as Admin

participant ": PanelWeb" as FE

participant ": WebhookCtrl" as WebhookCtrl

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": SrvFact" as SrvFact

participant ": PayAdp" as PayAdp

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

\== Flujo 1: Conciliación Automática vía Webhook (/webhooks/\*) \==

PayAdp \-\> WebhookCtrl : recibirWebhookPago(payload: Json, firmaHMAC: String)

activate WebhookCtrl

WebhookCtrl \-\> WebhookCtrl : validarFirmaHMAC(payload: Json, firmaHMAC: String)

WebhookCtrl \-\> SrvFact : conciliarMovimientoPago(movimiento: MovimientoPagoDTO)

activate SrvFact

SrvFact \-\> DB : buscarYMarcarFacturaPagada(movimiento: MovimientoPagoDTO)

activate DB

DB \--\> SrvFact : facturaConciliada: Factura

deactivate DB

SrvFact \--\> WebhookCtrl : conciliado: Boolean

deactivate SrvFact

&nbsp;

&nbsp;

WebhookCtrl \-\>\> AuditSvc : registrarEventoPago(evento: "PAGO\_CONCILIADO\_WEBHOOK", movimiento.id)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

WebhookCtrl \--\> PayAdp : 200 OK

deactivate WebhookCtrl

&nbsp;

&nbsp;

\== Flujo 2: Conciliación Manual (Transferencia / Banca) \==

Admin \-\> FE : registrarPagoManual(jwt: String, facturaId: UUID, datosPago: PagoManualDTO)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> SrvFact : conciliarPagoManual(facturaId: UUID, datosPago: PagoManualDTO)

activate SrvFact

SrvFact \-\> PayAdp : consultarCobroBancario(datosPago.referencia)

activate PayAdp

PayAdp \--\> SrvFact : verificado: Boolean

deactivate PayAdp

&nbsp;

&nbsp;

SrvFact \-\> DB : registrarPagoYActualizarFactura(facturaId: UUID, datosPago: PagoManualDTO)

activate DB

DB \--\> SrvFact : ok: Boolean

deactivate DB

&nbsp;

&nbsp;

SrvFact \-\>\> AuditSvc : registrarEventoFactura(evento: "PAGO\_MANUAL\_CONCILIADO", facturaId: UUID)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

SrvFact \--\> RBAC : pagoRegistrado: Boolean

deactivate SrvFact

RBAC \--\> FE : 200 OK

deactivate RBAC

FE \--\> Admin : actualizarListadoFacturas()

deactivate FE

@enduml

&nbsp;

@startuml

title CU-17: Tramitar Perdón de Deudas y Gestión de Impagos

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Admin" as Admin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": SrvFact" as SrvFact

participant ": GenDocs" as GenDocs

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

database ": Storage" as Storage

&nbsp;

&nbsp;

Admin \-\> FE : perdonarDeuda(jwt: String, facturaId: UUID, motivo: String, generarAbono: Boolean)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> SrvFact : tramitarPerdonDeuda(facturaId: UUID, motivo: String, generarAbono: Boolean)

activate SrvFact

SrvFact \-\> DB : marcarFacturaIncobrableOPerdonada(facturaId: UUID, motivo: String)

activate DB

DB \--\> SrvFact : ok: Boolean

deactivate DB

&nbsp;

&nbsp;

opt Generar Nota de Crédito / Abono

&nbsp;&nbsp;&nbsp;&nbsp;create participant ": NotaAbono" as Abono

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \--\>\> Abono : NotaAbono(facturaId, motivo)

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> DB : guardarFacturaAbono(Abono)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SrvFact : abonoGuardado: NotaAbono

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> GenDocs : emitirDocumento(abonoGuardado)

&nbsp;&nbsp;&nbsp;&nbsp;activate GenDocs

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \-\> Storage : guardarPDF(abonoGuardado.id, binario: Byte\[\])

&nbsp;&nbsp;&nbsp;&nbsp;activate Storage

&nbsp;&nbsp;&nbsp;&nbsp;Storage \--\> GenDocs : rutaPDF: String

&nbsp;&nbsp;&nbsp;&nbsp;deactivate Storage

&nbsp;&nbsp;&nbsp;&nbsp;GenDocs \--\> SrvFact : rutaPDF: String

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GenDocs

end

&nbsp;

&nbsp;

SrvFact \-\>\> AuditSvc : registrarEventoFactura(evento: "PERDON\_DEUDA", facturaId: UUID)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

SrvFact \--\> RBAC : deudaSaldada: Boolean

deactivate SrvFact

RBAC \--\> FE : 200 OK

deactivate RBAC

FE \--\> Admin : mostrarConfirmacionPerdonDeuda()

deactivate FE

@enduml

&nbsp;

@startuml

title CU-18: Gestión de Incidencias de Clientes y Facturas

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Operativo" as Op

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": GestorInc" as GestorInc

participant ": GestorCLI" as GestorCLI

participant ": SrvFact" as SrvFact

participant ": GestorNotif" as GestorNotif

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

\== Fase 1: Apertura de Incidencia y Pausa de Recordatorios \==

Op \-\> FE : abrirIncidencia(jwt: String, datosInc: IncidenciaDTO)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Operativo")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> GestorInc : crearIncidencia(datosInc: IncidenciaDTO)

activate GestorInc

GestorInc \-\> GestorCLI : consultarClienteReportante(datosInc.clienteId)

activate GestorCLI

GestorCLI \-\> DB : obtenerCliente(datosInc.clienteId)

activate DB

DB \--\> GestorCLI : cliente: Cliente

deactivate DB

GestorCLI \--\> GestorInc : cliente: Cliente

deactivate GestorCLI

&nbsp;

&nbsp;

GestorInc \-\> SrvFact : vincularFacturaAfectada(datosInc.facturaId)

activate SrvFact

SrvFact \-\> DB : obtenerFactura(datosInc.facturaId)

activate DB

DB \--\> SrvFact : factura: Factura

deactivate DB

SrvFact \--\> GestorInc : factura: Factura

deactivate SrvFact

&nbsp;

&nbsp;

create participant ": Incidencia" as Inc

GestorInc \--\>\> Inc : Incidencia(datosInc, cliente, factura)

&nbsp;

&nbsp;

GestorInc \-\> DB : insertarIncidencia(Inc)

activate DB

DB \--\> GestorInc : incidenciaId: UUID

deactivate DB

&nbsp;

&nbsp;

GestorInc \-\> GestorNotif : pausarRecordatoriosCobro(datosInc.facturaId)

activate GestorNotif

GestorNotif \-\> DB : marcarRecordatoriosPausados(datosInc.facturaId, true)

activate DB

DB \--\> GestorNotif : ok: Boolean

deactivate DB

GestorNotif \--\> GestorInc : pausados: Boolean

deactivate GestorNotif

&nbsp;

&nbsp;

GestorInc \-\>\> AuditSvc : registrarEventoIncidencia(evento: "APERTURA\_INCIDENCIA", incidenciaId: UUID)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

GestorInc \--\> RBAC : incidenciaId: UUID

deactivate GestorInc

RBAC \--\> FE : 201 Created (incidenciaId: UUID)

deactivate RBAC

FE \--\> Op : mostrarIncidenciaAbierta()

deactivate FE

&nbsp;

&nbsp;

\== Fase 2: Resolución de Incidencia \==

Op \-\> FE : cerrarIncidencia(jwt: String, incidenciaId: UUID, resolucion: String)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Operativo")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> GestorInc : cerrarIncidencia(incidenciaId: UUID, resolucion: String)

activate GestorInc

GestorInc \-\> DB : actualizarEstadoCerrada(incidenciaId: UUID, resolucion: String)

activate DB

DB \--\> GestorInc : facturaId: UUID

deactivate DB

&nbsp;

&nbsp;

GestorInc \-\> GestorNotif : reanudarRecordatoriosCobro(facturaId: UUID)

activate GestorNotif

GestorNotif \-\> DB : marcarRecordatoriosPausados(facturaId: UUID, false)

activate DB

DB \--\> GestorNotif : ok: Boolean

deactivate DB

GestorNotif \--\> GestorInc : reanudados: Boolean

deactivate GestorNotif

&nbsp;

&nbsp;

GestorInc \-\>\> AuditSvc : registrarEventoIncidencia(evento: "CIERRE\_INCIDENCIA", incidenciaId: UUID)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

GestorInc \--\> RBAC : cerrada: Boolean

deactivate GestorInc

RBAC \--\> FE : 200 OK

deactivate RBAC

FE \--\> Op : actualizarPanelIncidencias()

deactivate FE

@enduml

&nbsp;

@startuml

title CU-19: Gestión de Plantillas de Notificación

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Admin" as Admin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": ConfigApr" as ConfigApr

participant ": SrvFact" as SrvFact

participant ": GestorNotif" as GestorNotif

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Admin \-\> FE : actualizarPlantilla(jwt: String, plantillaId: UUID, contenido: String)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> ConfigApr : gestionarPlantillaNotificacion(plantillaId: UUID, contenido: String)

activate ConfigApr

ConfigApr \-\> SrvFact : delegarActualizacionPlantilla(plantillaId: UUID, contenido: String)

activate SrvFact

SrvFact \-\> GestorNotif : guardarPlantilla(plantillaId: UUID, contenido: String)

activate GestorNotif

GestorNotif \-\> DB : actualizarPlantillaEnBD(plantillaId: UUID, contenido: String)

activate DB

DB \--\> GestorNotif : ok: Boolean

deactivate DB

GestorNotif \--\> SrvFact : plantillaActualizada: Boolean

deactivate GestorNotif

SrvFact \--\> ConfigApr : ok: Boolean

deactivate SrvFact

&nbsp;

&nbsp;

ConfigApr \-\>\> AuditSvc : registrarEventoConfig(evento: "ACTUALIZACION\_PLANTILLA", plantillaId: UUID)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

ConfigApr \--\> RBAC : confirmacion: Boolean

deactivate ConfigApr

RBAC \--\> FE : 200 OK

deactivate RBAC

FE \--\> Admin : mostrarConfirmacionGuardado()

deactivate FE

@enduml

&nbsp;

@startuml

title CU-20: Envío de Recordatorios de Pago e Impago

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

participant ": CronJob" as Cron

participant ": SrvFact" as SrvFact

participant ": GestorNotif" as GestorNotif

participant ": GestorCLI" as GestorCLI

participant ": NotifAdp" as NotifAdp

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Cron \-\> SrvFact : procesarRecordatoriosImpago(fecha: Date)

activate SrvFact

SrvFact \-\> DB : obtenerFacturasVencidas(fecha: Date)

activate DB

DB \--\> SrvFact : facturasImpagadas: List\<Factura\>

deactivate DB

&nbsp;

&nbsp;

loop Por cada factura impagada

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> GestorNotif : solicitarEnvioRecordatorio(factura: Factura)

&nbsp;&nbsp;&nbsp;&nbsp;activate GestorNotif

&nbsp;&nbsp;&nbsp;&nbsp;GestorNotif \-\> DB : verificarPausaYRecuperarPlantilla(factura.id)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> GestorNotif : estadoYPlantilla: PlantillaRecordatorioDTO

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;alt Recordatorios activos (no pausados por incidencia)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GestorNotif \-\> GestorCLI : obtenerContactoYCanales(factura.clienteId)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;activate GestorCLI

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \-\> DB : consultarContactosPreferidos(factura.clienteId)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> GestorCLI : contacto: ContactoDTO

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GestorCLI \--\> GestorNotif : contacto: ContactoDTO

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;deactivate GestorCLI

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GestorNotif \-\>\> NotifAdp : despacharMensaje(contacto: ContactoDTO, estadoYPlantilla.contenido)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GestorNotif \-\> DB : registrarEnvioYContadorAvisos(factura.id)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> GestorNotif : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;end

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;GestorNotif \--\> SrvFact : procesado: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate GestorNotif

end

&nbsp;

&nbsp;

SrvFact \-\>\> AuditSvc : registrarEventoFactura(evento: "ENVIO\_RECORDATORIOS\_IMPAGO", fecha: Date)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

SrvFact \--\> Cron : resumenEnvios: ResumenDTO

deactivate SrvFact

@enduml

&nbsp;

@startuml

title CU-21: Visualización de Dashboards e Indicadores

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Admin" as Admin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": SrvFact" as SrvFact

participant ": GestorCLI" as GestorCLI

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Admin \-\> FE : cargarDashboard(jwt: String, periodo: PeriodoDTO)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> SrvFact : obtenerKPIsFacturacionYMorosidad(periodo: PeriodoDTO)

activate SrvFact

SrvFact \-\> DB : consultarMetricasFacturacion(periodo: PeriodoDTO)

activate DB

DB \--\> SrvFact : kpisFacturacion: KPIFacturacionDTO

deactivate DB

SrvFact \--\> RBAC : kpisFacturacion: KPIFacturacionDTO

deactivate SrvFact

&nbsp;

&nbsp;

RBAC \-\> GestorCLI : obtenerKPIsCarteraClientes(periodo: PeriodoDTO)

activate GestorCLI

GestorCLI \-\> DB : consultarMetricasClientes(periodo: PeriodoDTO)

activate DB

DB \--\> GestorCLI : kpisClientes: KPIClientesDTO

deactivate DB

GestorCLI \--\> RBAC : kpisClientes: KPIClientesDTO

deactivate GestorCLI

&nbsp;

&nbsp;

RBAC \--\> FE : 200 OK (kpisFacturacion, kpisClientes)

deactivate RBAC

FE \-\> FE : renderizarGraficosEIndicadores()

FE \--\> Admin : mostrarDashboardEjecutivo()

deactivate FE

@enduml

&nbsp;

@startuml

title CU-22: Generación de Informes y Paquetes para Gestoría

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Admin" as Admin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": GestoriaFac" as GestoriaFac

participant ": GenDocs" as GenDocs

database ": Storage" as Storage

&nbsp;

&nbsp;

Admin \-\> FE : exportarPaqueteGestoria(jwt: String, trimestre: Integer, anio: Integer)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> GestoriaFac : exportarZIP(trimestre: Integer, anio: Integer)

activate GestoriaFac

GestoriaFac \-\> GenDocs : empaquetarMasivo(trimestre: Integer, anio: Integer)

activate GenDocs

GenDocs \-\> Storage : recuperarPDFsYGuardarZIP(trimestre: Integer, anio: Integer)

activate Storage

Storage \--\> GenDocs : archivoZIP: FileStream

deactivate Storage

GenDocs \--\> GestoriaFac : archivoZIP: FileStream

deactivate GenDocs

&nbsp;

&nbsp;

GestoriaFac \--\> RBAC : archivoZIP: FileStream

deactivate GestoriaFac

RBAC \--\> FE : 200 OK (archivoZIP: FileStream)

deactivate RBAC

FE \--\> Admin : descargarZIP("Gestoria\_Q" \+ trimestre \+ "\_" \+ anio \+ ".zip")

deactivate FE

@enduml

&nbsp;

@startuml

title CU-23: Configuración Parámetros Globales del Sistema

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Superadmin" as Superadmin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": ConfigApr" as ConfigApr

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Superadmin \-\> FE : actualizarParametrosGlobales(jwt: String, config: ConfigGlobalDTO)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Superadmin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> ConfigApr : actualizarConfiguracionGlobal(config: ConfigGlobalDTO)

activate ConfigApr

ConfigApr \-\> DB : guardarConfiguracionGlobal(config: ConfigGlobalDTO)

activate DB

DB \--\> ConfigApr : ok: Boolean

deactivate DB

&nbsp;

&nbsp;

ConfigApr \-\>\> AuditSvc : registrarEventoConfig(evento: "CONFIG\_GLOBAL\_ACTUALIZADA", Superadmin.id)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

ConfigApr \--\> RBAC : configuracionGuardada: ConfigGlobalDTO

deactivate ConfigApr

RBAC \--\> FE : 200 OK (configuracionGuardada: ConfigGlobalDTO)

deactivate RBAC

FE \--\> Superadmin : mostrarConfirmacionGlobal()

deactivate FE

@enduml

&nbsp;

@startuml

title CU-24: Consulta de Registros de Auditoría

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Superadmin" as Superadmin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": ConfigApr" as ConfigApr

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Superadmin \-\> FE : consultarAuditoria(jwt: String, filtros: FiltroAuditDTO)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Superadmin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> ConfigApr : solicitarRegistrosAuditoria(filtros: FiltroAuditDTO)

activate ConfigApr

ConfigApr \-\> AuditSvc : consultarLogsInmutables(filtros: FiltroAuditDTO)

activate AuditSvc

AuditSvc \-\> DB : ejecutarConsultaAuditoria(filtros: FiltroAuditDTO)

activate DB

DB \--\> AuditSvc : registros: Page\<AuditEvent\>

deactivate DB

AuditSvc \--\> ConfigApr : registros: Page\<AuditEvent\>

deactivate AuditSvc

&nbsp;

&nbsp;

ConfigApr \--\> RBAC : registros: Page\<AuditEvent\>

deactivate ConfigApr

RBAC \--\> FE : 200 OK (registros: Page\<AuditEvent\>)

deactivate RBAC

FE \--\> Superadmin : mostrarTablaTrazabilidad(registros)

deactivate FE

@enduml

&nbsp;

@startuml

title CU-25: Integración API Externa (Consultas Autorizadas JWT)

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": ClienteAPI" as Ext

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": SrvFact" as SrvFact

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

&nbsp;

&nbsp;

Ext \-\> RBAC : peticionAPIExterna(jwt: String, clienteId: UUID)

activate RBAC

&nbsp;

&nbsp;

RBAC \-\> SessionMgr : validarSesionYRol(jwt: String, rolRequerido: "Operativo")

activate SessionMgr

SessionMgr \-\> DB : verificarTokenYPermisos(jwt: String)

activate DB

DB \--\> SessionMgr : valido: Boolean

deactivate DB

&nbsp;

&nbsp;

alt Token válido y rol autorizado

&nbsp;&nbsp;&nbsp;&nbsp;SessionMgr \--\> RBAC : autorizado: Boolean \= true

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SessionMgr

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;RBAC \-\> SrvFact : consultarEstadoCuentaCliente(clienteId: UUID)

&nbsp;&nbsp;&nbsp;&nbsp;activate SrvFact

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\> DB : obtenerSaldoYFacturasCliente(clienteId: UUID)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> SrvFact : estadoCuenta: EstadoCuentaDTO

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \-\>\> AuditSvc : registrarEventoFactura(evento: "CONSULTA\_EXTERNA\_ESTADO\_CUENTA", clienteId: UUID)

&nbsp;&nbsp;&nbsp;&nbsp;activate AuditSvc

&nbsp;&nbsp;&nbsp;&nbsp;AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

&nbsp;&nbsp;&nbsp;&nbsp;activate DB

&nbsp;&nbsp;&nbsp;&nbsp;DB \--\> AuditSvc : ok: Boolean

&nbsp;&nbsp;&nbsp;&nbsp;deactivate DB

&nbsp;&nbsp;&nbsp;&nbsp;deactivate AuditSvc

&nbsp;

&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;SrvFact \--\> RBAC : estadoCuenta: EstadoCuentaDTO

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SrvFact

&nbsp;&nbsp;&nbsp;&nbsp;RBAC \--\> Ext : 200 OK (estadoCuenta: EstadoCuentaDTO)

&nbsp;&nbsp;&nbsp;&nbsp;deactivate RBAC

else Token inválido o sin permisos

&nbsp;&nbsp;&nbsp;&nbsp;SessionMgr \--\> RBAC : errorAcceso: ForbiddenException

&nbsp;&nbsp;&nbsp;&nbsp;deactivate SessionMgr

&nbsp;&nbsp;&nbsp;&nbsp;RBAC \--\> Ext : 403 Forbidden (mensaje: String)

&nbsp;&nbsp;&nbsp;&nbsp;deactivate RBAC

end

@enduml

&nbsp;

@startuml

title CU-26: Cierre de Periodos Fiscales y Archivo de Cierre

autonumber

&nbsp;

&nbsp;

skinparam sequence {

&nbsp;&nbsp;&nbsp;&nbsp;ArrowColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;LifeLineBackgroundColor \#FF7B29

&nbsp;&nbsp;&nbsp;&nbsp;ActorBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;ParticipantFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseBackgroundColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;DatabaseFontColor \#FFFFFF

&nbsp;&nbsp;&nbsp;&nbsp;GroupBorderColor \#1E212B

&nbsp;&nbsp;&nbsp;&nbsp;GroupBackgroundColor \#F4F5F7

}

&nbsp;

&nbsp;

actor ": Superadmin" as Superadmin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": ConfigApr" as ConfigApr

participant ": SrvFact" as SrvFact

participant ": GenDocs" as GenDocs

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

database ": Storage" as Storage

&nbsp;

&nbsp;

Superadmin \-\> FE : cerrarPeriodoFiscal(jwt: String, anio: Integer)

activate FE

&nbsp;

&nbsp;

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Superadmin")

&nbsp;

&nbsp;

activate RBAC

RBAC \-\> ConfigApr : cerrarPeriodoFiscal(anio: Integer)

activate ConfigApr

ConfigApr \-\> SrvFact : bloquearPeriodoContable(anio: Integer)

activate SrvFact

SrvFact \-\> DB : bloquearFacturasDelPeriodo(anio: Integer)

activate DB

DB \--\> SrvFact : facturasPeriodo: List\<Factura\>

deactivate DB

&nbsp;

&nbsp;

SrvFact \-\> GenDocs : generarPaqueteCierreFiscal(anio: Integer, facturasPeriodo: List\<Factura\>)

activate GenDocs

GenDocs \-\> Storage : guardarArchivoPrivadoCierre(anio: Integer, paquete: Byte\[\])

activate Storage

Storage \--\> GenDocs : rutaArchivoCierre: String

deactivate Storage

GenDocs \--\> SrvFact : rutaArchivoCierre: String

deactivate GenDocs

&nbsp;

&nbsp;

SrvFact \--\> ConfigApr : periodoBloqueado: Boolean

deactivate SrvFact

&nbsp;

&nbsp;

ConfigApr \-\> DB : marcarPeriodoFiscalCerrado(anio: Integer)

activate DB

DB \--\> ConfigApr : ok: Boolean

deactivate DB

&nbsp;

&nbsp;

ConfigApr \-\>\> AuditSvc : registrarEventoConfig(evento: "CIERRE\_PERIODO\_FISCAL", anio: Integer)

activate AuditSvc

AuditSvc \-\> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB \--\> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

&nbsp;

&nbsp;

ConfigApr \--\> RBAC : cierreCompletado: Boolean

deactivate ConfigApr

RBAC \--\> FE : 200 OK

deactivate RBAC

FE \--\> Superadmin : mostrarConfirmacionCierrePeriodo()

deactivate FE

@enduml

&nbsp;

&nbsp;