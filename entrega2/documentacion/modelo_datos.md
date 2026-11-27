# Modelo físico de datos

## Alcance
El presente documento define el diseño físico de la base de datos relacional del sistema **FACTURBINEH**, compuesto por **19 tablas normalizadas (3FN)** agrupadas en cinco dominios funcionales:
1. **Seguridad y control de acceso:** `ROL`, `USUARIO`.
2. **Gestión comercial y clientes:** `TARIFA_PLAN`, `CLIENTE`, `ASIGNACION_CLIENTE`, `CONTACTO`.
3. **Motor de facturación y contabilidad:** `ENTIDAD_EMISORA`, `PERIODO_FISCAL`, `PRESUPUESTO`, `LINEA_PRESUPUESTO`, `FACTURA`, `LINEA_FACTURA`.
4. **Cobros, aprobaciones e incidencias:** `MOVIMIENTO_PAGO`, `SOLICITUD_APROBACION`, `INCIDENCIA`.
5. **Notificaciones, auditoría y parámetros:** `PLANTILLA_NOTIFICACION`, `NOTIFICACION`, `REGISTRO_AUDITORIA`, `CONFIGURACION_GLOBAL`.

Este esquema da soporte a la gestión integral del ciclo de vida de clientes, facturación recurrente y puntual multi-entidad, presupuestos, conciliación de cobros, flujos de aprobación con segregación de funciones, gestión de incidencias, envío automatizado de comunicaciones, cierre de periodos fiscales y auditoría inmutable (CU-01 a CU-26).

---

## Justificación del modelo

### 1. Seguridad, roles y asignaciones
- `ROL (1)` — `USUARIO (0..N)`: Un rol define un conjunto de permisos aplicable a múltiples usuarios; cada usuario pertenece obligatoriamente a un único rol (`id_rol`).
- `USUARIO (1)` — `ASIGNACION_CLIENTE (0..N)` y `CLIENTE (1)` — `ASIGNACION_CLIENTE (0..N)`: Tabla asociativa que resuelve la relación muchos a muchos entre usuarios internos y clientes, permitiendo distinguir al titular principal (`es_titular_principal`) de delegaciones temporales con fechas de inicio y fin (`fecha_inicio`, `fecha_fin`).

### 2. Clientes, contactos y tarifas
- `TARIFA_PLAN (1)` — `CLIENTE (0..N)`: Una tarifa o rango anual puede aplicar a varios clientes; un cliente tiene asignada una tarifa opcional (`id_tarifa`) hasta completar su configuración comercial.
- `CLIENTE (1)` — `CONTACTO (1..N)`: Todo cliente activo requiere al menos un contacto de facturación (`es_principal_facturacion`) y puede registrar múltiples personas de contacto con canales autorizados.
- `CLIENTE (0..1)` — `CLIENTE (0..1)`: Relación recursiva (`id_cliente_absorbente`) que permite fusionar fichas duplicadas conservando la trazabilidad histórica sin destruir registros contables.

### 3. Presupuestos, facturación y periodos fiscales
- `CLIENTE (1)` — `PRESUPUESTO (0..N)` y `ENTIDAD_EMISORA (1)` — `PRESUPUESTO (0..N)`: Cada presupuesto se emite desde una entidad emisora concreta hacia un cliente.
- `PRESUPUESTO (1)` — `LINEA_PRESUPUESTO (1..N)`: Todo presupuesto desglosa al menos una línea de detalle, normalizando el cálculo antes de su conversión.
- `PRESUPUESTO (0..1)` — `FACTURA (0..1)`: Un presupuesto aceptado puede convertirse en una única factura (`id_presupuesto_origen`), reutilizando sus conceptos.
- `CLIENTE (1)` — `FACTURA (0..N)`, `ENTIDAD_EMISORA (1)` — `FACTURA (0..N)` y `PERIODO_FISCAL (1)` — `FACTURA (0..N)`: Toda factura pertenece a un cliente, a una sociedad emisora y a un periodo contable para controlar bloqueos trimestrales.
- `USUARIO (0..1)` — `PERIODO_FISCAL (0..N)`: Registra qué administrador o superadministrador ejecutó el cierre fiscal (`id_usuario_cierre`).
- `FACTURA (1)` — `LINEA_FACTURA (1..N)`: Toda factura contiene al menos una línea de detalle con cantidades, precios unitarios, descuentos e impuestos.
- `FACTURA (0..1)` — `FACTURA (0..1)`: Relación recursiva (`id_factura_original`) que vincula una factura rectificativa con la factura original sin modificar esta última.

### 4. Cobros, aprobaciones e incidencias
- `FACTURA (1)` — `MOVIMIENTO_PAGO (0..N)`: Una factura puede recibir cero, uno o múltiples pagos (conciliación total, parcial o detección de cobros duplicados).
- `CLIENTE (1)` — `SOLICITUD_APROBACION (0..N)` y `FACTURA (0..1)` — `SOLICITUD_APROBACION (0..N)`: Toda solicitud de excepción comercial o condonación de deuda afecta a un cliente y, opcionalmente, a una factura específica.
- `USUARIO (1)` — `SOLICITUD_APROBACION (0..N)` (como `id_solicitante`) y `USUARIO (0..1)` — `SOLICITUD_APROBACION (0..N)` (como `id_aprobador`): Doble clave foránea que separa explícitamente al usuario que crea la solicitud del responsable que la resuelve.
- `CLIENTE (1)` — `INCIDENCIA (0..N)`, `FACTURA (0..1)` — `INCIDENCIA (0..N)` y `USUARIO (1)` — `INCIDENCIA (0..N)`: Cada incidencia está vinculada a un cliente, opcionalmente a una factura en disputa, y asignada a un usuario responsable (`id_responsable`).

### 5. Notificaciones, auditoría y configuración
- `PLANTILLA_NOTIFICACION (1)` — `NOTIFICACION (0..N)`, `CONTACTO (1)` — `NOTIFICACION (0..N)`, `CLIENTE (1)` — `NOTIFICACION (0..N)` y `FACTURA (0..1)` — `NOTIFICACION (0..N)`: Trazabilidad completa de cada envío automatizado (plantilla versionada empleada, destinatario, cliente y factura reclamada).
- `USUARIO (0..1)` — `REGISTRO_AUDITORIA (0..N)`: Vincula cada evento de negocio o acción técnica con el usuario que la originó (permitiendo `NULL` en procesos automáticos del sistema/cron).
- `CONFIGURACION_GLOBAL`: Entidad independiente de parámetros clave-valor (`clave_parametro`, `valor_parametro`) que evita acoplar claves foráneas a todas las tablas operativas.

---

## Reglas y restricciones

Dado que la notación gráfica Entidad-Relación no valida automáticamente la lógica de dominio ni las restricciones de negocio complejas, deben implementarse mediante *constraints* SQL (`CHECK`, `UNIQUE`, `TRIGGER`) y reglas en la capa de aplicación:

### Integridad de dominio y valores (`CHECK`)
- **Dominios enumerados:**
  - `ROL.nombre_rol` $\in$ `{'SUPERADMINISTRADOR', 'ADMINISTRADOR', 'USUARIO_OPERATIVO'}`.
  - `USUARIO.estado` $\in$ `{'ACTIVO', 'INACTIVO', 'BLOQUEADO'}`.
  - `CLIENTE.estado_comercial` $\in$ `{'PENDIENTE_ACTIVACION', 'ACTIVO', 'PAUSADO', 'BAJA_SOLICITADA', 'INACTIVO', 'BLOQUEADO_ADMIN', 'FUSIONADO'}`.
  - `PERIODO_FISCAL.estado` $\in$ `{'ABIERTO', 'CERRADO'}`.
  - `PRESUPUESTO.estado` $\in$ `{'BORRADOR', 'ENVIADO', 'ACEPTADO', 'RECHAZADO'}`.
  - `FACTURA.tipo_factura` $\in$ `{'RECURRENTE', 'PUNTUAL', 'RECTIFICATIVA'}` y `FACTURA.estado` $\in$ `{'BORRADOR', 'EMITIDA', 'PAGADA', 'PARCIAL', 'VENCIDA', 'CONDONADA'}`.
  - `MOVIMIENTO_PAGO.estado_conciliacion` $\in$ `{'TOTAL', 'PARCIAL', 'DISCREPANCIA', 'DOBLE_PAGO'}`.
  - `SOLICITUD_APROBACION.tipo_solicitud` $\in$ `{'DESCUENTO', 'CARENCIA', 'CONDONACION_DEUDA', 'CAMBIO_TARIFA'}` y `estado` $\in$ `{'PENDIENTE', 'APROBADA', 'RECHAZADA'}`.
  - `INCIDENCIA.prioridad` $\in$ `{'BAJA', 'MEDIA', 'ALTA', 'CRITICA'}` y `INCIDENCIA.estado` $\in$ `{'ABIERTA', 'CERRADA'}`.
  - `NOTIFICACION.canal` $\in$ `{'EMAIL', 'WHATSAPP'}` y `estado_entrega` $\in$ `{'ENVIADO', 'ENTREGADO', 'REBOTE'}`.
- **Validaciones cuantitativas y temporales:**
  - `USUARIO.intentos_fallidos >= 0`.
  - `TARIFA_PLAN.precio_rango >= 0` y `umbral_facturacion_anual >= 0`.
  - `CLIENTE.vencimiento_defecto_dias >= 0`; si `estado_comercial = 'PAUSADO'`, `fecha_inicio_pausa` es obligatoria y `fecha_fin_pausa >= fecha_inicio_pausa`.
  - `ASIGNACION_CLIENTE.fecha_fin IS NULL OR fecha_fin >= fecha_inicio`.
  - `LINEA_FACTURA.cantidad > 0`, `precio_unitario >= 0`, `0 <= descuento_aplicado <= 100`, `porcentaje_impuesto >= 0` (mismas reglas en `LINEA_PRESUPUESTO`).
  - `FACTURA.importe_total > 0` (excepto rectificativas negativas si aplica normativa) y `fecha_vencimiento >= fecha_emision`.
  - `MOVIMIENTO_PAGO.importe_recibido > 0`.
  - `SOLICITUD_APROBACION.porcentaje_descuento` entre `0` y `100`; `meses_carencia >= 0`.

### Reglas de negocio transaccionales y de seguridad
- **Segregación de funciones (RB-05, FR-46):** Cuando el importe o descuento supera el umbral configurado en `CONFIGURACION_GLOBAL`, debe cumplirse `SOLICITUD_APROBACION.id_solicitante <> SOLICITUD_APROBACION.id_aprobador`.
- **Inmutabilidad por cierre fiscal (FR-12, CU-26):** Si `PERIODO_FISCAL.estado = 'CERRADO'`, se prohíbe cualquier `INSERT`, `UPDATE` o `DELETE` sobre las facturas vinculadas a dicho `id_periodo`, obligando a emitir una factura rectificativa (`tipo_factura = 'RECTIFICATIVA'`, `id_factura_original IS NOT NULL`).
- **Correlatividad y emisión (CU-12, CU-13):** `FACTURA.numero_correlativo` solo se asigna al pasar de `BORRADOR` a `EMITIDA`, garantizando secuencia sin saltos ni duplicados por `ENTIDAD_EMISORA` y `serie`.
- **Contacto mínimo obligatorio (FR-43, CU-06):** Un cliente con `estado_comercial = 'ACTIVO'` debe mantener al menos un registro en `CONTACTO` con `activo = TRUE` y `es_principal_facturacion = TRUE`.
- **Cierre de incidencias y condonaciones (CU-17, CU-18):** No se permite actualizar `INCIDENCIA.estado = 'CERRADA'` si `resolucion` es nula o vacía; no se permite aprobar una condonación si `FACTURA.motivo_perdon_deuda` o `SOLICITUD_APROBACION.motivo_justificacion` están vacíos.
- **Inmutabilidad de auditoría (CU-24):** La tabla `REGISTRO_AUDITORIA` opera en modo *append-only* (solo `INSERT` y `SELECT`); se revocan permisos de `UPDATE` y `DELETE` incluso para administradores.
- **Exactitud decimal (NFR-18, RB-04):** Todos los importes monetarios utilizan tipo `DECIMAL(10,2)` o `DECIMAL(12,2)` (nunca punto flotante `FLOAT`/`DOUBLE`) aplicando redondeo uniforme (*half-up* a 2 decimales).

---

## Trazabilidad

| Requisitos / Casos de Uso | Tablas / Columnas principales | Observación |
|---|---|---|
| **FR-03, FR-06, FR-22, FR-23, FR-24, NFR-07, NFR-08** / CU-01, CU-02, CU-03 | `ROL`, `USUARIO` (`correo_corporativo`, `password_hash`, `estado`, `intentos_fallidos`, `token_recuperacion`, `expiracion_token`) | Autenticación corporativa, control de roles (Superadmin, Admin, Operativo/Lector), bloqueo por intentos fallidos y recuperación de contraseña. |
| **FR-07, NFR-08, RB-02** / CU-04 | `ASIGNACION_CLIENTE` (`id_cliente`, `id_usuario`, `fecha_inicio`, `fecha_fin`, `es_titular_principal`) | Asignación titular de clientes a usuarios operativos y gestión de delegaciones temporales con caducidad automática. |
| **FR-13, FR-17, FR-18, FR-19, FR-38, FR-39, FR-40, FR-41, FR-42, FR-43** / CU-05, CU-06, CU-07, CU-08, CU-09 | `CLIENTE` (`identificador_fiscal`, `estado_comercial`, `id_cliente_absorbente`, `es_duplicado_falso`, `fecha_inicio_pausa`, `fecha_fin_pausa`), `CONTACTO` (`es_principal_facturacion`, `activo`) | Alta con detección de duplicados, borrado lógico de contactos, pausas temporales, baja comercial con retención fiscal y fusión de clientes duplicados. |
| **FR-09** / CU-10 | `TARIFA_PLAN` (`precio_rango`, `umbral_facturacion_anual`), `CLIENTE` (`id_tarifa`) | Evaluación anual de ingresos del cliente frente a los umbrales de tarifa para proponer cambios de plan. |
| **FR-44, FR-45, FR-46, RB-05** / CU-11, CU-17 | `SOLICITUD_APROBACION` (`id_solicitante`, `id_aprobador`, `tipo_solicitud`, `estado`, `porcentaje_descuento`, `meses_carencia`, `motivo_justificacion`), `FACTURA` (`motivo_perdon_deuda`) | Bandeja de aprobaciones para excepciones comerciales y condonación de deudas con segregación de funciones. |
| **FR-01, FR-02, FR-10, FR-11, FR-12, FR-26, FR-36, NFR-05, NFR-18, RB-03** / CU-12, CU-13, CU-14, CU-15 | `ENTIDAD_EMISORA`, `PRESUPUESTO`, `LINEA_PRESUPUESTO`, `FACTURA` (`serie`, `numero_correlativo`, `tipo_factura`, `id_factura_original`, `id_presupuesto_origen`, `moneda`, `ruta_pdf_generado`), `LINEA_FACTURA` | Facturación recurrente (con prorrateo inicial) y puntual multi-entidad, conversión de presupuestos, rectificativas vinculadas y descarga de PDF. |
| **FR-02, NFR-10** / CU-16 | `MOVIMIENTO_PAGO` (`referencia_pasarela`, `origen`, `importe_recibido`, `estado_conciliacion`), `FACTURA` (`estado`) | Conciliación automática o manual de cobros (Stripe, Revolut, transferencia), pagos parciales y detección de dobles pagos. |
| **FR-34** / CU-18 | `INCIDENCIA` (`id_cliente`, `id_factura`, `id_responsable`, `prioridad`, `estado`, `pausar_notificaciones_cobro`, `resolucion`) | Gestión de disputas e incidencias operativas con pausa selectiva de recordatorios de cobro. |
| **FR-05, FR-31, FR-32, FR-35** / CU-19, CU-20 | `PLANTILLA_NOTIFICACION` (`tipo_notificacion`, `idioma`, `cuerpo_mensaje`, `version`), `NOTIFICACION` (`canal`, `destino`, `estado_entrega`) | Plantillas multi-idioma versionadas y registro de envíos/rebotes de avisos previos y recordatorios de impago por Email y WhatsApp. |
| **FR-04, FR-20, FR-25, FR-29, FR-30, FR-33, FR-37, NFR-09, RB-06** / CU-21, CU-22, CU-23 | `CONFIGURACION_GLOBAL` (`clave_parametro`, `valor_parametro`), `FACTURA`, `MOVIMIENTO_PAGO`, `CLIENTE` | Parametrización global (impuestos, divisas, tipos de cambio, festivos, umbrales, objetivos) y base de datos para analíticas e informes. |
| **FR-14, FR-15, FR-16, FR-20** / CU-24, CU-25 | `REGISTRO_AUDITORIA` (`id_usuario`, `accion_realizada`, `tabla_afectada`, `registro_id`, `valor_anterior`, `valor_nuevo`, `ip_origen`) | Auditoría de negocio inmutable para trazabilidad de cambios, línea temporal del cliente y llamadas externas vía API. |
| **FR-07, FR-12, NFR-15** / CU-26 | `PERIODO_FISCAL` (`trimestre`, `estado`, `id_usuario_cierre`, `fecha_cierre`) | Cierre contable trimestral/mensual que bloquea modificaciones retroactivas en facturas emitidas. |

---

## Decisiones abiertas

1. **Almacenamiento de documentos anexos múltiples (CU-15 A2 y CU-18):** Actualmente `FACTURA` almacena la ruta del PDF oficial (`ruta_pdf_generado`). Queda pendiente confirmar si los justificantes, pedidos firmados o adjuntos de incidencias requieren una tabla auxiliar `DOCUMENTO_ADJUNTO` (con asociación polimórfica o claves foráneas opcionales a `FACTURA` e `INCIDENCIA`) o si se gestionarán mediante almacenamiento de objetos referenciado en JSON.
2. **Hilo de comentarios internos y menciones `@usuario` en incidencias (CU-18):** El modelo actual recoge `descripcion` y `resolucion` en `INCIDENCIA`. Si se requiere persistir un historial cronológico de múltiples comentarios internos/externos con menciones a usuarios, será necesario incorporar una tabla `COMENTARIO_INCIDENCIA`.
3. **Persistencia de favoritos, recientes y preferencias personales de notificación (FR-21, FR-32):** Se plantea resolver mediante tablas ligeras de relación usuario-cliente (`CLIENTE_FAVORITO`) y preferencias por usuario (`PREFERENCIA_USUARIO`), o bien serializarlas dentro de `USUARIO` / almacenamiento local de sesión según el volumen esperado.
4. **Clasificación por proyecto / centro de coste (FR-27, FR-28 — Ampliación):** Dado que el cliente lo marcó como ampliación no obligatoria en el primer despliegue (A-09), no se ha creado aún la tabla `CENTRO_COSTE` ni su FK en `FACTURA` / `LINEA_FACTURA`, aunque el esquema permite añadirla sin alterar las relaciones existentes.
5. **Gestión de credenciales API externa (CU-25):** Se asume que las *API Keys* y el *rate limiting* para la integración con BrainOS se gestionan en la capa de *API Gateway* / variables de entorno (NFR-09), registrando únicamente las operaciones ejecutadas en `REGISTRO_AUDITORIA`.