# PupaApp - Sistema Distribuido de Venta de Pulpas Naturales
---
# Descripción del Proyecto

PupaApp es un sistema distribuido basado en microservicios que permite la gestión de productos, usuarios y pedidos para la venta de pulpas naturales.

El sistema fue diseñado bajo una arquitectura desacoplada, donde cada microservicio cumple una responsabilidad específica y se comunica con otros mediante APIs REST. Esto permite escalabilidad, mantenimiento sencillo y una estructura alineada con sistemas reales de producción.

---
## Arquitectura del Sistema
El sistema está compuesto por múltiples microservicios independientes, cada uno ejecutándose en su propio contenedor Docker.
### Microservicios

| Microservicio | Puerto | Descripción |
|--------------|--------|------------|
| ms-products  | 8082   | Gestión del catálogo de productos |
| ms-orders    | 8083   | Gestión de pedidos |
| ms-users     | 8081   | Gestión de usuarios |
| PostgreSQL   | 5432   | Base de datos principal |
| pgAdmin      | 5050   | Administración de base de datos |

---
### 🔗 Comunicación entre servicios
Los microservicios se comunican mediante HTTP usando nombres internos de Docker:
 - http://ms-products:8082
 - http://ms-orders:8083
 - http://ms-users:8081
Dentro de Docker no se utiliza `localhost`, ya que cada servicio corre en su propio contenedor.

---
### Despliegue con Docker
El sistema se ejecuta mediante `docker-compose`, lo que permite levantar todos los servicios de forma coordinada.

---
### Ejecutar el proyecto
docker compose up --build

---
### Ver estado de los contenedores
docker compose ps

---
# Acceso a los servicios

- Productos → http://localhost:8082/products
- Pedidos → http://localhost:8083/orders
- Usuarios → http://localhost:8081/users
- pgAdmin → http://localhost:5050

---
# Endpoints principales
### Productos (ms-products)
 - GET    /products
 - POST   /products
 - PUT    /products/{id}
### Pedidos (ms-orders)
 - POST   /orders
 - GET    /orders
###  Usuarios (ms-users)
 - GET    /users
 - POST   /users

---

# Relación Historia De Usuario Se pueden encontrar en nuestra emsa de trabajo con sus respectivas capturas de pantalla : 
   - [Mesa De Trabajo](https://julianguerra1231186-1773894024267.atlassian.net/?continue=https%3A%2F%2Fjulianguerra1231186-1773894024267.atlassian.net%2Fwelcome%2Fsoftware%3FprojectId%3D10000&atlOrigin=eyJpIjoiOTdhMWY4ZGU5N2YwNDQ0MDk3NTZjODkxYTU5ZWVlZWQiLCJwIjoiamlyYS1zb2Z0d2FyZSJ9)
     
---

# Base de Datos
- Motor: PostgreSQL
- Base de datos: pulpapp_db
- ORM: JPA / Hibernate
- Creación automática de tablas
# Flujo del Sistema
- Se crea un producto
- Se consulta catálogo
- Se registra pedido
- ms-orders consulta ms-products
- Se almacena el pedido
# Problemas enfrentados
- Fallos de conexión (UnknownHostException)
- Error socket hang up
- Problemas de Docker (red, puertos)
- Conflictos por merge
- Pérdida de Dockerfile
- Uso incorrecto de localhost
- Inconsistencias en endpoints
# Soluciones aplicadas
- Configuración de red Docker compartida
- Uso de depends_on + healthcheck
- Uso de nombres de servicio (ms-products)
- Restauración de archivos tras merge
- Corrección de endpoints y controladores
# Seguridad
- Validación de datos en backend
- Manejo controlado de errores
- Aislamiento con Docker
- Preparación para JWT
# Estado del Proyecto
✔ Arquitectura distribuida funcional
✔ Microservicios independientes
✔ Comunicación entre servicios
✔ CRUD completo de productos
✔ Gestión de pedidos
✔ Validaciones implementadas
✔ Sistema dockerizado

 
