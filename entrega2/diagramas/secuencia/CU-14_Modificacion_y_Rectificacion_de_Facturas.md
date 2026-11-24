# CU-14: Modificación y Rectificación de Facturas

```plantuml
@startuml

title CU-14: Modificación y Rectificación de Facturas

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

participant ": SrvFact" as SrvFact

participant ": GenDocs" as GenDocs

participant ": AuditService" as AuditSvc

database ": PostgreSQL" as DB

database ": Storage" as Storage

Admin -> FE : modificarORectificarFactura(jwt: String, facturaId: UUID, cambios: FacturaDTO)

activate FE

ref over FE, SessionMgr : SD-00: Validar Sesión JWT y Rol ("Admin")

activate RBAC

RBAC -> SrvFact : procesarAlteracionFactura(facturaId: UUID, cambios: FacturaDTO)

activate SrvFact

SrvFact -> DB : consultarFactura(facturaId: UUID)

activate DB

DB --> SrvFact : facturaActual: Factura

deactivate DB

alt Factura en estado "Borrador"

    SrvFact -> DB : actualizarBorrador(facturaId: UUID, cambios: FacturaDTO)

    activate DB

    DB --> SrvFact : facturaActualizada: Factura

    deactivate DB

    SrvFact -> GenDocs : emitirDocumento(facturaActualizada: Factura)

    activate GenDocs

    GenDocs -> Storage : guardarPDF(facturaActualizada.id, binario: Byte[])

    activate Storage

    Storage --> GenDocs : rutaPDF: String

    deactivate Storage

    GenDocs --> SrvFact : rutaPDF: String

    deactivate GenDocs

else Factura "Emitida" o "Pagada" (Inmutable)

    create participant ": FacturaRectificativa" as Rect

    SrvFact -->> Rect : FacturaRectificativa(facturaActual, cambios)

    SrvFact -> DB : guardarFacturaRectificativa(Rect)

    activate DB

    DB --> SrvFact : rectificativa: FacturaRectificativa

    deactivate DB

    SrvFact -> GenDocs : emitirDocumento(rectificativa)

    activate GenDocs

    GenDocs -> Storage : guardarPDF(rectificativa.id, binario: Byte[])

    activate Storage

    Storage --> GenDocs : rutaPDF: String

    deactivate Storage

    GenDocs --> SrvFact : rutaPDF: String

    deactivate GenDocs

end

SrvFact ->> AuditSvc : registrarEventoFactura(evento: "MODIFICACION_O_RECTIFICACION", facturaId: UUID)

activate AuditSvc

AuditSvc -> DB : insertarAuditoria(evento: AuditEvent)

activate DB

DB --> AuditSvc : ok: Boolean

deactivate DB

deactivate AuditSvc

SrvFact --> RBAC : resultado: FacturaDTO

deactivate SrvFact

RBAC --> FE : 200 OK (resultado: FacturaDTO)

deactivate RBAC

FE --> Admin : mostrarConfirmacionFactura()

deactivate FE

@enduml
```
