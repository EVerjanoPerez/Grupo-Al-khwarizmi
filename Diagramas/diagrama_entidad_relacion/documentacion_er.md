# Diagrama físico Entidad-Relación

El diagrama está en [diagrama_ER_turbine.puml](diagrama_ER_turbine.puml) (PlantUML, notación de patas de gallo, la misma que el ejemplo de clase). Modela **solo los datos que la base de datos tiene que guardar**: 34 tablas y 53 claves foráneas. Sale del diagrama de clases de [borrador_clases.md](../diagramas_clases/borrador_clases.md), siguiendo el principio KISS.

## Notación

- `*` delante de una columna: `NOT NULL`.
- `<<PK>>` clave primaria, `<<FK>>` clave foránea, `<<UK>>` valor único.
- Patas de gallo: raya = uno, círculo = cero, pata = muchos. Por ejemplo, `||--o{` es "uno a cero o muchos".

## Criterios KISS

1. **Solo se guardan datos de negocio.** Los procesos que solo cambian otros datos (importar clientes, acciones masivas) no tienen tabla: lo que cambian queda en sus tablas y en `REGISTRO_AUDITORIA`. Sí se guardan las ejecuciones del Planificador y las copias de seguridad, porque se consultan desde la pantalla de salud (CU-20, FR58).
2. **No se guarda lo que se puede calcular:**
   - el saldo pendiente: total menos cobros;
   - si una factura está en disputa: tiene una incidencia de disputa abierta;
   - si un cobro está conciliado: tiene factura;
   - si una excepción está vigente: aprobada y dentro de sus fechas;
   - el total de un presupuesto: suma de sus líneas.
3. **Un único archivo = una columna `ruta_*`** (PDF de la factura, justificante del cobro, evidencia del presupuesto, archivo del informe). La tabla `DOCUMENTO_ADJUNTO` queda solo para los adjuntos libres, que pueden ser varios.
4. **Las tablas que guardan lo mismo se fusionan:** el envío de la factura y el recordatorio de impago son los dos un mensaje enviado sobre una factura, así que van en `ENVIO`.

## De cada clase a su tabla

| Clase | En la base de datos |
|---|---|
| Usuario | `USUARIO` + `USUARIO_CANAL` (un usuario puede elegir varios canales, CU-32) |
| DelegacionTemporal | `DELEGACION_TEMPORAL` + `DELEGACION_CLIENTE` (se guardan los clientes delegados, que pueden ser todos o solo algunos, CU-07 2a; para cancelarla antes de tiempo se adelanta `fecha_fin`) |
| Notificacion | `NOTIFICACION` (`tipo_elemento` + `id_elemento` indican el elemento al que lleva, CU-32 paso 5) |
| Comentario | `COMENTARIO` (las menciones van en el texto y generan notificaciones) |
| DocumentoAdjunto | `DOCUMENTO_ADJUNTO` (con el usuario que lo sube, CU-32) + columnas `ruta_*` |
| ConfiguracionSistema | `CONFIGURACION_SISTEMA` (una sola fila; incluye los días de recordatorio y los códigos de serie) |
| PlantillaMensaje | `PLANTILLA_MENSAJE` |
| CalendarioLaboral, DiaFestivo | `FESTIVO` (el año sale de la fecha) |
| TarifaRango | `TARIFA_RANGO` |
| SerieFacturacion | Columnas `serie` + `numero` de `FACTURA`; el siguiente número es el máximo de la serie + 1 |
| RegistroAuditoria | `REGISTRO_AUDITORIA` |
| PeriodoContable | `PERIODO_CONTABLE` |
| Cliente | `CLIENTE` + `CLIENTE_ETIQUETA` + `FAVORITO` |
| ContactoCliente | `CONTACTO_CLIENTE` |
| RevisionRango | `REVISION_RANGO` (el rango anterior es el de la revisión previa) |
| Suscripcion | `SUSCRIPCION` |
| ExcepcionComercial | `EXCEPCION_COMERCIAL` |
| ProductoCatalogo | `PRODUCTO_CATALOGO` |
| Presupuesto | `PRESUPUESTO` |
| Factura | `FACTURA` |
| LineaFactura | `LINEA_FACTURA` (sirve para facturas y presupuestos) |
| EnvioFactura, RecordatorioImpago | `ENVIO` (columna `tipo`; guarda la plantilla usada, y `id_envio_original` distingue el envío original de sus reintentos, FR35.1) |
| SolicitudAprobacion | `SOLICITUD_APROBACION` (`id_usuario_asignado` guarda a quién se reasigna, CU-27 4a) |
| Cobro | `COBRO` |
| Incidencia | `INCIDENCIA` (abierta mientras `fecha_cierre` sea NULL) |
| Informe, PaqueteGestoria | `INFORME` (columna `tipo`; el paquete de la gestoría apunta a su periodo y lleva `version` si se regenera, FR54; `ruta_archivo` queda vacía mientras se genera en segundo plano) |
| Sesion | No se guarda: la sesión la gestiona el servidor (token o cookie) |
| ConexionPasarelaCobro | `CONEXION_PASARELA` (el superadministrador la configura desde la aplicación, CU-04; las claves se guardan cifradas y la clave de cifrado va en una variable de entorno, nunca en el repositorio, NFR17) |
| CopiaSeguridad | `COPIA_SEGURIDAD` (registro de cada copia, para listarlas, descargarlas y restaurarlas, CU-20 1b; el fichero va en `ruta_archivo`) |
| EjecucionProceso | `EJECUCION_PROCESO` (resultado de cada ejecución del Planificador, CU-19, CU-20 1a y CU-29) |
| ImportacionClientes, AccionMasiva | No se guardan: son procesos; el resultado se muestra en pantalla y queda en la auditoría |
| AlertaCorporativa | `ALERTA_CORPORATIVA` + `ALERTA_DESTINATARIO` (el administrador elige condición, umbral, destinatarios y canal, CU-32 2b; al cumplirse crea filas en `NOTIFICACION`) |

## Valores de los ENUM

| Columna | Valores |
|---|---|
| `USUARIO.rol` | USUARIO, ADMIN, SUPERADMIN |
| `canal` (todas las tablas) | INTERNO, EMAIL, WHATSAPP |
| `NOTIFICACION.estado` | NO_LEIDA, LEIDA, ATENDIDA |
| `NOTIFICACION.prioridad` | BAJA, MEDIA, ALTA |
| `NOTIFICACION.tipo_elemento` | CLIENTE, FACTURA, COBRO, INCIDENCIA, SOLICITUD_APROBACION, REVISION_RANGO, SUSCRIPCION |
| `ALERTA_CORPORATIVA.condicion` | FACTURA_VENCIDA_DIAS, DEUDA_CLIENTE_SUPERIOR, FALLOS_ENVIO, REVISION_SIN_RESOLVER_DIAS |
| `DOCUMENTO_ADJUNTO.formato` | PDF, CSV, PNG, JPG, ZIP |
| `CONEXION_PASARELA.proveedor` / `.entorno` | STRIPE, REVOLUT / SANDBOX, PRODUCCION |
| `EJECUCION_PROCESO.tipo_proceso` | GENERACION_MENSUAL, RECORDATORIOS_IMPAGO, CONSULTA_COBROS, REVISION_ANUAL, COPIA_SEGURIDAD, ACCIONES_PROGRAMADAS |
| `EJECUCION_PROCESO.estado` | PROCESANDO, COMPLETADA, COMPLETADA_CON_ERRORES, FALLIDA |
| `PLANTILLA_MENSAJE.tipo_evento` | EMISION_FACTURA, RECORDATORIO_IMPAGO, ENVIO_PRESUPUESTO, INVITACION_USUARIO, RECUPERACION_PASSWORD |
| `PERIODO_CONTABLE.tipo` | MENSUAL, TRIMESTRAL |
| `CLIENTE.estado` | ACTIVO, BAJA |
| `SUSCRIPCION.estado` | PROGRAMADA, ACTIVA, PAUSADA, CANCELADA |
| `EXCEPCION_COMERCIAL.tipo` | DESCUENTO, MES_GRATIS, PRECIO_PACTADO, NO_PRORRATEAR |
| `PRESUPUESTO.estado` | PENDIENTE, ENVIADO, ACEPTADO, RECHAZADO |
| `FACTURA.tipo` | PUNTUAL, RECURRENTE, RECTIFICATIVA |
| `FACTURA.estado` | BORRADOR, PENDIENTE_APROBACION, EMITIDA, PAGADA, VENCIDA, RECTIFICADA, ANULADA, INCOBRABLE |
| `ENVIO.tipo` | FACTURA, RECORDATORIO |
| `SOLICITUD_APROBACION.tipo` | IMPORTE_MANUAL, DESCUENTO_FACTURA, EXCEPCION_COMERCIAL, VENCIMIENTO_ESPECIAL, FACTURA_IMPORTE_ALTO, CAMBIO_RANGO, RECTIFICATIVA, INCOBRABLE |
| `SOLICITUD_APROBACION.estado` | PENDIENTE, APROBADA, RECHAZADA |
| `COBRO.origen` | STRIPE, REVOLUT, MANUAL |
| `INCIDENCIA.prioridad` | BAJA, MEDIA, ALTA |
| `INFORME.tipo` / `.formato` / `.estado` | LISTADO, PAQUETE_GESTORIA / CSV, PDF, ZIP / EN_PROCESO, LISTO, ERROR |

## Restricciones importantes

- **`FACTURA`:**
  - `UNIQUE (serie, numero)` para que la numeración sea correlativa.
  - `UNIQUE (id_suscripcion, mes_facturado)` solo para las facturas de tipo `RECURRENTE`, para no facturar dos veces el mismo mes (CU-29) sin impedir que una rectificativa copie esos datos.
  - Solo se puede borrar en `BORRADOR` (CU-14).
- **`COBRO.referencia_externa`** es única, así que el mismo pago no se aplica dos veces (CU-11).
- **`SOLICITUD_APROBACION`:** el aprobador no puede ser el solicitante (CU-27).
- **FK que pertenecen a una cosa u otra:** en `COMENTARIO`, `DOCUMENTO_ADJUNTO`, `INCIDENCIA`, `LINEA_FACTURA` y `SOLICITUD_APROBACION` solo una de las FK opcionales puede estar rellena.
- **Relaciones 1 a 1:** `PRESUPUESTO.id_factura`, `SOLICITUD_APROBACION.id_excepcion` y `SOLICITUD_APROBACION.id_revision` son únicas (`<<UK>>`), y también `COPIA_SEGURIDAD.id_ejecucion`.
- **`NOTIFICACION.tipo_elemento` e `id_elemento`** van juntos: o están los dos rellenos o ninguno. No son FK porque pueden apuntar a varias tablas.
- **`CLIENTE.nif` no es único**, porque CU-36 permite repetir un NIF indicando el motivo.
- **`REGISTRO_AUDITORIA`** solo admite insertar y consultar (CU-20).
- **Borrado:** por defecto no se permite borrar una fila referenciada. Las tablas que dependen de otra (`CONTACTO_CLIENTE`, `LINEA_FACTURA`, `CLIENTE_ETIQUETA`, `FAVORITO`, `USUARIO_CANAL`, `DELEGACION_CLIENTE` y `ALERTA_DESTINATARIO`) se borran en cascada.
