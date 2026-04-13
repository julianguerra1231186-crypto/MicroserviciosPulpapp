# PupaApp - Sistema Distribuido de Venta de Pulpas Naturales
Frontend, Roles JWT y Módulo de Empleados

## PulpApp — Sistema Distribuido de Venta Online de Pulpas Naturales

**Integrantes:** Julian Guerra · Edwin Menéndez — Grupo 5

---

## 1. Resumen de lo implementado

Esta semana se trabajaron tres grandes áreas:

1. Rediseño completo del frontend con experiencia de usuario profesional
2. Integración de autenticación JWT con control de roles (RBAC)
3. Módulo de reclutamiento (Trabaja con nosotros) con persistencia en base de datos

---

## 2. Frontend — Cambios y nuevas vistas

### 2.1 Estructura de archivos nuevos

```
frontend/
├── login.html              ← Reemplazado: formulario email + password con JWT
├── register.html           ← NUEVO: registro de usuarios
├── dashboard.html          ← NUEVO: panel de usuario autenticado
├── trabaja-con-nosotros.html ← NUEVO: página de reclutamiento
├── auth.js                 ← NUEVO: gestión de sesión JWT en el navbar
├── services.js             ← Actualizado: authLogin(), authRegister()
├── css/
│   ├── dashboard.css       ← NUEVO: estilos del dashboard y panel admin
│   └── jobs.css            ← NUEVO: estilos de la página de empleo
└── img/
    ├── slider1.png ... slider5.png
    ├── quienessomos.png
    ├── fotodeleequipo1.png
    └── terminoniodeempleados.png
```

### 2.2 Navbar dinámico (`auth.js`)

El navbar detecta automáticamente si hay una sesión JWT activa y cambia su contenido sin recargar la página.

**Sin sesión activa:**
```
Inicio | Catálogo | Carrito | Iniciar sesión
```

**Con sesión activa:**
```
Inicio | Catálogo | Carrito | 👤 nombre@email.com | [Cerrar sesión]
```

**Lógica implementada:**
- Lee el token JWT de `localStorage` con clave `pulpapp_jwt`
- Decodifica el payload con `atob()` para extraer email, nombre y rol
- Detecta expiración comparando `exp * 1000` con `Date.now()`
- Si el token expiró, lo elimina silenciosamente al cargar la página
- Al hacer logout: limpia `pulpapp_jwt`, `pulpapp_user` y `user` del localStorage
- Redirige a `login.html` tras cerrar sesión

### 2.3 Flujo de autenticación

```
register.html  →  POST /auth/register  →  guarda token  →  dashboard.html
login.html     →  POST /auth/login     →  guarda token  →  dashboard.html
dashboard.html →  sin token válido     →  redirige a login.html
```

**`login.html`** — formulario con email y contraseña. Guarda el token en `localStorage` y redirige al dashboard. Si venía del carrito, regresa al carrito automáticamente usando `pulpapp_redirect`.

**`register.html`** — formulario completo con cédula, teléfono, nombre, email, contraseña y dirección. Llama a `POST /auth/register` y redirige al dashboard tras el registro.

### 2.4 Dashboard (`dashboard.html`)

Vista protegida — si no hay token válido, redirige automáticamente al login.

**Tabs para ROLE_SELLER:**
| Tab | Descripción |
|-----|-------------|
| 👤 Perfil | Datos personales del usuario |
| 📦 Mis pedidos | Historial de pedidos propios |
| 🛒 Nueva compra | Enlace al catálogo |

**Tabs para ROLE_ADMIN:**
| Tab | Descripción |
|-----|-------------|
| 👤 Perfil | Datos personales |
| 📦 Productos | CRUD completo de productos |
| 👥 Clientes | Lista de usuarios registrados |
| 🧑‍💼 Postulaciones | Gestión de candidatos laborales |

### 2.5 Hero Slider (`index.html`)

Carrusel de imágenes implementado con JavaScript vanilla:

- 5 slides: `slider1.png` → `slider5.png`
- Autoplay cada 4 segundos con pausa al hover
- Flechas prev/next con efecto glassmorphism
- Dots de navegación sincronizados
- Soporte para swipe táctil en móvil
- Overlay con texto: *"Pulpas naturales frescas y listas para disfrutar"*
- Botón CTA: *"Comprar ahora"* → `catalog.html`

### 2.6 Franja de beneficios

Barra negra debajo del slider con 4 beneficios:

```
🔒 Pago 100% seguro | 🚚 Envío rápido | 🏠 Para tu negocio o hogar | 🎁 Envío GRATIS desde $99.900
```

### 2.7 Carrusel de productos destacados (Coverflow)

Efecto tipo Netflix/Apple con 5 productos:

- Producto central en escala 1.0, laterales en 0.82 y 0.68
- Botón "Agregar al carrito" solo visible en el producto central
- Flechas y dots de navegación
- Swipe táctil

### 2.8 Sección ¿Quiénes somos?

Diseño en dos columnas:
- Izquierda: imagen `quienessomos.png` con efecto zoom al hover
- Derecha: texto institucional de Frutos & Sabores con cita destacada en verde

### 2.9 CTA Trabaja con nosotros

Botón con efecto confeti al hacer hover, implementado con Canvas API:

- Partículas de colores (verde, naranja, blanco) con gravedad y rotación
- Se activa al pasar el cursor, se detiene al salir
- Redirige a `trabaja-con-nosotros.html`

### 2.10 Carrito — Sección CLIENTE mejorada

**Sin sesión:**
- Solo muestra botón "Iniciar sesión"
- Texto: *"¿No tienes cuenta? Regístrate aquí"*
- Al hacer clic guarda `pulpapp_redirect = cart.html` para volver tras el login

**Con sesión:**
- Avatar con iniciales del usuario
- Campos precargados automáticamente: nombre, teléfono, email, cédula, dirección
- Campos en modo `readonly`
- Botón "Cerrar sesión" dentro del bloque
- Botón "Finalizar compra" bloqueado si no hay sesión activa

---

## 3. Integración de Roles JWT (RBAC)

### 3.1 Roles definidos

| Rol | Descripción |
|-----|-------------|
| `ROLE_ADMIN` | Acceso completo al sistema |
| `ROLE_SELLER` | Solo puede consultar productos y gestionar sus pedidos |

### 3.2 Backend — `ms-users`

**Entidad `User`** — campo `role` agregado:
```java
@Enumerated(EnumType.STRING)
@Column(nullable = false)
private Role role = Role.ROLE_SELLER;
```

**Enum `Role`:**
```java
public enum Role {
    ROLE_ADMIN,
    ROLE_SELLER
}
```

**Liquibase — changeset 4:**
```yaml
- changeSet:
    id: 4-add-role-to-users
    changes:
      - addColumn:
          tableName: users
          columns:
            - column:
                name: role
                type: VARCHAR(20)
                defaultValue: ROLE_SELLER
```

### 3.3 JWT — Claim de rol

El token incluye el rol como claim adicional:

```json
{
  "role": "ROLE_ADMIN",
  "sub": "admin@pulpapp.com",
  "iat": 1234567890,
  "exp": 1234654290
}
```

### 3.4 Endpoints y permisos

| Endpoint | Método | Acceso |
|----------|--------|--------|
| `/auth/**` | cualquiera | Público |
| `/users` (POST) | POST | Público |
| `/users/cedula/**` | GET | Público |
| `/users/validar/**` | GET | Público |
| `/users` (GET) | GET | `ROLE_ADMIN` |
| `/users/**` (DELETE) | DELETE | `ROLE_ADMIN` |
| `/products/**` | GET | Público |
| `/products/**` | POST/PUT/DELETE | `ROLE_ADMIN` |
| `/orders` | POST | Público |
| `/orders/**` | GET | `ROLE_ADMIN`, `ROLE_SELLER` |
| `/job-applications` | POST | Público |
| `/job-applications` | GET | `ROLE_ADMIN` |
| `/job-applications/{id}/download` | GET | `ROLE_ADMIN` |

### 3.5 Componentes de seguridad implementados

| Clase | Responsabilidad |
|-------|----------------|
| `JwtService` | Genera y valida tokens JWT con JJWT 0.12.6 |
| `JwtAuthFilter` | `OncePerRequestFilter` — intercepta cada request |
| `JwtAuthEntryPoint` | Devuelve `401` en JSON (no HTML) |
| `UserPrincipal` | Adapter `UserDetails` para la entidad `User` |
| `UserDetailsServiceImpl` | Carga usuario por email desde la DB |
| `SecurityConfig` | `SecurityFilterChain` con reglas RBAC |
| `AuthService` | Login y registro con generación de JWT |
| `AuthController` | `POST /auth/login` y `POST /auth/register` |

### 3.6 Cómo crear un usuario ADMIN

Desde Postman:

```
POST http://localhost:8090/auth/register
Content-Type: application/json

{
  "cedula": "111222333",
  "telefono": "3001234567",
  "name": "Admin Principal",
  "email": "admin@pulpapp.com",
  "password": "admin123",
  "direccion": "Calle 1 # 2-3",
  "role": "ROLE_ADMIN"
}
```

Respuesta `201 Created`:
```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9...",
  "email": "admin@pulpapp.com",
  "name": "Admin Principal",
  "role": "ROLE_ADMIN"
}
```

### 3.7 Verificar el rol en el token

1. Copia el token de la respuesta
2. Ve a [jwt.io](https://jwt.io)
3. Pega el token en el campo `Encoded`
4. En el panel `Payload` verás:

```json
{
  "role": "ROLE_ADMIN",
  "sub": "admin@pulpapp.com"
}
```

---

## 4. Módulo de Empleados — Trabaja con nosotros

### 4.1 Página `trabaja-con-nosotros.html`

Secciones:

| Sección | Contenido |
|---------|-----------|
| Hero | Imagen `fotodeleequipo1.png` con overlay y CTA |
| Sobre el equipo | Dos columnas: imagen + texto institucional |
| Beneficios | Grid de 4 cards con iconos |
| Testimonios | 3 cards sobre fondo verde oscuro con `terminoniodeempleados.png` |
| Formulario | Campos + adjunto de hoja de vida |
| Mapa | Google Maps embebido |

### 4.2 Backend — Entidad `JobApplication`

```java
@Entity
@Table(name = "job_applications")
public class JobApplication {
    private Long id;
    private String fullName;
    private String email;
    private String phone;
    private String position;
    private String message;
    private String cvFile;       // nombre UUID del archivo en disco
    private LocalDateTime createdAt;
}
```

**Liquibase — changeset 5:**
```yaml
- changeSet:
    id: 5-create-job-applications-table
    changes:
      - createTable:
          tableName: job_applications
          columns:
            - id, full_name, email, phone, position, message, cv_file, created_at
```

### 4.3 Endpoints del módulo

| Endpoint | Método | Acceso | Descripción |
|----------|--------|--------|-------------|
| `/job-applications` | POST | Público | Enviar postulación con CV opcional |
| `/job-applications` | GET | `ROLE_ADMIN` | Listar todas las postulaciones |
| `/job-applications/{id}/download` | GET | `ROLE_ADMIN` | Descargar CV adjunto |

### 4.4 Manejo de archivos

- Los CVs se guardan en `/app/uploads/cv/` dentro del contenedor
- El nombre se genera con `UUID.randomUUID()` para evitar colisiones
- Solo se guarda el nombre del archivo en la DB (campo `cv_file`)
- El directorio es persistente gracias al volumen Docker `cv_uploads`

**`docker-compose.yml` — volumen agregado:**
```yaml
ms-users:
  volumes:
    - cv_uploads:/app/uploads/cv
  environment:
    UPLOADS_DIR: /app/uploads/cv

volumes:
  cv_uploads:
```

### 4.5 Panel ADMIN — Tab Postulaciones

El administrador puede desde el dashboard:

- Ver tabla completa con: ID, Nombre, Email, Teléfono, Cargo, Fecha, CV
- Filtrar en tiempo real por nombre, cargo o email
- Descargar CV con autenticación JWT (fetch + Blob, no enlace directo)
- Ver modal de detalle con toda la información del candidato

**Flujo de descarga con JWT:**
```javascript
// El navegador no puede enviar headers en <a href>, se usa fetch + Blob
const res = await fetch(`/job-applications/${id}/download`, {
    headers: { "Authorization": `Bearer ${token}` }
});
const blob = await res.blob();
const url  = URL.createObjectURL(blob);
const a    = document.createElement("a");
a.href = url;
a.download = filename;
a.click();
URL.revokeObjectURL(url);
```

### 4.6 Pruebas en Postman

**Enviar postulación con CV:**
- Método: `POST`
- URL: `http://localhost:8090/job-applications`
- Body: `form-data`

| Key | Type | Value |
|-----|------|-------|
| fullName | Text | Juan Pérez |
| email | Text | juan@ejemplo.com |
| phone | Text | 3001234567 |
| position | Text | Operario |
| message | Text | Me interesa trabajar |
| file | File | seleccionar PDF |

**Ver postulaciones (requiere token ADMIN):**
- Método: `GET`
- URL: `http://localhost:8090/job-applications`
- Authorization: `Bearer <token_admin>`

---

## 5. API Gateway — Rutas configuradas

```yaml
routes:
  - id: ms-auth
    uri: http://ms-users:8081
    predicates: Path=/auth, /auth/**

  - id: ms-job-applications
    uri: http://ms-users:8081
    predicates: Path=/job-applications, /job-applications/**

  - id: ms-users
    uri: http://ms-users:8081
    predicates: Path=/users, /users/**

  - id: ms-products
    uri: http://ms-products:8082
    predicates: Path=/products, /products/**

  - id: ms-orders
    uri: http://ms-orders:8083
    predicates: Path=/orders, /orders/**
```

---

## 6. Levantar el sistema

```bash
# Primera vez o después de cambios en el código
docker-compose down
docker-compose up --build

# Solo reiniciar un servicio
docker-compose up --build ms-users

# Ver logs en tiempo real
docker-compose logs -f ms-users
```

**Servicios disponibles:**

| Servicio | URL |
|----------|-----|
| Frontend | Abrir `frontend/index.html` en el navegador |
| API Gateway | `http://localhost:8090` |
| ms-users | `http://localhost:8081` |
| ms-products | `http://localhost:8082` |
| ms-orders | `http://localhost:8083` |
| pgAdmin | `http://localhost:5050` |

---
