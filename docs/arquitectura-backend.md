# Arquitectura del Backend

## 1. Stack Tecnológico

- **Framework**: Quarkus
- **Lenguaje**: Java 17
- **Base de datos**: Oracle Database
- **Acceso a datos**: Hibernate ORM with Panache (patrón Repository)
- **API**: REST (RESTEasy Reactive)
- **Formato de datos**: JSON

## 2. Estilo de arquitectura

Arquitectura en capas (Layered Architecture), donde cada capa tiene una única responsabilidad y solo se comunica con la capa inmediatamente inferior.

```
Cliente (React / Postman)
        ↓
   Resource (Controller)   → expone endpoints REST
        ↓
      Service              → lógica de negocio
        ↓
     Repository            → acceso a datos (Panache)
        ↓
      Entity               → mapeo objeto-relacional
        ↓
    Oracle Database
```

## 3. Estructura de paquetes

```
com.tuempresa/
├── entity/
│   └── Task.java                 → Entidad JPA, mapea la tabla TASKS
├── repository/
│   └── TaskRepository.java       → implements PanacheRepository<Task>
├── service/
│   └── TaskService.java          → lógica de negocio
├── resource/
│   └── TaskResource.java         → @Path("/api/tasks"), endpoints REST
├── dto/
│   ├── TaskRequestDTO.java       → datos de entrada (crear / actualizar)
│   └── TaskResponseDTO.java      → datos de salida (respuesta al cliente)
├── mapper/
│   └── TaskMapper.java           → convierte Task ↔ DTOs
└── exception/
    ├── TaskNotFoundException.java
    └── GlobalExceptionMapper.java  → manejo centralizado de errores HTTP
```

## 4. Decisiones técnicas

| Decisión | Elección | Justificación |
|---|---|---|
| Base de datos | Oracle | Requisito del proyecto |
| Acceso a datos | Hibernate ORM with Panache (Repository) | Reduce código repetitivo, mantiene la Entity desacoplada del resto de capas |
| Estrategia de datos en la API | DTOs separados de la Entity | No expone la estructura interna de la BD, permite evolucionar el modelo sin romper el contrato con el frontend |
| Alcance del dominio | Solo `Task` (sin `Project`) | Mantener el MVP simple; `Project` queda fuera de alcance por ahora |

## 5. Modelo de datos: Task

| Campo | Tipo (Oracle) | Descripción |
|---|---|---|
| `id` | NUMBER (PK) | Identificador único, autogenerado |
| `title` | VARCHAR2(150) | Título de la tarea |
| `description` | VARCHAR2(500) | Descripción detallada |
| `status` | VARCHAR2(20) | Estado: `PENDING`, `IN_PROGRESS`, `DONE` |
| `priority` | VARCHAR2(10) | Prioridad: `LOW`, `MEDIUM`, `HIGH` |
| `due_date` | DATE | Fecha límite |
| `created_at` | TIMESTAMP | Fecha de creación (auditoría) |
| `updated_at` | TIMESTAMP | Última fecha de actualización |

## 6. Flujo de una petición (ejemplo: crear tarea)

```
POST /api/tasks
   ↓
TaskResource.create(TaskRequestDTO)
   ↓
TaskService.createTask(dto)        → valida reglas de negocio
   ↓
TaskMapper.toEntity(dto)           → convierte DTO a Entity
   ↓
TaskRepository.persist(task)       → guarda en Oracle
   ↓
TaskMapper.toResponseDTO(task)     → convierte Entity a DTO de salida
   ↓
TaskResource devuelve 201 Created + JSON
```

## 7. Endpoints planeados (a definir en detalle en el Paso 9)

| Método | Endpoint | Descripción |
|---|---|---|
| GET | `/api/tasks` | Listar todas las tareas |
| GET | `/api/tasks/{id}` | Obtener una tarea por ID |
| POST | `/api/tasks` | Crear una nueva tarea |
| PUT | `/api/tasks/{id}` | Actualizar una tarea existente |
| DELETE | `/api/tasks/{id}` | Eliminar una tarea |

## 8. Dependencias Maven necesarias (a agregar en el Paso 5)

- `quarkus-resteasy-reactive`
- `quarkus-resteasy-reactive-jackson`
- `quarkus-hibernate-orm-panache`
- `quarkus-jdbc-oracle`

## 9. Manejo de errores

Se centraliza mediante `GlobalExceptionMapper`, que captura excepciones de negocio (ej. `TaskNotFoundException`) y las traduce a respuestas HTTP consistentes:

| Excepción | Código HTTP |
|---|---|
| `TaskNotFoundException` | 404 Not Found |
| Errores de validación | 400 Bad Request |
| Errores no controlados | 500 Internal Server Error |
