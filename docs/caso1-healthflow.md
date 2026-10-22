# Caso 1: HealthFlow - Sistema de Gestión Hospitalaria Integral

**Autor:** José Fernando Díaz del Cid  
**Rama:** `feature/modelo-hospital`  
**Referencia:** *Guía de Práctica UML y GitHub - UTEC*, página 1, caso 1; página 3, lineamientos y rúbrica.

## 1. Alcance y artefactos

Modelo de análisis del ciclo clínico y del registro de intervenciones. Incluye 13 clases, tres generalizaciones y diez asociaciones, de las cuales cuatro son composiciones y una es agregación compartida. Se entrega el [proyecto editable de StarUML](../models/caso1-healthflow.mdj) y su [exportación PNG](../models/caso1-healthflow.png).

El diagrama completo reúne todos los atributos, operaciones y extremos de relación. Los nombres no llevan tildes para facilitar su uso como identificadores; la documentación está en español. No incluye implementación de software ni los casos 2 y 3.

## 2. Convenciones y decisiones de análisis

- Todos los atributos son privados (`-`) y todas las operaciones públicas (`+`). `Empleado` y `Empleado.describirRol()` son abstractos; cada subclase redefine esta operación.
- `String`, `Decimal`, `Boolean`, `Date` y `DateTime` son tipos de valor. `void` expresa ausencia de valor de retorno. Los importes y cantidades usan `Decimal`, sin errores de redondeo binario.
- Las referencias a clases en parámetros son dependencias de uso; no generan por sí solas propiedad ni asociaciones permanentes. Las relaciones persistentes se muestran mediante extremos UML, evitando duplicarlas como atributos `idPaciente` dentro del expediente o listas dentro de los propietarios.
- Las asociaciones dejan la navegabilidad sin especificar porque el caso requiere un modelo conceptual, no una estrategia de acceso en código. Los nombres de rol identifican los extremos.
- Los datos adicionales (fechas, identificadores, cantidad y lote) y las restricciones siguientes son decisiones de diseño explícitas, no requisitos textuales adicionales de la guía.

### Multiplicidades y ciclo de vida

Se interpreta "múltiples consultas y recetas" como capacidad de almacenar muchas entradas durante la vida del expediente. Se adopta `0..*` porque un expediente recién abierto aún no tiene atenciones, y una atención puede no generar receta. Cada entrada pertenece siempre a exactamente un expediente. Esta decisión evita obligar a inventar una receta para poder registrar un paciente. Si la cátedra define un expediente ya atendido como único estado válido, el límite inferior de consultas puede ajustarse a `1`; no se presupone esa regla en el enunciado.

Paciente y expediente se crean de forma conjunta y atómica: una vez registrado el paciente en el sistema hay exactamente un expediente, siempre del mismo titular. Que el expediente sea vitalicio no implica borrarlo ante alta o fallecimiento; ambos se conservan como historia clínica. La composición expresa propiedad exclusiva y dependencia conceptual. No se traduce en una política automática de eliminación física.

## 3. Diccionario de clases

### 3.1. `Empleado` (abstracta)

Abstracción de los datos y el comportamiento común de todo empleado del hospital.

| Atributo privado | Tipo | Significado |
|---|---|---|
| `-idEmpleado` | `String` | Identificador único del empleado. |
| `-nombre` | `String` | Nombre completo. |
| `-salario` | `Decimal` | Remuneración no negativa. |


| Operación pública | Responsabilidad |
|---|---|
| `+calcularSalario(): Decimal` | Consulta la remuneración del empleado. |
| `+describirRol(): String` | Operación abstracta que cada especialidad redefine. Es abstracta. |

### 3.2. `Enfermero`

Empleado responsable de los cuidados de enfermería; hereda identificación, nombre y salario.

**Hereda de `Empleado`:** `idEmpleado: String`, `nombre: String`, `salario: Decimal` y `calcularSalario(): Decimal`. Redefine `describirRol(): String`.

| Atributo privado | Tipo | Significado |
|---|---|---|
| `-numeroRegistro` | `String` | Registro profesional de enfermería. |
| `-turno` | `String` | Turno asignado. |


| Operación pública | Responsabilidad |
|---|---|
| `+registrarCuidado(paciente: Paciente, nota: String): void` | Registra un cuidado para un paciente. |
| `+describirRol(): String` | Devuelve el rol de enfermería. |

### 3.3. `Administrativo`

Empleado que realiza tareas administrativas; conserva los atributos heredados de Empleado.

**Hereda de `Empleado`:** `idEmpleado: String`, `nombre: String`, `salario: Decimal` y `calcularSalario(): Decimal`. Redefine `describirRol(): String`.

| Atributo privado | Tipo | Significado |
|---|---|---|
| `-area` | `String` | Área administrativa. |
| `-cargo` | `String` | Cargo desempeñado. |


| Operación pública | Responsabilidad |
|---|---|
| `+registrarIngreso(paciente: Paciente): void` | Registra el ingreso administrativo. |
| `+describirRol(): String` | Devuelve el rol administrativo. |

### 3.4. `Medico`

Empleado con especialidad que emite expedientes y participa en intervenciones.

**Hereda de `Empleado`:** `idEmpleado: String`, `nombre: String`, `salario: Decimal` y `calcularSalario(): Decimal`. Redefine `describirRol(): String`.

| Atributo privado | Tipo | Significado |
|---|---|---|
| `-numeroJunta` | `String` | Identificador profesional del médico. |
| `-especialidad` | `String` | Especialidad médica requerida por el enunciado. |


| Operación pública | Responsabilidad |
|---|---|
| `+emitirExpediente(paciente: Paciente): ExpedienteClinico` | Emite el expediente único del paciente. |
| `+describirRol(): String` | Devuelve el rol y la especialidad médica. |

### 3.5. `Paciente`

Persona atendida y raíz de propiedad exclusiva del expediente clínico vitalicio.

| Atributo privado | Tipo | Significado |
|---|---|---|
| `-idPaciente` | `String` | Identificador único del paciente. |
| `-nombre` | `String` | Nombre completo. |
| `-fechaNacimiento` | `Date` | Fecha de nacimiento. |
| `-telefono` | `String` | Contacto del paciente. |


| Operación pública | Responsabilidad |
|---|---|
| `+consultarExpediente(): ExpedienteClinico` | Recupera su expediente único. |
| `+actualizarContacto(telefono: String): void` | Actualiza el contacto. |

### 3.6. `ExpedienteClinico`

Registro clínico exclusivo de un paciente, emitido por un médico; contiene consultas y recetas.

| Atributo privado | Tipo | Significado |
|---|---|---|
| `-numeroExpediente` | `String` | Número único e inmutable. |
| `-fechaApertura` | `DateTime` | Momento de emisión. |
| `-antecedentes` | `String` | Antecedentes clínicos consolidados. |


| Operación pública | Responsabilidad |
|---|---|
| `+agregarConsulta(consulta: Consulta): void` | Incorpora una consulta exclusiva de este expediente. |
| `+agregarReceta(receta: Receta): void` | Incorpora una receta exclusiva de este expediente. |

### 3.7. `Consulta`

Entrada clínica con los hallazgos de una atención; pertenece a un único expediente.

| Atributo privado | Tipo | Significado |
|---|---|---|
| `-idConsulta` | `String` | Identificador único de la entrada. |
| `-fechaHora` | `DateTime` | Momento de la atención. |
| `-motivo` | `String` | Motivo de consulta. |
| `-diagnostico` | `String` | Conclusión clínica registrada. |


| Operación pública | Responsabilidad |
|---|---|
| `+registrarDiagnostico(texto: String): void` | Completa el diagnóstico. |
| `+consultarResumen(): String` | Devuelve un resumen de la atención. |

### 3.8. `Receta`

Documento de prescripción archivado exclusivamente en el expediente del paciente.

| Atributo privado | Tipo | Significado |
|---|---|---|
| `-idReceta` | `String` | Identificador único de la receta. |
| `-fechaEmision` | `DateTime` | Momento de emisión. |
| `-prescripcion` | `String` | Medicamentos, dosis, vía y frecuencia. |
| `-indicaciones` | `String` | Instrucciones y duración del tratamiento. |


| Operación pública | Responsabilidad |
|---|---|
| `+registrarIndicaciones(texto: String): void` | Completa las instrucciones de tratamiento. |
| `+consultarPrescripcion(): String` | Recupera el contenido de la receta. |

### 3.9. `EquipoMedico`

Recurso inventariable trasladable, como un monitor o bisturí; existe independientemente de la sala.

| Atributo privado | Tipo | Significado |
|---|---|---|
| `-idEquipo` | `String` | Identificador de inventario. |
| `-nombre` | `String` | Nombre o descripción del equipo. |
| `-estado` | `String` | Disponible, enUso o mantenimiento. |


| Operación pública | Responsabilidad |
|---|---|
| `+trasladarA(destino: SalaQuirurgica): void` | Cambia su sala sin destruir el equipo. |
| `+registrarMantenimiento(): void` | Actualiza su disponibilidad para mantenimiento. |

### 3.10. `SalaQuirurgica`

Espacio quirúrgico que agrupa equipos y aloja intervenciones en diferentes momentos.

| Atributo privado | Tipo | Significado |
|---|---|---|
| `-idSala` | `String` | Identificador único de la sala. |
| `-nombre` | `String` | Nombre de la sala. |
| `-ubicacion` | `String` | Ubicación física. |
| `-habilitada` | `Boolean` | Indica si admite programación. |


| Operación pública | Responsabilidad |
|---|---|
| `+asignarEquipo(equipo: EquipoMedico): void` | Asigna o reasigna un equipo. |
| `+estaDisponible(inicio: DateTime, fin: DateTime): Boolean` | Comprueba habilitación y ausencia de solapamientos. |

### 3.11. `Intervencion`

Registro de un procedimiento que vincula paciente, médicos, sala y consumos de insumos.

| Atributo privado | Tipo | Significado |
|---|---|---|
| `-idIntervencion` | `String` | Identificador único del procedimiento. |
| `-inicio` | `DateTime` | Inicio programado o real. |
| `-fin` | `DateTime` | Fin programado o real. |
| `-procedimiento` | `String` | Descripción del procedimiento. |
| `-estado` | `String` | Programada, enCurso, finalizada o cancelada. |


| Operación pública | Responsabilidad |
|---|---|
| `+registrarConsumo(detalle: ConsumoInsumo): void` | Valida e incorpora un detalle y descuenta existencias de forma atómica. |
| `+finalizar(): void` | Cierra la intervención con datos clínicos completos. |

### 3.12. `ConsumoInsumo`

Detalle exclusivo de una intervención que cuantifica un insumo utilizado y conserva su lote.

| Atributo privado | Tipo | Significado |
|---|---|---|
| `-idConsumo` | `String` | Identificador único del detalle. |
| `-cantidad` | `Decimal` | Cantidad consumida, estrictamente positiva. |
| `-lote` | `String` | Lote de procedencia para trazabilidad. |
| `-fechaHora` | `DateTime` | Momento del consumo. |


| Operación pública | Responsabilidad |
|---|---|
| `+validarCantidad(): Boolean` | Comprueba que la cantidad es positiva y respeta la unidad del insumo. |
| `+obtenerUnidad(): String` | Consulta la unidad del insumo asociado. |

### 3.13. `Insumo`

Definición inventariable del material consumible, independiente de las intervenciones.

| Atributo privado | Tipo | Significado |
|---|---|---|
| `-idInsumo` | `String` | Identificador del material. |
| `-nombre` | `String` | Descripción del insumo. |
| `-unidadMedida` | `String` | Unidad con que se mide stock y consumo. |
| `-stock` | `Decimal` | Existencia disponible, nunca negativa. |


| Operación pública | Responsabilidad |
|---|---|
| `+descontarStock(cantidad: Decimal): void` | Resta una cantidad positiva si existe disponibilidad. |
| `+reponer(cantidad: Decimal): void` | Añade existencias con cantidad positiva. |

## 4. Diccionario de relaciones y multiplicidades

En cada fila, la multiplicidad se escribe junto al nombre de su extremo. Por ejemplo, `Medico [1] - ExpedienteClinico [0..*]` significa que cada expediente tiene un médico emisor y que cada médico puede emitir cero o muchos expedientes.

### 4.1. Herencia (generalización)

`Medico`, `Enfermero` y `Administrativo` especializan a `Empleado`. Cada instancia de estas subclases es un empleado, hereda sus datos comunes y concreta un rol. El triángulo vacío apunta a `Empleado`. No se usa asociación para esta relación "es un" ni se repiten los atributos heredados. La generalización no lleva multiplicidades. No se impone que los subtipos sean disjuntos o completos: la guía no define esa restricción.

### 4.2. Relaciones con extremos

| Relación | Tipo | Roles | Justificación y lectura |
|---|---|---|---|
| `Paciente [1] - ExpedienteClinico [1]` (posee) | Composición | `titular` / `expediente` | El expediente carece de identidad clínica separable de su paciente y no se comparte ni transfiere. El rombo negro se sitúa en Paciente. |
| `Medico [1] - ExpedienteClinico [0..*]` (emite) | Asociación | `emisor` / `emitidos` | Un médico puede no haber emitido expedientes o haber emitido muchos; cada expediente registra exactamente un emisor inicial. El médico existe independientemente del expediente. |
| `ExpedienteClinico [1] - Consulta [0..*]` (contiene) | Composición | `expediente` / `consultas` | Cada consulta queda archivada en exactamente un expediente y no se comparte. Un expediente recién abierto admite cero entradas; después puede acumular muchas. |
| `ExpedienteClinico [1] - Receta [0..*]` (archiva) | Composición | `expediente` / `recetas` | La receta se modela como documento clínico archivado, exclusivo de un expediente. Puede haber cero recetas cuando no se prescribe medicación. |
| `Medico [1..*] - Intervencion [0..*]` (participa) | Asociación | `medicos` / `intervenciones` | Una intervención registrada requiere al menos un médico; cada médico participa en cero o muchas. La relación puede ser muchos a muchos y no implica propiedad ni destrucción. |
| `Paciente [1] - Intervencion [0..*]` (recibe) | Asociación | `paciente` / `intervenciones` | Cada intervención corresponde a un único paciente; un paciente puede no ser intervenido o acumular varias intervenciones. |
| `SalaQuirurgica [1] - Intervencion [0..*]` (aloja) | Asociación | `sala` / `intervenciones` | Cada intervención tiene una sala; una sala aloja cero o varias intervenciones a lo largo del tiempo, nunca simultáneas según la restricción temporal. |
| `SalaQuirurgica [0..1] - EquipoMedico [0..*]` (agrupa) | Agregación compartida | `salaActual` / `equipos` | El rombo blanco está en SalaQuirurgica: agrupa equipos que conservan identidad, pueden quedar sin sala o trasladarse. El cierre de una sala no elimina equipos. |
| `Intervencion [1] - ConsumoInsumo [0..*]` (registra) | Composición | `intervencion` / `consumos` | Cada detalle solo tiene sentido como registro de esa intervención. Una intervención programada o cancelada puede no tener consumos; si finaliza debe registrar al menos uno. |
| `ConsumoInsumo [0..*] - Insumo [1]` (corresponde a) | Asociación | `consumos` / `insumo` | Cada detalle referencia un insumo; el catálogo del insumo se comparte por muchos detalles y existe independientemente. Se consume cantidad de stock, no la entidad del catálogo. |

## 5. Por qué composición, agregación o asociación

### 5.1. Paciente y expediente: composición

El enunciado afirma dependencia vitalicia del expediente respecto del paciente. El paciente es el todo y el expediente la parte: rombo negro en `Paciente`, multiplicidad `1` en ambos extremos y propiedad exclusiva. No se usa agregación porque permitir que el expediente se desligue y se reasigne a otra persona contradice esa dependencia. El alta o fallecimiento no son destrucción del objeto paciente ni autorizan borrar datos clínicos.

### 5.2. Expediente, consultas y recetas: composición adoptada

Se modelan las consultas y recetas como entradas documentales del expediente, con identidad local al historial y sin reasignación a otro paciente. Por eso el rombo negro está en `ExpedienteClinico`; cada entrada tiene un propietario `1`. El enunciado dice que el expediente las contiene, pero no fija su ciclo de vida con la misma precisión que en paciente-expediente: la propiedad documental exclusiva es una decisión explícita del modelo. Una copia impresa de la receta no es una segunda entidad compartida. Si una futura integración necesita documentos clínicos autónomos entre instituciones, habría que reevaluar esta composición en vez de trasladar el mismo objeto.

### 5.3. Sala y equipo: agregación compartida

La sala agrupa los equipos actualmente asignados, por eso el rombo blanco se coloca en `SalaQuirurgica`. El equipo puede estar en ninguna sala (`0..1` en el extremo sala), por ejemplo durante mantenimiento, o en exactamente una; una sala admite `0..*` equipos. La agregación compartida no obliga a que un equipo pertenezca simultáneamente a varias salas. Se descarta composición porque el traslado y la conservación de identidad son requisitos expresos. Una asociación simple también permitiría describir el inventario; se escoge agregación para expresar el vínculo de agrupación débil que pide el ejercicio y se precisa su semántica en estas reglas.

### 5.4. Intervención y consumo: composición; consumo e insumo: asociación

`ConsumoInsumo` resuelve el detalle de una relación entre intervención y material: conserva cantidad, lote y fecha. Es una clase ordinaria de detalle con dos relaciones binarias, no la notación especial UML de clase de asociación. Pertenece a una sola intervención (rombo negro en `Intervencion`) y no se comparte entre procedimientos. En cambio, `Insumo` es el artículo inventariable, que existe aunque ningún procedimiento lo utilice. El consumo descuenta existencias; no destruye el artículo del catálogo. Por ello `ConsumoInsumo - Insumo` es asociación simple, no composición. Varios detalles pueden referirse al mismo insumo y a lotes distintos.

### 5.5. Médicos, pacientes y salas: asociaciones simples

`Intervencion` es el registro central que conecta un paciente (`1`), una sala (`1`), médicos (`1..*`) y detalles de consumo (`0..*`). Médicos, pacientes y salas no son partes poseídas por el procedimiento y sobreviven a su cancelación. Un médico puede participar en muchas intervenciones y una intervención puede reunir varios médicos; no se duplica el procedimiento por cada participante. El emisor inicial del expediente tampoco es su propietario: cambiar la situación laboral del médico no elimina la historia.


## 6. Restricciones de integridad y métodos

1. Los identificadores son únicos dentro de su clase. `salario >= 0`; la fecha de nacimiento no es futura.
2. `Medico.emitirExpediente(paciente)` crea el vínculo exclusivo de titular y emisor. Si el paciente ya tiene expediente, se devuelve el existente o se rechaza la duplicación, sin crear otro.
3. `ExpedienteClinico.agregarConsulta` y `agregarReceta` rechazan una entrada perteneciente a otro expediente. No se transfieren entradas ni se comparten propietarios de partes compuestas.
4. La intervención registrada tiene un paciente, una sala y al menos un médico, incluso si está programada. No se representan borradores sin asignaciones. Sus médicos constituyen un conjunto sin duplicados.
5. `fin > inicio`. Para intervenciones programadas o en curso, no se solapan intervalos de la misma sala, paciente ni médico. `SalaQuirurgica.estaDisponible` evalúa intervalos, no solo un indicador booleano. Canceladas no bloquean agenda.
6. Una intervención programada o cancelada puede tener cero consumos. Al finalizar se exige al menos un detalle de consumo, conservando el estado general `0..*` en el diagrama para todo el ciclo de vida.
7. `cantidad > 0`, expresada en la unidad del insumo, y `stock >= cantidad`. `registrarConsumo` valida e incorpora el detalle y descuenta stock en una sola transacción lógica; si falla la disponibilidad no se registra un consumo parcial. La cantidad de un material indivisible debe ser entera.
8. `lote` no es vacío. Se conserva como dato histórico en el detalle; el modelo no introduce un módulo completo de inventario por lotes, caducidad o compras, que la guía no exige.
9. `EquipoMedico.trasladarA` sustituye atómicamente la asociación de sala actual. Nunca deja al mismo equipo en dos salas a la vez. El cierre de una sala permite conservar equipos sin ubicación y mantiene el historial de intervenciones.
10. La baja de pacientes, médicos, salas e insumos es lógica cuando existen referencias históricas. La composición no impone borrar registros clínicos. Estos son criterios de integridad del modelo académico, no una política legal de conservación.

Las operaciones son responsabilidades de dominio con firma y tipos; no son una implementación ejecutable. La comprobación del modelo valida su estructura y notación, sin simular un sistema hospitalario implementado.


## 7. Trazabilidad con el enunciado

| Requerimiento del caso 1 | Elementos que lo satisfacen |
|---|---|
| Empleados con ID, nombre y salario; médicos, enfermeros y administrativos | Clase abstracta `Empleado`, tres atributos privados, tres generalizaciones y operaciones de rol |
| Especialidad del médico y emisión de expedientes | `Medico.especialidad`, `emitirExpediente` y asociación `emite` con emisor único |
| Expediente dependiente y vitalicio del paciente | Composición `Paciente - ExpedienteClinico`, propiedad exclusiva `1:1` y regla de conservación histórica |
| Múltiples consultas y recetas | Composiciones desde el expediente a `Consulta` y `Receta`, `0..*` con mínimo justificado por ciclo de vida |
| Salas quirúrgicas y equipos trasladables (vínculo débil) | `SalaQuirurgica`, `EquipoMedico`, agregación compartida y operación `trasladarA` |
| Intervenciones con médicos, pacientes, salas e insumos consumidos | `Intervencion`, tres asociaciones de participantes, composición de `ConsumoInsumo` y asociación a `Insumo` |
| Clases, atributos con visibilidad, métodos y multiplicidades | Trece clases con miembros tipados y extremos explícitos en el modelo y la exportación |

## 8. Revisión y entrega

Abrir el `.mdj` en StarUML y seleccionar **HealthFlow - Diagrama de clases completo**. El PNG se exporta desde ese mismo proyecto con el renderizador de StarUML; cualquier marca de licencia que aplique se conserva. El modelo es editable: clases, atributos, parámetros y relaciones son elementos UML nativos, no una imagen incrustada.

Para la revisión entre pares, comprobar especialmente el rombo en el extremo propietario, las multiplicidades de ambos extremos, la justificación del mínimo cero, los consumos con cantidad y lote, y la coincidencia entre diccionario, `.mdj` y PNG. El PR debe ir de `feature/modelo-hospital` a `main` y recibir aprobación de otro integrante antes de integrarse.

La existencia de estos archivos no acredita revisiones ni commits en GitHub. La trazabilidad y el reparto real de trabajo se verifican en el repositorio del equipo.
