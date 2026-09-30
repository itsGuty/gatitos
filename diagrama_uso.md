@startuml
left to right direction
actor Lector as g
package Clientes {
  actor "Cliente" as fc
}
package Turbineh {
  usecase "Revisar impagos" as UC1
  usecase "Obtener analíticas" as UC2
  usecase "Crear facturas" as UC3
  usecase "Ver facturas" as UC4
}
fc --> UC4
g --> UC1
g --> UC2
g --> UC3
@enduml
