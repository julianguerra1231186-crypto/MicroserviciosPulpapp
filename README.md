# ms-products — Microservicio de Gestión de Productos

## PulpApp — Sistema Distribuido de Venta Online de Pulpas Naturales

Microservicio responsable del catálogo de pulpas de fruta. Gestiona productos y categorías, expone una API REST consumida por el frontend y por `ms-orders` para validar precios al momento de crear pedidos.

---

## Tecnologías

| Tecnología | Versión |
|-----------|---------|
| Java | 17 |
| Spring Boot | 4.0.3 |
| Spring Data JPA | Incluido en Boot |
| Spring Validation | Incluido en Boot |
| PostgreSQL | 15 |
| Liquibase | Incluido en Boot |
| Lombok | Incluido en Boot |
| Maven | 3.x |

---

## Puerto

```
8082 (local)
8082:8082 (Docker)
```

---

## Estructura de paquetes

```
com.pulpapp.msproducts/
├── config/
│   ├── CorsConfig.java            ← Habilitación de CORS para todos los orígenes
│   └── LiquibaseConfig.java       ← Configuración de migraciones de BD
│
├── controller/
│   └── ProductController.java     ← CRUD completo de productos en /products
│
├── service/
│   └── ProductService.java        ← Lógica de negocio: validación de nombre único,
│                                     mapeo DTO ↔ entidad, operaciones CRUD
│
├── entity/
│   ├── Product.java               ← Entidad producto con relación a Category
│   └── Category.java              ← Entidad categoría (1 categoría → N productos)
│
├── dto/
│   ├── ProductRequestDTO.java     ← Entrada: nombre, descripción, precio, stock,
│   │                                 disponibilidad, imageUrl (con validaciones)
│   └── ProductResponseDTO.java    ← Salida: id + todos los campos del producto
│
├── repository/
│   ├── ProductRepository.java     ← JpaRepository + búsqueda por nombre único
│   └── CategoryRepository.java    ← JpaRepository básico para categorías
│
├── exception/
│   ├── GlobalExceptionHandler.java ← Manejo centralizado de errores en JSON
│   └── ResourceNotFoundException.java ← 404 producto no encontrado
│
└── MsProductsApplication.java     ← Clase principal Spring Boot
```

---

## Endpoints

### Productos

| Método | Ruta | Código | Descripción |
|--------|------|--------|-------------|
| GET | `/products` | 200 | Listar todos los productos |
| GET | `/products/{id}` | 200 | Obtener producto por ID |
| POST | `/products` | 201 | Crear nuevo producto |
| PUT | `/products/{id}` | 200 | Actualizar producto existente |
| DELETE | `/products/{id}` | 204 | Eliminar producto |

> **Nota de seguridad:** Las reglas de acceso son gestionadas por el API Gateway y `ms-users`. `GET /products` es público. `POST`, `PUT` y `DELETE` requieren `ROLE_ADMIN`.

---

### Ejemplos de uso

**Listar todos los productos:**
```
GET http://localhost:8090/products
```

**Obtener producto por ID:**
```
GET http://localhost:8090/products/1
```

**Crear producto (requiere token ADMIN):**
```
POST http://localhost:8090/products
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "Pulpa de Mango",
  "description": "Pulpa natural de mango sin conservantes",
  "price": 8500.0,
  "stock": 100,
  "available": true,
  "imageUrl": "img/mango.png"
}
```

**Respuesta `201 Created`:**
```json
{
  "id": 1,
  "name": "Pulpa de Mango",
  "description": "Pulpa natural de mango sin conservantes",
  "price": 8500.0,
  "stock": 100,
  "available": true,
  "imageUrl": "img/mango.png"
}
```

**Actualizar producto (requiere token ADMIN):**
```
PUT http://localhost:8090/products/1
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "Pulpa de Mango Premium",
  "description": "Pulpa natural de mango seleccionado",
  "price": 9500.0,
  "stock": 80,
  "available": true,
  "imageUrl": "img/mango1.png"
}
```

**Eliminar producto (requiere token ADMIN):**
```
DELETE http://localhost:8090/products/1
Authorization: Bearer <token>
```

---

## Modelo de datos

### Tabla `category`

| Columna | Tipo | Restricción |
|---------|------|-------------|
| id | BIGINT | PK, autoincrement |
| name | VARCHAR(100) | NOT NULL, UNIQUE |
| description | VARCHAR(255) | nullable |

**Categorías iniciales (seed):**

| ID | Nombre | Descripción |
|----|--------|-------------|
| 1 | Tropicales | Pulpas de frutas tropicales como mango, maracuyá y lulo |
| 2 | Cítricas | Pulpas de frutas cítricas como naranja y limón |
| 3 | Berries | Pulpas de frutos rojos como fresa y mora |
| 4 | Exóticas | Pulpas de frutas exóticas y poco comunes |

---

### Tabla `products`

| Columna | Tipo | Restricción |
|---------|------|-------------|
| id | BIGINT | PK, autoincrement |
| name | VARCHAR(120) | NOT NULL, UNIQUE |
| description | VARCHAR(255) | NOT NULL |
| price | DOUBLE PRECISION | NOT NULL |
| stock | INTEGER | NOT NULL |
| available | BOOLEAN | NOT NULL |
| image_url | VARCHAR(255) | nullable |
| category_id | BIGINT | FK → category (SET NULL on delete), nullable |

> Hibernate convierte `imageUrl` → `image_url` automáticamente (snake_case).

---

### Relación entre entidades

```
Category (1) ──────────── (N) Product
    id                         id
    name                       name
    description                description
                               price
                               stock
                               available
                               image_url
                               category_id (FK)
```

- Un producto puede pertenecer a una categoría (nullable)
- Si se elimina una categoría, los productos quedan con `category_id = NULL` (SET NULL)
- Un nombre de producto debe ser único (validado en servicio y a nivel de BD)

---

## DTOs

### `ProductRequestDTO` — Entrada

| Campo | Tipo | Validación |
|-------|------|-----------|
| name | String | `@NotBlank`, máx 120 caracteres |
| description | String | `@NotBlank`, máx 255 caracteres |
| price | Double | `@NotNull`, mayor que 0 |
| stock | Integer | `@NotNull`, mayor o igual a 0 |
| available | Boolean | `@NotNull` |
| imageUrl | String | opcional, máx 255 caracteres |

### `ProductResponseDTO` — Salida

| Campo | Tipo |
|-------|------|
| id | Long |
| name | String |
| description | String |
| price | Double |
| stock | Integer |
| available | Boolean |
| imageUrl | String |

---

## Lógica de negocio

### Validación de nombre único

Antes de crear o actualizar un producto, el servicio verifica que no exista otro con el mismo nombre (ignorando mayúsculas/minúsculas):

- **Crear:** `existsByNameIgnoreCase(name)` — si existe, retorna `409 Conflict`
- **Actualizar:** `existsByNameIgnoreCaseAndIdNot(name, id)` — excluye el propio producto del chequeo

### Mapeo DTO → Entidad

El método `applyDtoToEntity` aplica `trim()` a los campos de texto para evitar espacios innecesarios antes de persistir.

---

## Migraciones Liquibase

| Changeset | Descripción |
|-----------|-------------|
| `1-create-category-table` | Tabla `category` con nombre único |
| `2-create-products-table` | Tabla `products` con todos sus campos base |
| `3-add-category-fk-to-products` | Agrega `category_id` a `products` y crea FK con `SET NULL` |
| `4-seed-categories` | Inserta las 4 categorías iniciales si la tabla está vacía |

Todos los changesets usan `onFail: MARK_RAN` — son idempotentes y seguros para ejecutar múltiples veces.

---

## Manejo de errores

Todos los errores retornan JSON:

```json
{
  "error": "Product not found with id: 99"
}
```

| Excepción | Código HTTP | Descripción |
|-----------|-------------|-------------|
| `ResourceNotFoundException` | 404 | Producto no encontrado |
| `ResponseStatusException` (CONFLICT) | 409 | Nombre de producto duplicado |
| `MethodArgumentNotValidException` | 400 | Campos inválidos en el request |
| `Exception` (fallback) | 500 | Error interno del servidor |

---

## Configuración (`application.properties`)

```properties
spring.application.name=ms-products
server.port=${SERVER_PORT:8082}

spring.datasource.url=${SPRING_DATASOURCE_URL:jdbc:postgresql://localhost:5434/pulpapp_db}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME:postgres}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD:1234}
spring.datasource.driver-class-name=org.postgresql.Driver

spring.jpa.hibernate.ddl-auto=none
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

spring.liquibase.change-log=classpath:db/changelog/changelog-master.yml
spring.liquibase.enabled=true
```

### Variables de entorno (Docker)

| Variable | Descripción | Default |
|----------|-------------|---------|
| `SERVER_PORT` | Puerto del servicio | `8082` |
| `SPRING_DATASOURCE_URL` | URL de PostgreSQL | `jdbc:postgresql://localhost:5434/pulpapp_db` |
| `SPRING_DATASOURCE_USERNAME` | Usuario DB | `postgres` |
| `SPRING_DATASOURCE_PASSWORD` | Contraseña DB | `1234` |

---

## Dependencias (`pom.xml`)

| Dependencia | Propósito |
|-------------|-----------|
| `spring-boot-starter-web` | API REST |
| `spring-boot-starter-data-jpa` | Persistencia con Hibernate |
| `spring-boot-starter-validation` | Validaciones con `@Valid` |
| `postgresql` | Driver de base de datos |
| `liquibase-core` | Versionado del esquema de BD |
| `lombok` | Reducción de código boilerplate |
| `spring-boot-starter-test` | Testing |

---

## Levantar el servicio

```bash
# Con Docker Compose (recomendado)
docker-compose up --build ms-products

# Ver logs
docker-compose logs -f ms-products
```

El servicio queda disponible en `http://localhost:8082` y accesible desde el API Gateway en `http://localhost:8090/products`.

---

## Integración con otros microservicios

| Microservicio | Tipo | Descripción |
|---------------|------|-------------|
| `api-gateway` | Consumidor | Enruta `/products/**` hacia este servicio |
| `ms-orders` | Consumidor | Consulta `GET /products/{id}` para obtener el precio real al crear un pedido |
| `ms-users` | Seguridad | Valida el token JWT y el rol antes de permitir escritura |
