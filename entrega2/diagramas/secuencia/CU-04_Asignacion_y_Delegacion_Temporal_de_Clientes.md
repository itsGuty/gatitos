# CU-04: Asignación y Delegación Temporal de Clientes

```plantuml
@startuml

title CU-04: Asignación y Delegación Temporal de Clientes

autonumber

skinparam sequence {

    ArrowColor #FF7B29

    LifeLineBorderColor #1E212B

    LifeLineBackgroundColor #FF7B29

    ActorBorderColor #1E212B

    ParticipantBorderColor #1E212B

    ParticipantBackgroundColor #1E212B

    ParticipantFontColor #FFFFFF

    DatabaseBorderColor #1E212B

    DatabaseBackgroundColor #1E212B

    DatabaseFontColor #FFFFFF

    GroupBorderColor #1E212B

    GroupBackgroundColor #F4F5F7

}

actor ": Admin" as Admin

participant ": PanelWeb" as FE

participant ": RBACGuard" as RBAC

participant ": SessionManager" as SessionMgr

participant ": GestorCLI" as GestorCLI

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

Admin -> FE : asignarCliente(jwt: String, clienteId: UUID, operativoId: UUID, fechaFin: Date)

activate FE

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

activate RBAC

RBAC -> GestorCLI : asignarCliente(clienteId: UUID, operativoId: UUID, fechaFin: Date)

activate GestorCLI

GestorCLI -> DB : actualizarAsignacionYDelegacion(clienteId: UUID, operativoId: UUID, fechaFin: Date)

activate DB

DB --> GestorCLI : ok: Boolean

deactivate DB

GestorCLI ->> AuditSvc : registrarEventoCliente(evento: "ASIGNACION_CLIENTE", clienteId: UUID)

activate AuditSvc

AuditSvc -> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB --> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

GestorCLI --> RBAC : asignacionConfirmada: Boolean

deactivate GestorCLI

RBAC --> FE : 200 OK

deactivate RBAC

FE --> Admin : actualizarCarteraClientes()

deactivate FE

@enduml
```
