# Borrador: Esquema de Clases del Sistema (Turbine)

* **Orientado a Objetos (no Modelo ER):** Cada clase incluye tanto estado (atributos privados `-`) como comportamiento (métodos públicos `+` extraídos de los Casos de Uso).
* **Sin claves foráneas (FK):** Las relaciones entre clases se modelan mediante referencias de objetos, listas y multiplicidades UML.

---

## 1. Bloque de Usuarios, Accesos y Trabajo Diario
*(Extraído de CU-01, CU-02, CU-07, CU-13, CU-32)*

### Clase `Usuario`
Representa a los empleados internos de Turbine (Usuario, Administrador, Superadministrador).
* **Atributos:**
  * `- idUsuario : int`
  * `- nombre : String`
  * `- correoCorporativo : String`
  * `- passwordHash : String`
  * `- rol : RolUsuario` *(Enum: USUARIO, ADMIN, SUPERADMIN)*
  * `- puedeGestionarUsuarios : bool` *(Permiso del Administrador, CU-02/CU-13)*
  * `- activo : bool`
  * `- intentosFallidos : int`
  * `- bloqueadoHasta : DateTime` *(Bloqueo temporal tras varios intentos, CU-01)*
  * `- canalesPreferidos : List<CanalNotificacion>`
* **Métodos:**
  * `+ iniciarSesion(correo : String, pass : String) : Sesion` *(CU-01)*
  * `+ cerrarSesion() : void` *(CU-01)*
  * `+ recuperarPassword() : void` *(CU-01)*
  * `+ enviarInvitacion() : bool` *(CU-02)*
  * `+ reactivarCuenta() : void` *(CU-02)*
  * `+ desactivarCuenta(motivo : String) : void` *(CU-13)*
  * `+ invalidarSesionesActivas() : void` *(CU-13)*
  * `+ configurarCanales(canales : List<CanalNotificacion>) : void` *(CU-32)*

### Clase `Sesion`
Representa cada inicio de sesión activo de un usuario para permitir su gestión e invalidación centralizada.
* **Atributos:**
  * `- idSesion : String`
  * `- token : String`
  * `- fechaInicio : DateTime`
  * `- ultimaActividad : DateTime`
  * `- direccionIP : String`
  * `- dispositivo : String`
  * `- activa : bool`
* **Métodos:**
  * `+ esValida() : bool` *(CU-01)*
  * `+ invalidar() : void` *(CU-01, CU-13)*
  * `{static} + invalidarTodasDeUsuario(u : Usuario) : void` *(CU-13)*

### Clase `DelegacionTemporal`
Permite delegar los clientes de un usuario a un sustituto durante una ausencia.
* **Atributos:**
  * `- idDelegacion : int`
  * `- fechaInicio : Date`
  * `- fechaFin : Date`
  * `- activa : bool`
* **Métodos:**
  * `+ activarDelegacion() : void` *(CU-07)*
  * `+ cancelarDelegacion() : void` *(CU-07)*
  * `+ estaVigente(fecha : Date) : bool` *(CU-07, la delegación caduca sola)*

### Clase `Notificacion`
Avisos internos en la pantalla de trabajo diario.
* **Atributos:**
  * `- idNotificacion : int`
  * `- mensaje : String`
  * `- fechaCreacion : DateTime`
  * `- estado : EstadoNotificacion` *(Enum: NO_LEIDA, LEIDA, ATENDIDA)*
  * `- canal : CanalNotificacion` *(Enum: INTERNO, EMAIL, WHATSAPP)*
* **Métodos:**
  * `+ marcarComoLeida() : void` *(CU-32)*
  * `+ marcarComoAtendida() : void` *(CU-32)*
  * `+ enviar() : bool` *(CU-32)*

### Clase `AlertaCorporativa`
Reglas y condiciones del sistema para disparar alertas automáticas a grupos o roles de usuarios.
* **Atributos:**
  * `- idAlerta : int`
  * `- nombre : String`
  * `- condicionEvaluacion : String` *(ej. "Factura > 5000€" o "Impago > 15 días")*
  * `- canal : CanalNotificacion`
  * `- activa : bool`
* **Métodos:**
  * `+ evaluarCondicion(contexto : Object) : bool` *(CU-32 2b)*
  * `+ dispararAlerta(destinatarios : List<Usuario>) : void` *(CU-32 2b)*

---

## 2. Bloque de Configuración, Auditoría y Datos
*(Extraído de CU-03, CU-04, CU-19, CU-20, CU-24, CU-34)*

### Clase `ConfiguracionSistema`
Parámetros globales modificables por el Superadministrador.
* **Atributos:**
  * `- plazoPagoDefecto : int`
  * `- diasRecordatorio : List<int>` *(ej. días 3 y 5, CU-19)*
  * `- impuestoDefecto : double`
  * `- monedaBase : String`
  * `- umbralAprobacionInferior : double`
  * `- umbralAprobacionSuperior : double`
  * `- periodicidadCopias : int`
  * `- datosEmisor : String`
  * `- datosSoporte : String`
* **Métodos:**
  * `+ actualizarParametros() : bool` *(CU-03)*
  * `+ validarUmbrales() : bool` *(CU-03)*

### Clase `ConexionPasarelaCobro`
Gestión de integraciones con pasarelas de pago externas (Stripe, Revolut) mediante credenciales cifradas.
* **Atributos:**
  * `- idConexion : int`
  * `- proveedor : ProveedorPasarela` *(Enum: STRIPE, REVOLUT)*
  * `- apiKeyCifrada : String`
  * `- webhookSecretCifrado : String`
  * `- activa : bool`
  * `- entorno : String` *(Enum: SANDBOX, PRODUCCION)*
* **Métodos:**
  * `+ guardarCredencialesCifradas(apiKey : String, secret : String) : bool` *(CU-04)*
  * `+ probarConexion() : bool` *(CU-04)*
  * `+ obtenerClienteAPI() : Object` *(CU-04, CU-11)*

### Clase `PlantillaMensaje`
Modelado de plantillas dinámicas por idioma y canal para comunicaciones con clientes e informes.
* **Atributos:**
  * `- idPlantilla : int`
  * `- nombre : String`
  * `- tipoEvento : TipoEvento` *(Enum: EMISION_FACTURA, RECORDATORIO_IMPAGO, NOTIFICACION_GENERAL)*
  * `- idioma : String`
  * `- canal : CanalNotificacion`
  * `- asunto : String`
  * `- cuerpo : String`
  * `- variablesDisponibles : List<String>`
* **Métodos:**
  * `+ previsualizar(datosPrueba : Map<String, String>) : String` *(CU-03, CU-10)*
  * `+ enviarPrueba(destinatarioTest : String) : bool` *(CU-03)*
  * `+ renderizar(datos : Map<String, String>) : String` *(CU-10, CU-19)*

### Clase `CalendarioLaboral`
Definición del calendario de trabajo anual para el ajuste automático de fechas límite y recordatorios.
* **Atributos:**
  * `- anio : int`
  * `- region : String`
* **Métodos:**
  * `+ esLaborable(fecha : Date) : bool` *(CU-19 2b)*
  * `+ obtenerSiguienteDiaLaborable(fecha : Date) : Date` *(CU-03, CU-19 2b)*
  * `+ agregarFestivo(festivo : DiaFestivo) : void` *(CU-03)*

### Clase `DiaFestivo`
Fecha concreta marcada como no laborable dentro del calendario.
* **Atributos:**
  * `- fecha : Date`
  * `- descripcion : String`
  * `- esNacional : bool`

### Clase `TarifaRango`
Tabla de tramos de facturación anual para calcular automáticamente el rango y tarifa de cada cliente.
* **Atributos:**
  * `- idTarifaRango : int`
  * `- nombreRango : String`
  * `- facturacionMinima : double`
  * `- facturacionMaxima : double`
  * `- cuotaBaseAsociada : double`
* **Métodos:**
  * `+ calcularRangoCorrespondiente(facturacionAnual : double) : TarifaRango` *(CU-03, CU-05)*

### Clase `RegistroAuditoria`
Trazabilidad inmutable de todas las operaciones críticas del sistema (no tiene métodos de modificación).
* **Atributos:**
  * `- idRegistro : int`
  * `- fechaHora : DateTime`
  * `- tipoOperacion : String`
  * `- entidadAfectada : String`
  * `- valorAnterior : String`
  * `- valorNuevo : String`
* **Métodos:**
  * `{static} + registrarEvento(u : Usuario, tipoOperacion : String, entidad : String, valorAnterior : String, valorNuevo : String) : RegistroAuditoria` *(CU-03, CU-20)*
  * `{static} + consultar(usuario : Usuario, tipoOperacion : String, entidad : String, desde : Date, hasta : Date) : List<RegistroAuditoria>` *(CU-20)*

### Clase `PeriodoContable`
Control de cierre mensual o trimestral para evitar modificaciones en facturas declaradas.
* **Atributos:**
  * `- idPeriodo : String` *(ej. "2026-03" o "2026-Q3")*
  * `- tipo : TipoPeriodo` *(Enum: MENSUAL, TRIMESTRAL)*
  * `- fechaInicio : Date`
  * `- fechaFin : Date`
  * `- cerrado : bool`
  * `- fechaCierre : DateTime`
  * `- motivoReapertura : String`
* **Métodos:**
  * `+ obtenerAvisosPendientes() : List<String>` *(CU-34, borradores o cobros sin conciliar)*
  * `+ cerrarPeriodo(admin : Usuario) : bool` *(CU-34)*
  * `+ reabrirPeriodo(superadmin : Usuario, motivo : String) : bool` *(CU-34)*
  * `+ contieneFecha(fecha : Date) : bool` *(CU-16)*

### Clase `EjecucionProceso`
Registro de cada tarea automática lanzada por el Planificador (generación mensual, recordatorios, copias).
* **Atributos:**
  * `- idEjecucion : int`
  * `- tipoProceso : String`
  * `- fechaHoraInicio : DateTime`
  * `- fechaHoraFin : DateTime`
  * `- estadoSalud : String`
  * `- totalGeneradas : int`
  * `- totalOmitidas : int`
  * `- totalErrores : int`
  * `- detalleErrores : List<String>`
* **Métodos:**
  * `+ registrarResultado() : void` *(CU-19, CU-20 1a, CU-24, CU-29)*
    
### Clase `CopiaSeguridad`
Representa los backups periódicos de datos y movimientos del sistema.
* **Atributos:**
  * `- idCopia : int`
  * `- fechaCreacion : DateTime`
  * `- rutaArchivo : String`
  * `- tamanoBytes : long`
  * `- verificadaCompleta : bool`
* **Métodos:**
  * `+ generarCopia() : bool` *(CU-24)*
  * `+ verificarIntegridad() : bool` *(CU-24)*
  * `+ descargar() : File` *(CU-20 1b)*
  * `+ restaurar(superadmin : Usuario) : bool` *(CU-20 1b)*

---

## 3. Bloque de Clientes y Suscripciones
*(Extraído de CU-05, CU-06, CU-08, CU-10, CU-14, CU-15, CU-17, CU-18, CU-25, CU-26, CU-29, CU-35, CU-36)*

### Clase `Cliente`
Entidad principal sobre la que gira la facturación.
* **Atributos:**
  * `- idCliente : int`
  * `- razonSocial : String`
  * `- nif : String`
  * `- direccionFiscal : String`
  * `- pais : String`
  * `- sector : String`
  * `- etiquetas : List<String>`
  * `- rangoTarifa : String`
  * `- facturacionAnual : double`
  * `- estado : EstadoCliente` *(Enum: ACTIVO, BAJA)*
  * `- fechaBaja : Date`
  * `- eliminadoLogico : bool`
* **Métodos:**
  * `+ darDeAlta() : bool` *(CU-36)*
  * `+ reactivar() : void` *(CU-36, cliente que vuelve)*
  * `+ modificarDatosFiscales() : void` *(CU-17)*
  * `+ asignarResponsable(u : Usuario) : void` *(CU-08)*
  * `+ desasignarResponsable() : void` *(CU-08)*
  * `+ registrarFacturacionAnual(importe : double, fuente : String, fecha : Date) : void` *(CU-05)*
  * `+ calcularRango() : String` *(CU-05)*
  * `+ obtenerFacturasPendientes() : List<Factura>` *(CU-15)*
  * `+ calcularDeudaTotal() : double` *(CU-15)*
  * `+ darDeBaja(fechaEfectiva : Date) : void` *(CU-17)*
  * `+ borradoLogico(motivo : String) : void` *(CU-14)*

### Clase `ContactoCliente`
Persona de contacto dentro de la empresa cliente (separado para cumplir protección de datos CU-35).
* **Atributos:**
  * `- idContacto : int`
  * `- nombre : String`
  * `- email : String`
  * `- telefono : String`
  * `- funcion : String` *(ej. contacto de facturación, CU-10)*
  * `- idioma : String`
  * `- consienteWhatsApp : bool`
* **Métodos:**
  * `+ obtenerCanalEnvio() : CanalNotificacion` *(CU-10)*
  * `+ anonimizarDatos() : void` *(CU-35)*
  * `+ eliminarDatos() : void` *(CU-35)*

### Clase `ImportacionClientes`
Gestión de cargas masivas de clientes desde archivos CSV/Excel, con mapeo e inspección de errores por fila.
* **Atributos:**
  * `- idImportacion : int`
  * `- nombreArchivo : String`
  * `- fechaEjecucion : DateTime`
  * `- mapeoColumnas : Map<String, String>`
  * `- totalFilas : int`
  * `- filasExitosas : int`
  * `- filasConError : int`
  * `- erroresDetalle : List<String>`
* **Métodos:**
  * `+ cargarArchivo(archivo : File) : bool` *(CU-18)*
  * `+ definirMapeo(mapeo : Map<String, String>) : void` *(CU-18)*
  * `+ validarFilas() : bool` *(CU-18)*
  * `+ procesarImportacion() : List<Cliente>` *(CU-18)*
  * `+ obtenerResumen() : String` *(CU-18)*

### Clase `Suscripcion`
Contrato de facturación recurrente asociado a un cliente.
* **Atributos:**
  * `- idSuscripcion : int`
  * `- plan : String`
  * `- cuotaMensual : double`
  * `- diaVencimientoPactado : int`
  * `- fechaInicio : Date`
  * `- fechaFin : Date`
  * `- estado : EstadoSuscripcion` *(Enum: PROGRAMADA, ACTIVA, PAUSADA, CANCELADA)*
  * `- motivoPausa : String`
  * `- fechaInicioPausa : Date`
  * `- fechaReactivacionPrevista : Date`
* **Métodos:**
  * `+ activar() : void` *(CU-06)*
  * `+ calcularProrrateo(fechaAlta : Date) : double` *(CU-06)*
  * `+ pausar(fechaInicio : Date, fechaReactivacion : Date, motivo : String) : void` *(CU-25)*
  * `+ reactivar() : void` *(CU-25)*
  * `+ cancelar(fechaEfectiva : Date) : void` *(CU-17)*
  * `+ actualizarTarifa(nuevoRango : String, fechaEfecto : Date) : void` *(CU-05)*
  * `+ aplicarExcepcion(descuento : double, fechaFin : Date) : void` *(CU-26)*
  * `+ calcularImporteMensual(mes : Date) : double` *(CU-29)*
  * `+ generarFacturaMensual(mes : Date) : Factura` *(CU-29)*

### Clase `ExcepcionComercial`
Condición especial sujeta a aprobación (descuento temporal, mes gratis, precio pactado o exención de prorrateo).
* **Atributos:**
  * `- idExcepcion : int`
  * `- tipo : TipoExcepcion` *(Enum: DESCUENTO, MES_GRATIS, PRECIO_PACTADO, NO_PRORRATEAR)*
  * `- valor : double`
  * `- motivo : String`
  * `- fechaFin : Date`
  * `- vigente : bool`
* **Métodos:**
  * `+ solicitarAprobacion() : SolicitudAprobacion` *(CU-06 4a, CU-26)*
  * `+ aplicarEnCuota(importeBase : double) : double` *(CU-26, CU-29)*

---

## 4. Bloque de Facturación, Presupuestos y Aprobaciones
*(Extraído de CU-10, CU-11, CU-12, CU-14, CU-15, CU-16, CU-19, CU-21, CU-23, CU-27, CU-28, CU-29, CU-30)*

### Clase `ProductoCatalogo`
Catálogo maestro de productos y servicios reutilizables para confeccionar presupuestos y facturas.
* **Atributos:**
  * `- idProducto : int`
  * `- codigoSKU : String`
  * `- nombre : String`
  * `- descripcion : String`
  * `- precioUnitarioBase : double`
  * `- porcentajeImpuestoDefecto : double`
  * `- activo : bool`
* **Métodos:**
  * `+ crearLineaFactura(cantidad : int, descuento : double) : LineaFactura` *(CU-28)*
  * `+ actualizarPrecio(nuevoPrecio : double) : void` *(CU-28)*

### Clase `Presupuesto`
Propuesta comercial previa que puede convertirse en borrador de factura.
* **Atributos:**
  * `- idPresupuesto : String`
  * `- fechaCreacion : Date`
  * `- estado : EstadoPresupuesto` *(Enum: PENDIENTE, ENVIADO, ACEPTADO, RECHAZADO)*
  * `- documentoEvidencia : String`
* **Métodos:**
  * `+ calcularTotal() : double` *(CU-30)*
  * `+ enviar() : bool` *(CU-30)*
  * `+ aceptar(evidencia : String) : void` *(CU-30)*
  * `+ rechazar(evidencia : String) : void` *(CU-30)*
  * `+ convertirEnFactura() : Factura` *(CU-30)*

### Clase `Factura`
Documento contable (puntual, recurrente o rectificativa).
* **Atributos:**
  * `- idFactura : int` *(un borrador aún no tiene número)*
  * `- numeroFactura : String` *(se asigna al emitir, CU-28)*
  * `- serie : String`
  * `- tipo : TipoFactura` *(Enum: PUNTUAL, RECURRENTE, RECTIFICATIVA)*
  * `- fechaEmision : Date`
  * `- fechaVencimiento : Date`
  * `- mesFacturado : String` *(solo recurrentes, evita duplicados en CU-29)*
  * `- moneda : String`
  * `- centroCoste : String`
  * `- baseImponible : double`
  * `- totalImpuestos : double`
  * `- importeTotal : double`
  * `- saldoPendiente : double`
  * `- estado : EstadoFactura` *(Enum: BORRADOR, PENDIENTE_APROBACION, EMITIDA, PAGADA, VENCIDA, RECTIFICADA, ANULADA, INCOBRABLE)*
  * `- enDisputa : bool`
  * `- rutaPdf : String`
* **Métodos:**
  * `+ calcularTotales() : void` *(CU-28)*
  * `+ agregarLineaDesdeCatalogo(p : ProductoCatalogo, cantidad : int, dto : double) : void` *(CU-28)*
  * `+ modificarBorrador() : bool` *(CU-16)*
  * `+ eliminarBorrador() : bool` *(CU-14, solo si nunca se emitió)*
  * `+ emitir() : bool` *(CU-06, CU-28, CU-29)*
  * `+ generarPdf() : String` *(CU-16, CU-28)*
  * `+ obtenerPdf() : String` *(CU-23, devuelve el PDF guardado, no lo regenera)*
  * `+ enviarAlCliente() : bool` *(CU-10)*
  * `+ marcarComoPagada(c : Cobro) : void` *(CU-11, CU-12)*
  * `+ calcularDiasRetraso() : int` *(CU-15)*
  * `+ requiereRecordatorio(fecha : Date) : bool` *(CU-19)*
  * `+ crearRectificativa(motivo : String) : Factura` *(CU-21)*
  * `+ anular(motivo : String) : bool` *(CU-21)*
  * `+ declararIncobrable(motivo : String) : void` *(CU-21)*

### Clase `LineaFactura`
Detalle de cada concepto facturado. Cada línea pertenece **o** a una `Factura` **o** a un `Presupuesto` (composición con restricción `{xor}`); al convertir un presupuesto o crear una rectificativa, las líneas se copian.
* **Atributos:**
  * `- numeroLinea : int`
  * `- concepto : String`
  * `- cantidad : int`
  * `- precioUnitario : double`
  * `- porcentajeDescuento : double`
  * `- porcentajeImpuesto : double`
* **Métodos:**
  * `+ calcularSubtotal() : double` *(CU-28)*

### Clase `EnvioFactura`
Historial de cada intento de entrega de una factura al cliente por email o WhatsApp.
* **Atributos:**
  * `- idEnvio : int`
  * `- fechaHoraEnvio : DateTime`
  * `- canalUtilizado : CanalNotificacion`
  * `- destinatario : String`
  * `- exitoso : bool`
  * `- detalleErrorRebote : String`
* **Métodos:**
  * `+ enviar(plantilla : PlantillaMensaje) : bool` *(CU-10)*
  * `+ reintentarEnvio() : bool` *(CU-10 4a)*

### Clase `SolicitudAprobacion`
Bandeja de control para operaciones que superan umbrales o requieren validación de un Administrador.
* **Atributos:**
  * `- idSolicitud : int`
  * `- tipoOperacion : TipoAprobacion` *(Enum: IMPORTE_MANUAL, DESCUENTO, EXCEPCION_COMERCIAL, NO_PRORRATEO, VENCIMIENTO_ESPECIAL, FACTURA_IMPORTE_ALTO, CAMBIO_RANGO, RECTIFICATIVA, INCOBRABLE)*
  * `- importeAfectado : double`
  * `- motivoSolicitud : String`
  * `- comentarioResolucion : String`
  * `- fechaSolicitud : DateTime`
  * `- fechaResolucion : DateTime`
  * `- estado : EstadoAprobacion` *(Enum: PENDIENTE, APROBADA, RECHAZADA)*
* **Métodos:**
  * `+ requiereSuperadmin() : bool` *(CU-27, supera el umbral superior)*
  * `+ aprobar(aprobador : Usuario, comentario : String) : bool` *(CU-27, el aprobador no puede ser el solicitante)*
  * `+ rechazar(aprobador : Usuario, comentario : String) : void` *(CU-27)*
  * `+ reasignar(suplente : Usuario) : void` *(CU-27)*

---

## 5. Bloque de Cobros, Impagos e Incidencias
*(Extraído de CU-11, CU-12, CU-19, CU-31)*

### Clase `Cobro`
Registro de un pago recibido (automático por Stripe/Revolut o registrado manualmente).
* **Atributos:**
  * `- idCobro : int`
  * `- referenciaExterna : String` *(evita aplicar dos veces el mismo evento, CU-11)*
  * `- fechaCobro : Date`
  * `- importe : double`
  * `- origen : OrigenCobro` *(Enum: STRIPE, REVOLUT, MANUAL)*
  * `- justificanteAdjunto : String`
  * `- conciliado : bool`
* **Métodos:**
  * `+ procesarCobroAutomatico() : bool` *(CU-11)*
  * `+ conciliarManualmente(f : Factura, justificante : String) : bool` *(CU-12)*
  * `+ deshacerConciliacion(motivo : String) : void` *(CU-12)*

### Clase `RecordatorioImpago`
Avisos amistosos enviados automáticamente cuando una factura está próxima a vencer o vencida.
* **Atributos:**
  * `- idRecordatorio : int`
  * `- fechaEnvio : DateTime`
  * `- numeroIntento : int`
  * `- canal : CanalNotificacion`
  * `- enviadoConExito : bool`
* **Métodos:**
  * `+ enviarRecordatorio(plantilla : PlantillaMensaje) : bool` *(CU-19)*

### Clase `Incidencia`
Ticket interno o disputa abierta sobre un cliente o una factura (detiene los recordatorios).
* **Atributos:**
  * `- idIncidencia : int`
  * `- descripcion : String`
  * `- prioridad : Prioridad` *(Enum: BAJA, MEDIA, ALTA)*
  * `- fechaApertura : DateTime`
  * `- esDisputaFactura : bool`
  * `- fechaRevision : Date`
  * `- resolucion : String`
  * `- resuelta : bool`
* **Métodos:**
  * `+ abrirIncidencia() : void` *(CU-31)*
  * `+ pausarSeguimientoImpago(motivo : String, fechaRevision : Date) : void` *(CU-19, CU-31)*
  * `+ reanudarSeguimientoImpago() : void` *(CU-31, si la factura sigue pendiente al cerrar)*
  * `+ agregarComentario(u : Usuario, texto : String) : void` *(CU-31, CU-32)*
  * `+ resolverYCerrar(resolucion : String) : void` *(CU-31)*

---

## 6. Bloque de Informes, Paquetes y Operaciones Masivas
*(Extraído de CU-09, CU-33)*

### Clase `Informe`
Generación y exportación de reportes analíticos de facturación, impagos o IVA.
* **Atributos:**
  * `- idInforme : int`
  * `- tipoInforme : TipoInforme` *(Enum: VENTAS, IMPAGOS, IVA, FACTURACION)*
  * `- fechaDesde : Date`
  * `- fechaHasta : Date`
  * `- formatoExportacion : FormatoArchivo` *(Enum: CSV, PDF)*
* **Métodos:**
  * `+ generarReporte() : File` *(CU-09)*
  * `+ exportarCSV() : File` *(CU-09)*
  * `+ exportarPDF() : File` *(CU-09)*

### Clase `PaqueteGestoria`
Empaquetado periódico (mensual o trimestral) de facturas y documentos contables en un archivo comprimido listo para gestoría.
* **Atributos:**
  * `- idPaquete : int`
  * `- periodo : String` *(ej. "2026-Q1")*
  * `- tipoPeriodo : TipoPeriodo` *(Enum: MENSUAL, TRIMESTRAL)*
  * `- fechaGeneracion : DateTime`
  * `- rutaZip : String`
  * `- totalFacturasIncluidas : int`
* **Métodos:**
  * `+ compilarPaquete(p : PeriodoContable) : File` *(CU-09, CU-33)*
  * `+ descargarZip() : File` *(CU-09)*

### Clase `AccionMasiva`
Orquestador y registro de ejecución para tareas ejecutadas en bloque sobre múltiples elementos.
* **Atributos:**
  * `- idAccionMasiva : int`
  * `- tipoAccion : TipoAccionMasiva` *(Enum: EMISION_FACTURAS, ENVIAR_NOTIFICACIONES, CAMBIO_ESTADO, DESCARGA_GESTORIA)*
  * `- fechaEjecucion : DateTime`
  * `- totalProcesados : int`
  * `- totalExitosos : int`
  * `- totalFallidos : int`
  * `- resultadosPorElemento : Map<String, bool>`
  * `- erroresDetalle : Map<String, String>`
* **Métodos:**
  * `+ ejecutar(elementos : List<Object>) : void` *(CU-33)*
  * `+ obtenerInformeResultados() : String` *(CU-33)*

---

## 7. Resumen de Relaciones Principales (UML)
1. **Composición (`*--`, rombo negro - ciclo de vida dependiente):**
   * `Usuario "1" *-- "0..*" Sesion : -sesionesActivas` *(Al desactivar o invalidar las sesiones del usuario, CU-13).*
   * `CalendarioLaboral "1" *-- "0..*" DiaFestivo : -festivos` *(El calendario contiene los días festivos).*
   * `Factura "0..1" *-- "1..*" LineaFactura` *(Si se borra un borrador de factura, desaparecen sus líneas).*
   * `Presupuesto "0..1" *-- "1..*" LineaFactura` *(`{xor}` con la anterior: cada línea pertenece a uno solo de los dos).*
   * `Cliente "1" *-- "1..*" ContactoCliente`

2. **Agregación / Asociación (`o--` / `-->` - ciclo de vida independiente):**
   * `ConfiguracionSistema "1" --> "0..*" ConexionPasarelaCobro : -pasarelasConfiguradas` *(CU-04).*
   * `ConfiguracionSistema "1" --> "0..*" PlantillaMensaje : -plantillasNotificacion` *(CU-03).*
   * `ConfiguracionSistema "1" --> "1" CalendarioLaboral : -calendarioVigente` *(CU-03).*
   * `AlertaCorporativa "0..*" --> "0..*" Usuario : -destinatarios` *(CU-32 2b).*
   * `Usuario "0..1" --> "0..*" Cliente : -clientesAsignados` *(Un empleado gestiona varios clientes; un cliente puede quedar sin responsable, CU-08/CU-13).*
   * `Usuario "0..*" --> "0..*" Cliente : -favoritos` *(CU-22).*
   * `Usuario "1" --> "0..*" Notificacion : -notificaciones` *(CU-32).*
   * `DelegacionTemporal "0..*" --> "1" Usuario : -titular`
   * `DelegacionTemporal "0..*" --> "1" Usuario : -sustituto`
   * `DelegacionTemporal "0..*" --> "0..*" Cliente : -clientesDelegados` *(Si está vacía, se delegan todos, CU-07).*
   * `ImportacionClientes "0..*" --> "1" Usuario : -ejecutadoPor` *(CU-18).*
   * `Cliente "1" --> "0..*" Suscripcion : -suscripciones`
   * `Cliente "1" --> "0..*" Factura : -facturas`
   * `Cliente "1" --> "0..*" Presupuesto : -presupuestos` *(CU-30).*
   * `LineaFactura "0..*" --> "0..1" ProductoCatalogo : -productoOrigen` *(CU-28).*
   * `Suscripcion "0..1" --> "0..*" Factura : -facturasGeneradas` *(CU-06, CU-29).*
   * `Presupuesto "0..1" --> "0..1" Factura : -facturaGenerada` *(CU-30).*
   * `Factura "0..1" --> "0..*" Cobro : -cobros` *(Un cobro sin coincidencia clara queda sin factura hasta conciliarse, CU-11/CU-12).*
   * `Factura "1" --> "0..*" RecordatorioImpago : -recordatorios`
   * `Factura "0..1" <-- "0..*" Factura : -facturaOriginal` *(Asociación reflexiva: una original puede tener varias rectificativas, CU-21).*
   * `EnvioFactura "0..*" --> "1" PlantillaMensaje : -plantillaUsada` *(CU-10).*
   * `RecordatorioImpago "0..*" --> "1" PlantillaMensaje : -plantillaUsada` *(CU-19).*
   * `SolicitudAprobacion "0..*" --> "1" Usuario : -solicitante`
   * `SolicitudAprobacion "0..*" --> "0..1" Usuario : -aprobador` *(CU-27).*
   * `Incidencia "0..*" --> "0..1" Cliente` y `Incidencia "0..*" --> "0..1" Factura` *(`{xor}`: sobre un cliente o sobre una factura, CU-31).*
   * `Incidencia "0..*" --> "1" Usuario : -responsable` *(CU-31).*
   * `RegistroAuditoria "0..*" --> "0..1" Usuario : -autor` *(0..1 porque los procesos del Planificador no tienen usuario).*
   * `PaqueteGestoria "0..*" --> "1" PeriodoContable : -periodoAsociado` *(CU-09, CU-33).*
   * `AccionMasiva "0..*" --> "1" Usuario : -ejecutadoPor` *(CU-33).*

3. **Dependencia (`..>`, línea discontinua - relación de uso):**
   * `Factura ..> PeriodoContable : <<uses>>` *(Comprueba si el periodo está abierto antes de editar, CU-16/CU-34).*
   * `Factura ..> ConfiguracionSistema : <<uses>>` *(Lee los impuestos y series vigentes al emitirse, CU-03/CU-28).*
   * `RecordatorioImpago ..> CalendarioLaboral : <<uses>>` *(Salta festivos para posponer al siguiente día hábil, CU-19 2b).*
   * `SolicitudAprobacion ..> ConfiguracionSistema : <<uses>>` *(Lee los umbrales de aprobación, CU-27).*
   * `PaqueteGestoria ..> Factura : <<uses>>` *(Recopila e incluye los PDF/CSV de facturas del periodo).*
