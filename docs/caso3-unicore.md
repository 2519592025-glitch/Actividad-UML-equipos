# CASO 3: Sistema de Gestión Académica y Campus Universitario ("UniCore")

## 1. Introducción
El presente documento contiene la especificación y documentación técnica del modelo de clases UML para la plataforma académica **UniCore** de la Universidad Tecnológica de El Salvador. El sistema abstrae los procesos de gestión académica, estructura curricular, oferta de grupos, asignación de espacios físicos y control de matriculación de estudiantes.

---

## 2. Diagrama de Clases UML
El diagrama fuente se encuentra disponible en `models/caso3-unicore.mdj` y la exportación gráfica en `models/caso3-unicore.png`.

---

## 3. Diccionario de Clases y Atributos

### 3.1. Clase `Facultad`

Representa una unidad académica mayor dentro de la universidad.

* **Atributos:**
  * `- idFacultad: String`
  * `- nombre: String`
  * `- decano: String`

* **Métodos:**
  * `+ agregarCarrera(carrera: Carrera): void`
  * `+ obtenerCarreras(): List<Carrera>`

### 3.2. Clase `Carrera`
Programa académico ofertado por una facultad.

* **Atributos:**
  * `- idCarrera: String`
  * `- nombre: String`
  * `- duracionAnios: int`

* **Métodos:**
  * `+ asignarPensum(pensum: Pensum): void`

### 3.3. Clase `Pensum`
Plan de estudios estructurado para una carrera técnica o profesional.

* **Atributos:**
  * `- idPensum: String`
  * `- anioVigilancia: int`
  * `- totalCreditos: int`
* **Métodos:**

  * `+ agregarAsignatura(asignatura: Asignatura): void`

### 3.4. Clase `Asignatura`
Unidad de aprendizaje o materia académica.

* **Atributos:**

  * `- idAsignatura: String`
  * `- nombre: String`
  * `- unidadesValorativas: int`
* **Métodos:**

  * `+ agregarPrerrequisito(materia: Asignatura): void`
  * `+ validarPrerrequisitos(estudiante: Estudiante): boolean`

### 3.5. Clase `Estudiante`
Usuario que cursa programas académicos en la universidad.

* **Atributos:**
  * `- carnet: String`
  * `- nombres: String`
  * `- apellidos: String`
  * `- emailInstitucional: String`

* **Métodos:**
  * `+ inscribirCiclo(ciclo: CicloAcademico): Matricula`

### 3.6. Clase `CicloAcademico`
Período electivo específico (ej. Ciclo 01-2026).

* **Atributos:**
  * `- idCiclo: String`
  * `- anio: int`
  * `- numeroCiclo: int`
  * `- fechaInicio: Date`
  * `- fechaFin: Date`

### 3.7. Clase `Matricula`
Registro formal de la inscripción de asignaturas/grupos por parte de un estudiante en un ciclo.

* **Atributos:**
  * `- idMatricula: String`
  * `- fechaInscripcion: Date`
  * `- estado: String`

* **Métodos:**

  * `+ agregarGrupo(grupo: GrupoClase): void`
  * `+ calcularMonto(): double`

### 3.8. Clase `GrupoClase`
Sección o cupo habilitado para impartir una asignatura en un ciclo lectivo.

* **Atributos:**
  * `- idGrupo: String`
  * `- numeroGrupo: int`
  * `- horario: String`
  * `- cupoMaximo: int`

### 3.9. Clase `Docente`
Profesor encargado de impartir la docencia del grupo de clase.

* **Atributos:**
  * `- idDocente: String`
  * `- nombres: String`
  * `- especialidad: String`

### 3.10. Clase `Aula`

Espacio físico o virtual dentro del campus donde se desarrollan las sesiones.

* **Atributos:**
  * `- idAula: String`
  * `- edificio: String`
  * `- capacidad: int`
  * `- tipo: String`

---

## 4. Justificación de Decisiones de Diseño (Composición vs. Agregación)

Según la especificación formal del problema y las reglas de diseño orientadas a objetos, se aplicaron los siguientes criterios:

### 4.1. Composición: Facultad -> Carrera

* **Tipo:** Composición fuerte (Rombo relleno).

* **Justificación:** El enunciado establece explícitamente que la carrera pertenece fuertemente a la facultad, y si la facultad cierra, las carreras asociadas se dan de baja. Existe una dependencia de ciclo de vida total: la entidad Carrera no posee existencia independiente sin una Facultad contenedora.

### 4.2. Composición: Carrera -> Pensum

* **Tipo:** Composición fuerte.

* **Justificación:** Un Pensum representa el plan de estudios único diseñado específicamente para una Carrera. Si la carrera desaparece de la oferta académica institucional, el pensum pierde su razón de ser en el sistema.

### 4.3. Agregación: Pensum -> Asignatura

* **Tipo:** Agregación (Rombo transparente).

* **Justificación:** Una Asignatura (por ejemplo, Matemática I o Programación I) puede formar parte de múltiples pensums pertenecientes a distintas carreras. La eliminación de un pensum no destruye la asignatura del catálogo general de la universidad.

### 4.4. Agregación: GrupoClase -> Aula

* **Tipo:** Agregación (Rombo transparente).

* **Justificación:** El requerimiento indica que las Aulas pertenecen al inventario del campus, pero existen independientemente de si hay clases asignadas a ellas en un ciclo determinado. El aula es una entidad física permanente; asignarle o desasignarle un GrupoClase no afecta su ciclo de vida.

### 4.5. Relación Reflexiva: Asignatura -> Asignatura

* **Tipo:** Asociación dirigida reflexiva (esPrerrequisitoDe).

* **Justificación:** Permite modelar que una materia dependa conceptualmente de otra u otras asignaturas previas para ser cursada.
*