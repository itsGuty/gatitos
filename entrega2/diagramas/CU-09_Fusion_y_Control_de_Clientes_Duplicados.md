# CU-09: Fusión y Control de Clientes Duplicados

```plantuml
@startuml

title CU-09: Fusión y Control de Clientes Duplicados

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

participant ": SrvFact" as SrvFact

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

Admin -> FE : fusionarClientes(jwt: String, origenId: UUID, destinoId: UUID)

activate FE

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

activate RBAC

RBAC -> GestorCLI : fusionarClientes(origenId: UUID, destinoId: UUID)

activate GestorCLI

GestorCLI -> DB : reasignarContactosYMarcarOrigenFusionado(origenId: UUID, destinoId: UUID)

activate DB

DB --> GestorCLI : ok: Boolean

deactivate DB

GestorCLI -> SrvFact : reasignarFacturasPorFusion(origenId: UUID, destinoId: UUID)

activate SrvFact

SrvFact -> DB : transferirFacturasYPresupuestos(origenId: UUID, destinoId: UUID)

activate DB

DB --> SrvFact : ok: Boolean

deactivate DB

SrvFact --> GestorCLI : impactoContableAplicado: Boolean

deactivate SrvFact

GestorCLI ->> AuditSvc : registrarEventoCliente(evento: "FUSION_CLIENTES", origenId: UUID, destinoId: UUID)

activate AuditSvc

AuditSvc -> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB --> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

GestorCLI --> RBAC : fusionCompletada: Boolean

deactivate GestorCLI

RBAC --> FE : 200 OK

deactivate RBAC

FE --> Admin : recargarFichaClienteDestino(destinoId)

deactivate FE

@enduml
```
