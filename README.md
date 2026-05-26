# Jira-Like Task Management System

A production-ready Spring Boot backend that replicates core Jira functionality: projects, tasks, user assignment, comments, audit history, and dashboard metrics.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Spring Boot 3.2 |
| ORM | Spring Data JPA / Hibernate |
| Database (default) | H2 In-Memory (zero config) |
| Database (production) | MySQL 8 |
| Validation | Spring Validation (JSR-380) |
| API Docs | SpringDoc OpenAPI 2 (Swagger UI) |
| Logging | SLF4J + Logback |
| Build | Maven |
| Java | 17+ |

---

## Project Structure

```
src/main/java/com/taskmanager/
├── TaskManagementApplication.java   ← Entry point
├── config/
│   ├── DataInitializer.java         ← Sample data loader (H2 profile)
│   └── OpenApiConfig.java           ← Swagger configuration
├── controller/
│   ├── UserController.java
│   ├── ProjectController.java
│   ├── TaskController.java
│   └── DashboardController.java
├── dto/
│   ├── request/                     ← Inbound request DTOs
│   └── response/                    ← Outbound response DTOs
├── entity/
│   ├── User.java
│   ├── Project.java
│   ├── Task.java
│   ├── TaskComment.java
│   └── TaskActivity.java
├── enums/
│   ├── TaskStatus.java              ← OPEN | IN_PROGRESS | BLOCKED | DONE
│   ├── TaskPriority.java            ← LOW | MEDIUM | HIGH | CRITICAL
│   ├── UserRole.java                ← ADMIN | MANAGER | DEVELOPER | TESTER
│   └── ActivityType.java
├── exception/
│   ├── GlobalExceptionHandler.java  ← @ControllerAdvice
│   ├── ResourceNotFoundException.java
│   └── DuplicateResourceException.java
├── repository/                      ← Spring Data JPA repositories
├── service/                         ← Service interfaces
│   └── impl/                        ← Service implementations
└── util/
    └── MapperUtil.java              ← Entity → DTO mapping
```

---

## Quick Start (H2 – No database setup required)

### Prerequisites
- Java 17+
- Maven 3.6+
- Eclipse IDE (or IntelliJ / VS Code)

### Step 1 – Import into Eclipse

1. Open Eclipse → **File → Import → Maven → Existing Maven Projects**
2. Browse to the `jira-task-management` folder and click **Finish**
3. Wait for Maven to download dependencies (first run may take 2–3 minutes)

### Step 2 – Run the application

**Option A – Eclipse:**
Right-click `TaskManagementApplication.java` → **Run As → Spring Boot App**

**Option B – Maven terminal:**
```bash
cd jira-task-management
mvn spring-boot:run
```

**Option C – JAR:**
```bash
mvn clean package -DskipTests
java -jar target/jira-task-management-1.0.0.jar
```

### Step 3 – Verify

Open a browser and visit:

| URL | Purpose |
|---|---|
| http://localhost:8080/swagger-ui.html | Interactive API documentation |
| http://localhost:8080/h2-console | H2 database browser |
| http://localhost:8080/api/dashboard | Global dashboard (JSON) |

> **H2 Console settings:** JDBC URL `jdbc:h2:mem:taskmanagerdb`, username `sa`, password *(blank)*

Sample data (4 users, 2 projects, 7 tasks, 4 comments) is automatically loaded on startup.

---

## Switch to MySQL (Production)

### Step 1 – Create the database
```sql
CREATE DATABASE taskmanagerdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Step 2 – Update credentials

Edit `src/main/resources/application-mysql.properties`:
```properties
spring.datasource.username=YOUR_USERNAME
spring.datasource.password=YOUR_PASSWORD
```

### Step 3 – Activate the MySQL profile

Edit `application.properties`:
```properties
spring.profiles.active=mysql
```

Tables are created automatically by Hibernate (`ddl-auto=update`).

---

## API Reference

All responses follow this envelope:
```json
{
  "success": true,
  "message": "Operation successful",
  "data": { ... },
  "timestamp": "2026-05-25T10:00:00"
}
```

### Users  `POST /api/users`
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/users` | Create user |
| GET | `/api/users` | List users (paginated) |
| GET | `/api/users/{id}` | Get user by ID |
| PUT | `/api/users/{id}` | Update user |
| DELETE | `/api/users/{id}` | Delete user |

**Create User – Request body:**
```json
{
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "role": "ADMIN"
}
```
Valid roles: `ADMIN` `MANAGER` `DEVELOPER` `TESTER`

---

### Projects  `/api/projects`
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/projects` | Create project |
| GET | `/api/projects` | List projects (paginated, optional `?keyword=`) |
| GET | `/api/projects/{id}` | Get project (includes progress %) |
| PUT | `/api/projects/{id}` | Update project |
| DELETE | `/api/projects/{id}` | Delete project |

---

### Tasks  `/api/tasks`
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/tasks` | Create task |
| GET | `/api/tasks` | List all tasks (`?keyword=` `?status=`) |
| GET | `/api/tasks/{id}` | Get task by ID |
| PUT | `/api/tasks/{id}` | Update task (auto-audits every change) |
| DELETE | `/api/tasks/{id}` | Delete task |
| GET | `/api/tasks/project/{projectId}` | Tasks by project (`?status=` `?priority=` `?keyword=`) |
| GET | `/api/tasks/user/{userId}` | Tasks assigned to user |
| GET | `/api/tasks/overdue` | All overdue tasks |
| GET | `/api/tasks/overdue/project/{id}` | Overdue tasks in a project |

**Create Task – Request body:**
```json
{
  "title": "Implement JWT auth",
  "description": "Login and registration using JWT",
  "status": "OPEN",
  "priority": "HIGH",
  "projectId": 1,
  "assignedUserId": 2,
  "dueDate": "2026-06-30"
}
```
Valid statuses: `OPEN` `IN_PROGRESS` `BLOCKED` `DONE`  
Valid priorities: `LOW` `MEDIUM` `HIGH` `CRITICAL`

**Update Task – `updatedById` is required for audit tracking:**
```json
{
  "status": "IN_PROGRESS",
  "priority": "CRITICAL",
  "updatedById": 1
}
```

---

### Comments  `/api/tasks/{taskId}/comments`
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/tasks/{taskId}/comments` | Add comment |
| GET | `/api/tasks/{taskId}/comments` | List comments (paginated) |

---

### Task History  `/api/tasks/{taskId}/history`
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/tasks/{taskId}/history` | Full audit trail (newest first) |

Each history entry records: activity type, old value, new value, who changed it, and when.

---

### Dashboard  `/api/dashboard`
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/dashboard` | Global metrics (all projects) |
| GET | `/api/dashboard/project/{id}` | Per-project metrics + progress % |

---

## Pagination & Filtering

All list endpoints support these query parameters:

| Parameter | Default | Description |
|---|---|---|
| `page` | `0` | Zero-based page number |
| `size` | `10` | Items per page |
| `sortBy` | `id` / `createdAt` | Field to sort by |
| `direction` | `asc` | `asc` or `desc` |

Filter parameters specific to tasks:

| Parameter | Values | Endpoint |
|---|---|---|
| `status` | `OPEN` `IN_PROGRESS` `BLOCKED` `DONE` | `/tasks`, `/tasks/project/{id}` |
| `priority` | `LOW` `MEDIUM` `HIGH` `CRITICAL` | `/tasks/project/{id}` |
| `keyword` | any string | `/tasks`, `/tasks/project/{id}`, `/projects` |

---

## Error Handling

| HTTP Status | Scenario |
|---|---|
| 400 | Validation failure (missing required fields, bad email format) |
| 404 | Resource not found |
| 409 | Duplicate email on user creation |
| 500 | Unexpected server error |

Validation errors return a map of field → message:
```json
{
  "success": false,
  "message": "Validation failed",
  "data": {
    "email": "Email must be valid",
    "title": "Task title is required"
  }
}
```

---

## Running Tests

```bash
mvn test
```

Tests run against H2 by default (no external database needed).

---

## Common Issues

**Port 8080 already in use**
```properties
# Add to application.properties
server.port=9090
```

**Eclipse red errors after import**
- Right-click project → **Maven → Update Project** (Alt+F5)
- Ensure Java 17 JDK is configured in **Window → Preferences → Java → Installed JREs**

**MySQL connection refused**
- Verify MySQL is running: `mysql -u root -p`
- Check credentials in `application-mysql.properties`
- Ensure `spring.profiles.active=mysql` in `application.properties`
