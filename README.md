# ms-users — Microservicio de Gestión de Usuarios

## PulpApp — Sistema Distribuido de Venta Online de Pulpas Naturales

Microservicio responsable de la autenticación JWT, gestión de usuarios, pedidos internos y postulaciones laborales del sistema PulpApp.

---

## Tecnologías

| Tecnología | Versión |
|-----------|---------|
| Java | 17 |
| Spring Boot | 4.0.3 |
| Spring Security | Incluido en Boot |
| JJWT | 0.12.6 |
| PostgreSQL | 15 |
| Liquibase | Incluido en Boot |
| Lombok | 1.18.38 |
| MapStruct | 1.5.5.Final |
| Maven | 3.x |

---

## Puerto

```
8081 (local)
8081:8081 (Docker)
```

---

## Estructura de paquetes

```
com.pulpapp.ms_users/
├── config/
│   ├── SecurityConfig.java        ← Cadena de filtros JWT + reglas RBAC
│   ├── CorsConfig.java            ← Habilitación de CORS global
│   └── LiquibaseConfig.java       ← Configuración de migraciones
│
├── controller/
│   ├── AuthController.java        ← POST /auth/login, POST /auth/register
│   ├── UserController.java        ← CRUD /users
│   ├── PedidoController.java      ← CRUD /pedidos
│   └── JobApplicationController.java ← /job-applications
│
├── service/
│   ├── AuthService.java           ← Lógica de login y registro con JWT
│   ├── UserServiceImpl.java       ← CRUD usuarios + encriptación BCrypt
│   ├── PedidoServiceImpl.java     ← Gestión de pedidos internos
│   └── JobApplicationService.java ← Postulaciones + almacenamiento de CVs
│
├── security/
│   ├── JwtService.java            ← Genera, firma y valida tokens JWT
│   ├── JwtAuthFilter.java         ← OncePerRequestFilter — intercepta cada request
│   ├── JwtAuthEntryPoint.java     ← Responde 401 en JSON (no HTML)
│   ├── UserDetailsServiceImpl.java ← Carga usuario por email desde la DB
│   └── UserPrincipal.java         ← Adapter User → UserDetails
│
├── entity/
│   ├── User.java                  ← Entidad usuario con rol
│   ├── Role.java                  ← Enum: ROLE_ADMIN, ROLE_SELLER
│   ├── Pedido.java                ← Entidad pedido interno
│   └── JobApplication.java        ← Entidad postulación laboral
│
├── dto/
│   ├── AuthResponseDTO.java       ← {token, email, name, role}
│   ├── LoginRequestDTO.java       ← {email, password}
│   ├── RegisterRequestDTO.java    ← {cedula, telefono, name, email, password, direccion, role}
│   ├── UserRequestDTO.java        ← Entrada para crear/actualizar usuario
│   ├── UserResponseDTO.java       ← Salida de datos del usuario
│   ├── PedidoDTO.java             ← Transferencia de pedidos
│   ├── JobApplicationRequestDTO.java ← Entrada de postulación
│   └── JobApplicationResponseDTO.java ← Salida de postulación
│
├── repository/
│   ├── UserRepository.java        ← findByCedula, findByEmail, existsByCedula
│   ├── PedidoRepository.java      ← JpaRepository básico
│   └── JobApplicationRepository.java ← findAllByOrderByCreatedAtDesc
│
├── mapper/
│   └── UserMapper.java            ← Conversión User ↔ DTOs
│
├── exception/
│   ├── GlobalExceptionHandler.java ← Manejo centralizado de errores
│   ├── BadCredentialsException.java ← 401 credenciales inválidas
│   └── ResourceNotFoundException.java ← 404 recurso no encontrado
│
└── core/
    ├── BaseServiceImpl.java       ← Clase base genérica para CRUD
    └── IBaseService.java          ← Interfaz base genérica
```

---

## Endpoints

### Autenticación — Públicos

| Método | Ruta | Descripción | Body |
|--------|------|-------------|------|
| POST | `/auth/register` | Registra usuario y retorna JWT | `RegisterRequestDTO` |
| POST | `/auth/login` | Autentica y retorna JWT | `LoginRequestDTO` |

**Ejemplo registro:**
```json
POST /auth/register
{
  "cedula": "123456789",
  "telefono": "3001234567",
  "name": "Juan Pérez",
  "email": "juan@pulpapp.com",
  "password": "mipassword",
  "direccion": "Calle 10 # 20-30",
  "role": "ROLE_SELLER"
}
```

**Respuesta:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9...",
  "email": "juan@pulpapp.com",
  "name": "Juan Pérez",
  "role": "ROLE_SELLER"
}
```

---

### Usuarios

| Método | Ruta | Acceso | Descripción |
|--------|------|--------|-------------|
| GET | `/users` | `ROLE_ADMIN` | Listar todos los usuarios |
| GET | `/users/{id}` | Autenticado | Buscar por ID |
| GET | `/users/cedula/{cedula}` | Público | Buscar por cédula |
| GET | `/users/validar/{cedula}/{telefono}` | Público | Validar cédula + teléfono |
| POST | `/users` | Público | Crear usuario |
| PUT | `/users/{id}` | Público | Actualizar usuario |
| DELETE | `/users/{id}` | `ROLE_ADMIN` | Eliminar usuario |

---

### Pedidos internos

| Método | Ruta | Acceso | Descripción |
|--------|------|--------|-------------|
| POST | `/pedidos` | Público | Crear pedido |
| GET | `/pedidos` | Autenticado | Listar pedidos |
| GET | `/pedidos/{id}` | Autenticado | Buscar pedido por ID |
| DELETE | `/pedidos/{id}` | Autenticado | Eliminar pedido |

---

### Postulaciones laborales

| Método | Ruta | Acceso | Descripción |
|--------|------|--------|-------------|
| POST | `/job-applications` | Público | Enviar postulación con CV opcional |
| GET | `/job-applications` | `ROLE_ADMIN` | Listar todas las postulaciones |
| GET | `/job-applications/{id}/download` | `ROLE_ADMIN` | Descargar CV adjunto |

**Ejemplo postulación (multipart/form-data):**
```
fullName  = Juan Pérez
email     = juan@ejemplo.com
phone     = 3001234567
position  = Operario de producción
message   = Me interesa trabajar con ustedes
file      = [archivo PDF]
```

---

## Modelo de datos

### Tabla `users`

| Columna | Tipo | Restricción |
|---------|------|-------------|
| id | BIGINT | PK, autoincrement |
| cedula | VARCHAR(20) | NOT NULL, UNIQUE |
| telefono | VARCHAR(20) | nullable |
| name | VARCHAR(150) | NOT NULL |
| email | VARCHAR(150) | NOT NULL, UNIQUE |
| password | VARCHAR(255) | NOT NULL (BCrypt) |
| direccion | VARCHAR(255) | NOT NULL |
| role | VARCHAR(20) | NOT NULL, default `ROLE_SELLER` |

### Tabla `pedidos`

| Columna | Tipo | Restricción |
|---------|------|-------------|
| id | BIGINT | PK, autoincrement |
| descripcion | VARCHAR(255) | NOT NULL |
| total | DOUBLE PRECISION | NOT NULL |
| user_id | BIGINT | FK → users (RESTRICT) |

### Tabla `job_applications`

| Columna | Tipo | Restricción |
|---------|------|-------------|
| id | BIGINT | PK, autoincrement |
| full_name | VARCHAR(150) | NOT NULL |
| email | VARCHAR(150) | NOT NULL |
| phone | VARCHAR(50) | NOT NULL |
| position | VARCHAR(150) | nullable |
| message | TEXT | nullable |
| cv_file | VARCHAR(255) | nullable (nombre UUID del archivo) |
| created_at | TIMESTAMP | NOT NULL, default NOW() |

---

## Seguridad JWT

### Algoritmo y configuración

- **Algoritmo:** HMAC-SHA256
- **Clave:** Base64 decodificada desde variable de entorno `JWT_SECRET`
- **Expiración:** 24 horas (configurable con `JWT_EXPIRATION_MS`)

### Estructura del token

```json
{
  "role": "ROLE_ADMIN",
  "sub": "admin@pulpapp.com",
  "iat": 1234567890,
  "exp": 1234654290
}
```

### Flujo de autenticación

```
1. Cliente → POST /auth/login {email, password}
2. AuthService → AuthenticationManager.authenticate()
3. DaoAuthenticationProvider → UserDetailsServiceImpl.loadUserByUsername()
4. BCrypt verifica la contraseña
5. JwtService.generateToken() → firma el token con HMAC-SHA256
6. Respuesta: {token, email, name, role}

En cada request posterior:
7. JwtAuthFilter extrae "Bearer <token>" del header Authorization
8. JwtService.extractUsername() → obtiene el email del payload
9. UserDetailsServiceImpl carga el usuario desde la DB
10. JwtService.isTokenValid() → verifica firma y expiración
11. SecurityContextHolder establece la autenticación con los roles
```

### Roles disponibles

| Rol | Descripción |
|-----|-------------|
| `ROLE_ADMIN` | Acceso completo — gestión de usuarios, productos, pedidos y postulaciones |
| `ROLE_SELLER` | Acceso limitado — consulta de productos y pedidos propios |

### Tabla de permisos

| Endpoint | Público | ROLE_SELLER | ROLE_ADMIN |
|----------|---------|-------------|------------|
| POST /auth/** | ✅ | ✅ | ✅ |
| GET /products | ✅ | ✅ | ✅ |
| POST/PUT/DELETE /products | ❌ | ❌ | ✅ |
| GET /orders | ❌ | ✅ | ✅ |
| GET /users | ❌ | ❌ | ✅ |
| DELETE /users | ❌ | ❌ | ✅ |
| GET /job-applications | ❌ | ❌ | ✅ |
| POST /job-applications | ✅ | ✅ | ✅ |

---

## Migraciones Liquibase

| Changeset | Descripción |
|-----------|-------------|
| `1-create-users-table` | Tabla `users` con cédula y email únicos |
| `2-create-pedidos-table` | Tabla `pedidos` con FK hacia `users` |
| `3-add-fk-pedidos-to-users` | FK `pedidos.user_id → users.id` con RESTRICT |
| `4-add-role-to-users` | Columna `role` en `users` con default `ROLE_SELLER` |
| `5-create-job-applications-table` | Tabla `job_applications` para postulaciones |

Todos los changesets usan `onFail: MARK_RAN` — son idempotentes y seguros para ejecutar múltiples veces.

---

## Configuración (`application.yml`)

```yaml
server:
  port: ${SERVER_PORT:8081}

spring:
  datasource:
    url: ${SPRING_DATASOURCE_URL:jdbc:postgresql://localhost:5434/pulpapp_db}
    username: ${SPRING_DATASOURCE_USERNAME:postgres}
    password: ${SPRING_DATASOURCE_PASSWORD:1234}

  jpa:
    hibernate:
      ddl-auto: none        # Liquibase gestiona el esquema
    show-sql: true

  liquibase:
    change-log: classpath:db/changelog/changelog-master.yml

  servlet:
    multipart:
      max-file-size: 5MB    # Límite para CVs adjuntos
      max-request-size: 10MB

jwt:
  secret: ${JWT_SECRET:...} # Clave Base64 para firmar tokens
  expiration-ms: ${JWT_EXPIRATION_MS:86400000}  # 24 horas

app:
  uploads-dir: ${UPLOADS_DIR:uploads/cv}  # Directorio de CVs
```

### Variables de entorno (Docker)

| Variable | Descripción | Default |
|----------|-------------|---------|
| `SERVER_PORT` | Puerto del servicio | `8081` |
| `SPRING_DATASOURCE_URL` | URL de PostgreSQL | `jdbc:postgresql://localhost:5434/pulpapp_db` |
| `SPRING_DATASOURCE_USERNAME` | Usuario DB | `postgres` |
| `SPRING_DATASOURCE_PASSWORD` | Contraseña DB | `1234` |
| `JWT_SECRET` | Clave secreta Base64 para JWT | valor por defecto incluido |
| `JWT_EXPIRATION_MS` | Expiración del token en ms | `86400000` (24h) |
| `UPLOADS_DIR` | Directorio de CVs en el contenedor | `uploads/cv` |

---

## Manejo de errores

Todos los errores retornan JSON con el formato:

```json
{
  "status": 401,
  "error": "Unauthorized",
  "message": "Token JWT ausente, inválido o expirado"
}
```

| Excepción | Código HTTP |
|-----------|-------------|
| `BadCredentialsException` | 401 |
| `AccessDeniedException` | 403 |
| `ResourceNotFoundException` | 404 |
| `RuntimeException` | 404 |
| `MethodArgumentNotValidException` | 400 |
| `IllegalArgumentException` | 400 |
| `Exception` (fallback) | 500 |

---

## Levantar el servicio

```bash
# Con Docker Compose (recomendado)
docker-compose up --build ms-users

# Solo este servicio tras cambios
docker-compose up --build ms-users

# Ver logs
docker-compose logs -f ms-users
```

El servicio queda disponible en `http://localhost:8081` y accesible desde el API Gateway en `http://localhost:8090`.

