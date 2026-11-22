# Diagramas

Código [PlantUML](https://plantuml.com/es/) de los diagramas UML del proyecto, junto con sus imágenes. Cualquier información adicional debe quedar reflejada en este readme.

---

## Contenido

| Carpeta | Diagrama | Ficheros principales |
|---|---|---|
| `diagrama_CU` | Casos de uso: el diagrama completo y 5 subdiagramas por bloques | `Diagrama_completo_sistema.puml`, `imagenes/` |
| `diagramas_clases` | Clases, organizado en 6 bloques | `diagrama_clases` (completo), `bloques/` (un diagrama por bloque y otro de enumeraciones, para verlos mejor), `borrador_clases.md` (explicación de cada clase) |
| `diagrama_entidad_relacion` | Entidad-relación físico de la base de datos | `diagrama_ER_turbine.puml`, `documentacion_er.md` (notación y decisiones de diseño) |
| `diagramas_secuencia` | Secuencia: un diagrama maestro y 6 bloques | `DiagramaCompletoBorrador`, `DiagrmasPorBloques/` |
| `diagrama_de_componentes` | Componentes del sistema (versión simplificada) | `diagrama_componentes.puml`, `diagrama_de_componentes.svg` |
| `diagramas_de_ejemplo` | Ejemplos hechos en clase durante las prácticas | — |

## Relación entre los diagramas

- **Clases → ER:** el diagrama ER sale del de clases. Lo que no se guarda en la base de datos y las tablas que se fusionan están explicados en `documentacion_er.md`.
- **Clases → secuencia:** los diagramas de secuencia solo usan clases y métodos del diagrama de clases. Los atributos privados se leen con su getter (por ejemplo, `getMonedaBase()`).
- **Clases → componentes:** cada bloque del diagrama de clases es un componente del backend.
- **Secuencia → componentes:** el participante `:Controlador / Interfaz` de los diagramas de secuencia equivale a la Interfaz web más la API REST.
- **Maestro de secuencia:** es la unión de los 6 bloques. Si se cambia un bloque, hay que cambiar también el maestro y volver a generar la imagen del bloque.

---

## Reparto del trabajo del diagrama de casos de uso

La numeración de los casos es la del primer borrador. La actual está en [Casos de Uso.md](../Documentacion/casosDeUso/Casos%20de%20Uso.md).

| Persona | Área | Casos de uso |
|---|---|---|
| Francis | Usuarios y acceso | Login, registro de usuarios internos, asignación de usuarios a administradores, asignación de clientes y borrado de cuentas |
| Marcondes | Administración y configuración | Configuración global, cuenta de recepción de pagos (Stripe/Revolut), revisión anual del rango, facturación recurrente y métricas globales |
| Esperanza | Pagos e impagos | Consulta de pagos e impagos, control de pagos por notificación, registro manual de pagos, facturas pendientes y flujo de impagos |
| Juan | Clientes y facturas | Modificación de facturas, alta, baja y edición de clientes, importación masiva, anulación de facturas y búsqueda de clientes |
| Álvaro | Documentos, métricas y sistema | Métricas por equipos, envío de facturas, borrado lógico, auditoría y copias de seguridad, y descarga del PDF |
