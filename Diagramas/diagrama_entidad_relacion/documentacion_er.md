# Diagrama físico Entidad-Relación

El diagrama está en [diagrama_ER_turbine.puml](diagrama_ER_turbine.puml) (PlantUML, notación de patas de gallo, la misma que el ejemplo de clase). Modela **solo los datos que la base de datos tiene que guardar**: 27 tablas y 42 claves foráneas. Sale del diagrama de clases de [borrador_clases.md](../diagramas_clases/borrador_clases.md), siguiendo el principio KISS.

## Notación

- `*` delante de una columna: `NOT NULL`.
- `<<PK>>` clave primaria, `<<FK>>` clave foránea, `<<UK>>` valor único.
- Patas de gallo: raya = uno, círculo = cero, pata = muchos. Por ejemplo, `||--o{` es "uno a cero o muchos".

## Criterios KISS

1. **Solo se guardan datos de negocio.** Los procesos (importar clientes, acciones masivas, ejecuciones del Planificador) no son datos: lo que cambian queda en sus tablas y en `REGISTRO_AUDITORIA`.
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
| Usuario | `USUARIO` (los canales preferidos se quedan en una columna, `canal_preferido`) |
| DelegacionTemporal | `DELEGACION_TEMPORAL` (se delegan todos los clientes del titular; para cancelarla antes de tiempo se adelanta `fecha_fin`) |
| Notificacion | `NOTIFICACION` |
| Comentario | `COMENTARIO` (las menciones van en el texto y generan notificaciones) |
| DocumentoAdjunto | `DOCUMENTO_ADJUNTO` + columnas `ruta_*` |
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
| EnvioFactura, RecordatorioImpago | `ENVIO` (columna `tipo`) |
| SolicitudAprobacion | `SOLICITUD_APROBACION` |
| Cobro | `COBRO` |
| Incidencia | `INCIDENCIA` (abierta mientras `fecha_cierre` sea NULL) |
| Informe, PaqueteGestoria | `INFORME` (columna `tipo`; el paquete de la gestoría apunta a su periodo) |
| Sesion | No se guarda: la sesión la gestiona el servidor (token o cookie) |
| ConexionPasarelaCobro | No se guarda: las claves de Stripe/Revolut van en variables de entorno, no en la base de datos |
| CopiaSeguridad | No se guarda: las copias son ficheros que genera la herramienta de backup del motor |
| EjecucionProceso | No se guarda: es el log de la aplicación |
| ImportacionClientes, AccionMasiva | No se guardan: son procesos; el resultado se muestra en pantalla y queda en la auditoría |
| AlertaCorporativa | No se guarda: las alertas son reglas de la aplicación que crean filas en `NOTIFICACION` |

## Valores de los ENUM

| Columna | Valores |
|---|---|
| `USUARIO.rol` | USUARIO, ADMIN, SUPERADMIN |
| `canal` / `canal_preferido` | INTERNO, EMAIL, WHATSAPP |
| `NOTIFICACION.estado` | NO_LEIDA, LEIDA, ATENDIDA |
| `PLANTILLA_MENSAJE.tipo_evento` | EMISION_FACTURA, RECORDATORIO_IMPAGO, ENVIO_PRESUPUESTO, INVITACION_USUARIO, RECUPERACION_PASSWORD |
| `PERIODO_CONTABLE.tipo` | MENSUAL, TRIMESTRAL |
| `CLIENTE.estado` | ACTIVO, BAJA |
| `SUSCRIPCION.estado` | PROGRAMADA, ACTIVA, PAUSADA, CANCELADA |
| `EXCEPCION_COMERCIAL.tipo` | DESCUENTO, MES_GRATIS, PRECIO_PACTADO, NO_PRORRATEAR |
| `PRESUPUESTO.estado` | PENDIENTE, ENVIADO, ACEPTADO, RECHAZADO |
| `FACTURA.tipo` | PUNTUAL, RECURRENTE, RECTIFICATIVA |
| `FACTURA.estado` | BORRADOR, PENDIENTE_APROBACION, EMITIDA, PAGADA, VENCIDA, ANULADA, INCOBRABLE |
| `ENVIO.tipo` | FACTURA, RECORDATORIO |
| `SOLICITUD_APROBACION.tipo` | IMPORTE_MANUAL, DESCUENTO_FACTURA, EXCEPCION_COMERCIAL, VENCIMIENTO_ESPECIAL, FACTURA_IMPORTE_ALTO, CAMBIO_RANGO, RECTIFICATIVA, INCOBRABLE |
| `SOLICITUD_APROBACION.estado` | PENDIENTE, APROBADA, RECHAZADA |
| `COBRO.origen` | STRIPE, REVOLUT, MANUAL |
| `INCIDENCIA.prioridad` | BAJA, MEDIA, ALTA |
| `INFORME.tipo` / `.formato` | LISTADO, PAQUETE_GESTORIA / CSV, PDF, ZIP |

## Restricciones importantes

- **`FACTURA`:**
  - `UNIQUE (serie, numero)` para que la numeración sea correlativa.
  - `UNIQUE (id_suscripcion, mes_facturado)` para no facturar dos veces el mismo mes (CU-29).
  - Solo se puede borrar en `BORRADOR` (CU-14).
- **`COBRO.referencia_externa`** es única, así que el mismo pago no se aplica dos veces (CU-11).
- **`SOLICITUD_APROBACION`:** el aprobador no puede ser el solicitante (CU-27).
- **FK que pertenecen a una cosa u otra:** en `COMENTARIO`, `INCIDENCIA`, `LINEA_FACTURA` y `SOLICITUD_APROBACION` solo una de las FK opcionales puede estar rellena.
- **`CLIENTE.nif` no es único**, porque CU-36 permite repetir un NIF indicando el motivo.
- **`REGISTRO_AUDITORIA`** solo admite insertar y consultar (CU-20).
- **Borrado:** por defecto no se permite borrar una fila referenciada. Las tablas que dependen de otra (`CONTACTO_CLIENTE`, `LINEA_FACTURA`, `CLIENTE_ETIQUETA` y `FAVORITO`) se borran en cascada.
