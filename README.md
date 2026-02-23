[README.md](https://github.com/user-attachments/files/25502067/README.md)
# 🛒 Plataforma Distribuida de Gestión de Pedidos
### Reto Microservicios — Ingeniería de Software

![Java](https://img.shields.io/badge/Java-21-orange?logo=java)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.2-green?logo=springboot)
![Angular](https://img.shields.io/badge/Angular-17+-red?logo=angular)
![Docker](https://img.shields.io/badge/Docker-Compose-blue?logo=docker)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue?logo=postgresql)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-3-orange?logo=rabbitmq)

---

## 📋 Tabla de Contenido

- [Descripción General](#descripción-general)
- [Arquitectura](#arquitectura)
- [Estructura del Repositorio](#estructura-del-repositorio)
- [Tecnologías](#tecnologías)
- [Requisitos Previos](#requisitos-previos)
- [Configuración del Entorno](#configuración-del-entorno)
- [Levantar el Proyecto](#levantar-el-proyecto)
- [Servicios y Puertos](#servicios-y-puertos)
- [Endpoints Principales](#endpoints-principales)
- [Flujos de Negocio](#flujos-de-negocio)
- [Variables de Entorno](#variables-de-entorno)
- [Pruebas con Postman](#pruebas-con-postman)
- [Equipo](#equipo)

---

## 📖 Descripción General

Sistema distribuido basado en arquitectura de microservicios para la gestión de pedidos, catálogo de productos y autenticación de usuarios. Implementa comunicación síncrona mediante REST y asíncrona mediante RabbitMQ, con seguridad basada en JWT y trazabilidad distribuida.

**Funcionalidades principales:**
- Autenticación y autorización por roles (ADMIN / CUSTOMER)
- Gestión de catálogo de productos con control de stock
- Creación, consulta y cancelación de pedidos
- Comunicación por eventos con consistencia eventual
- Trazabilidad distribuida con Correlation ID

---

## 🏗️ Arquitectura

```
Cliente Angular (4200)
        |
        ▼
API Gateway (8080)
  HTTP + Authorization + X-Correlation-Id
        |
   ┌────┴──────────────────┐
   ▼                       ▼                    ▼
Auth Service (8081)   Catalog Service (8082)   Orders Service (8083)
JWT + Roles           Productos + Stock        Pedidos + Estados
   |                       |                        |
   ▼                       ▼                        ▼
PostgreSQL AuthDB     PostgreSQL CatalogDB     PostgreSQL OrdersDB
(5432)                (5433)                   (5434)
                            ▲
                            |  (consume eventos)
                       RabbitMQ (5672)
                   order.created / order.cancelled
                            ▲
                            | (publica eventos)
                       Orders Service
```

### Capas de la Arquitectura

| Capa | Componentes | Responsabilidad |
|------|------------|-----------------|
| Application Layer | Gateway, Auth, Catalog, Orders | Lógica de negocio y APIs REST |
| Messaging Layer | RabbitMQ | Comunicación asíncrona por eventos |
| Infrastructure Layer | PostgreSQL (x3) | Persistencia independiente por servicio |

---

## 📁 Estructura del Repositorio

```
reto-microservicios/
├── README.md
├── .env.example
├── docker-compose.yml
├── docs/
│   ├── arquitectura.md
│   ├── flujos.md
│   └── postman-collection.json
└── services/
    ├── api-gateway/
    │   ├── README.md
    │   ├── pom.xml
    │   └── src/main/
    │       ├── java/com/reto/gateway/
    │       │   ├── ApiGatewayApplication.java
    │       │   └── security/
    │       │       └── JwtGatewayFilter.java
    │       └── resources/
    │           └── application.yml
    ├── auth-service/
    │   ├── README.md
    │   ├── pom.xml
    │   └── src/main/
    │       ├── java/com/reto/auth/
    │       │   ├── AuthServiceApplication.java
    │       │   ├── config/
    │       │   ├── controller/
    │       │   ├── dto/
    │       │   ├── entity/
    │       │   ├── repository/
    │       │   ├── security/
    │       │   └── service/
    │       └── resources/
    │           └── application.yml
    ├── catalog-service/
    │   ├── README.md
    │   ├── pom.xml
    │   └── src/main/
    │       ├── java/com/reto/catalog/
    │       └── resources/
    │           └── application.yml
    └── order-service/
        ├── README.md
        ├── pom.xml
        └── src/main/
            ├── java/com/reto/order/
            └── resources/
                └── application.yml
```

---

## 🛠️ Tecnologías

| Capa | Tecnología |
|------|-----------|
| Backend | Spring Boot 3.2 + Java 21 |
| Seguridad | Spring Security + JWT |
| Base de Datos | PostgreSQL 16 |
| Mensajería | RabbitMQ 3 |
| Frontend | Angular 17+ |
| Infraestructura | Docker + Docker Compose |
| Gateway | Spring Cloud Gateway |

---

## ✅ Requisitos Previos

Antes de comenzar, asegúrate de tener instalado:

- **Java 21** (LTS)
- **Maven 3.9+**
- **Docker Desktop**
- **Node.js 18+** y **Angular CLI** (para frontend)
- **Postman** (para pruebas)
- **IDE**: IntelliJ IDEA o VS Code

Verificar desde consola:
```bash
java --version       # debe mostrar openjdk 21
mvn --version
docker --version
docker compose version
node --version
ng version
```

---

## ⚙️ Configuración del Entorno

1. Clonar el repositorio:
```bash
git clone https://github.com/<tu-usuario>/reto-microservicios.git
cd reto-microservicios
```

2. Copiar el archivo de variables de entorno:
```bash
cp .env.example .env
```

3. Editar el archivo `.env` con tus valores (ver sección [Variables de Entorno](#variables-de-entorno)).

---

## 🚀 Levantar el Proyecto

### 1. Levantar infraestructura (PostgreSQL + RabbitMQ)

```bash
docker compose up -d
```

Verificar que los contenedores estén corriendo:
```bash
docker ps
```

### 2. Levantar microservicios (en terminales separadas)

```bash
# Auth Service
cd services/auth-service
mvn spring-boot:run

# Catalog Service
cd services/catalog-service
mvn spring-boot:run

# Orders Service
cd services/order-service
mvn spring-boot:run

# API Gateway
cd services/api-gateway
mvn spring-boot:run
```

### 3. Levantar Frontend Angular

```bash
cd frontend
npm install
ng serve
```

---

## 🌐 Servicios y Puertos

| Servicio | Puerto | URL |
|----------|--------|-----|
| Angular Frontend | 4200 | http://localhost:4200 |
| API Gateway | 8080 | http://localhost:8080 |
| Auth Service | 8081 | http://localhost:8081 |
| Catalog Service | 8082 | http://localhost:8082 |
| Orders Service | 8083 | http://localhost:8083 |
| PostgreSQL AuthDB | 5432 | localhost:5432 |
| PostgreSQL CatalogDB | 5433 | localhost:5433 |
| PostgreSQL OrdersDB | 5434 | localhost:5434 |
| RabbitMQ AMQP | 5672 | localhost:5672 |
| RabbitMQ Management UI | 15672 | http://localhost:15672 |

> **RabbitMQ UI:** usuario `reto` / contraseña `reto`

---

## 📡 Endpoints Principales

Todos los endpoints se consumen a través del **API Gateway en el puerto 8080**.

### Auth Service

| Método | Endpoint | Descripción | Auth requerida |
|--------|----------|-------------|----------------|
| POST | `/auth/register` | Registrar usuario | No |
| POST | `/auth/login` | Iniciar sesión, retorna JWT | No |

### Catalog Service

| Método | Endpoint | Descripción | Auth requerida |
|--------|----------|-------------|----------------|
| GET | `/catalog/products` | Listar productos | Sí |
| GET | `/catalog/products/{id}` | Detalle de producto | Sí |
| POST | `/catalog/products` | Crear producto | Sí (ADMIN) |
| GET | `/catalog/check-stock` | Validar stock disponible | Sí |

### Orders Service

| Método | Endpoint | Descripción | Auth requerida |
|--------|----------|-------------|----------------|
| POST | `/orders` | Crear pedido | Sí (CUSTOMER) |
| GET | `/orders` | Listar pedidos del usuario | Sí |
| GET | `/orders/{id}` | Detalle de pedido | Sí |
| PUT | `/orders/{id}/cancel` | Cancelar pedido | Sí |

### Health Checks

| Servicio | Endpoint |
|----------|----------|
| API Gateway | http://localhost:8080/actuator/health |
| Auth Service | http://localhost:8081/actuator/health |
| Catalog Service | http://localhost:8082/actuator/health |
| Orders Service | http://localhost:8083/actuator/health |

---

## 🔄 Flujos de Negocio

### Flujo 1 — Pedido Exitoso

```
1. Usuario hace login → recibe JWT
2. Usuario envía POST /orders con Authorization: Bearer <token>
3. Gateway valida JWT y propaga X-Correlation-Id
4. Orders llama a GET /catalog/check-stock (REST síncrono)
5. Catalog responde { "available": true }
6. Orders persiste el pedido con estado CREATED
7. Orders publica evento order.created en RabbitMQ (con eventId + correlationId)
8. Catalog consume el evento y descuenta stock (idempotente)
```

### Flujo 2 — Cancelación de Pedido

```
1. Usuario envía PUT /orders/{id}/cancel
2. Orders valida que el estado sea CREATED
3. Orders actualiza estado a CANCELLED
4. Orders publica evento order.cancelled en RabbitMQ
5. Catalog consume el evento y repone el stock
```

### Flujo 3 — Sin Stock (409 Conflict)

```
1. Usuario envía POST /orders
2. Orders llama a /catalog/check-stock
3. Catalog responde { "available": false }
4. Orders retorna 409 Conflict
5. NO se persiste pedido, NO se publica evento
```

---

## 🔐 Variables de Entorno

Copiar `.env.example` a `.env` y configurar los siguientes valores:

```env
# JWT
JWT_SECRET=CAMBIAR_ESTA_CLAVE_SUPER_LARGA_DE_32_CHARS_MINIMO
JWT_EXPIRATION_MINUTES=15

# PostgreSQL - Auth
AUTH_DB_URL=jdbc:postgresql://localhost:5432/authdb
AUTH_DB_USER=reto
AUTH_DB_PASSWORD=reto

# PostgreSQL - Catalog
CATALOG_DB_URL=jdbc:postgresql://localhost:5433/catalogdb
CATALOG_DB_USER=reto
CATALOG_DB_PASSWORD=reto

# PostgreSQL - Orders
ORDERS_DB_URL=jdbc:postgresql://localhost:5434/orderdb
ORDERS_DB_USER=reto
ORDERS_DB_PASSWORD=reto

# RabbitMQ
RABBITMQ_HOST=localhost
RABBITMQ_PORT=5672
RABBITMQ_USER=reto
RABBITMQ_PASSWORD=reto
```

> ⚠️ **Nunca subas el archivo `.env` con valores reales al repositorio.** El `.gitignore` ya lo excluye.

---

## 🧪 Pruebas con Postman

La colección de Postman está disponible en `docs/postman-collection.json`.

Para importarla:
1. Abrir Postman
2. Click en **Import**
3. Seleccionar el archivo `docs/postman-collection.json`

### Flujo de prueba mínimo (Sprint 1)

```
1. POST /auth/register       → crear usuario
2. POST /auth/login          → obtener token JWT
3. GET  /catalog/ping        → verificar routing (sin token → 401)
4. GET  /orders/ping         → con token válido → 200
5. GET  /orders/ping         → con token inválido → 401
```

---

## 👥 Equipo

| Nombre | Rol | GitHub |
|--------|-----|--------|
| Nombre 1 | Auth Service + Gateway | @usuario1 |
| Nombre 2 | Catalog Service | @usuario2 |
| Nombre 3 | Orders Service + RabbitMQ | @usuario3 |
| Nombre 4 | Frontend Angular | @usuario4 |
| Nombre 5 | Documentación + DevOps | @usuario5 |

---

## 📌 Estado del Proyecto

| Sprint | Estado | Descripción |
|--------|--------|-------------|
| Sprint 1 | ✅ Completado | Infraestructura base + JWT + Gateway |
| Sprint 2 | 🔄 En progreso | Catálogo, Pedidos y Eventos RabbitMQ |
| Sprint 3 | ⏳ Pendiente | Cancelación, Observabilidad y Frontend |

---

> **Autora del reto:** Gloria Gutiérrez — Ingeniería de Software
