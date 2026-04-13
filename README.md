# Frontend
## ¿Qué hace?

Es la interfaz visual del sistema. El usuario navega por aquí para ver productos, registrarse, iniciar sesión, agregar productos al carrito y finalizar compras. También incluye el panel de administrador y la página de reclutamiento.

## ¿Cómo funciona?

Es una aplicación web en HTML, CSS y JavaScript. Se comunica con el backend a través del **API Gateway** usando `fetch()`. El token JWT se guarda en `localStorage` y se envía en cada petición protegida.

## Páginas disponibles

| Archivo | Descripción |
|---------|-------------|
| index.html | Página principal con slider, carrusel de productos y sección "¿Quiénes somos?" |
| catalog.html | Catálogo de productos conectado a ms-products |
| cart.html | Carrito de compras y checkout |
| login.html | Inicio de sesión con email y contraseña |
| register.html | Registro de nuevos usuarios |
| dashboard.html | Panel del usuario autenticado (diferente según el rol) |
| trabaja-con-nosotros.html | Página de reclutamiento con formulario de postulación |

## Archivos JavaScript principales

| Archivo | Descripción |
|---------|-------------|
| services.js | Centraliza todas las llamadas al API (productos, usuarios, pedidos, auth) |
| auth.js | Gestiona el token JWT: guardar, leer, verificar expiración, actualizar navbar |
| app.js | Lógica principal de cada página (carrito, catálogo, checkout) |

## ¿Cómo se conecta con el backend?

Todo pasa por el **API Gateway** en `http://localhost:8090`. El frontend nunca llama directamente a los microservicios.

```
Frontend → http://localhost:8090/auth/login       → ms-users
Frontend → http://localhost:8090/products         → ms-products
Frontend → http://localhost:8090/orders           → ms-orders
Frontend → http://localhost:8090/job-applications → ms-users
```

## Dashboard según rol

**ROLE_SELLER ve:**
- Su perfil
- Sus pedidos
- Enlace al catálogo

**ROLE_ADMIN ve:**
- Su perfil
- Gestión de productos (crear, editar, eliminar)
- Lista de clientes
- Postulaciones laborales con descarga de CVs

## Navbar dinámico

- Sin sesión → muestra "Iniciar sesión"
- Con sesión → muestra el nombre del usuario y botón "Cerrar sesión"
- Al cerrar sesión → limpia el localStorage y redirige al login

## Tecnologías usadas

HTML5 · CSS3 · JavaScript vanilla · localStorage · Fetch API
