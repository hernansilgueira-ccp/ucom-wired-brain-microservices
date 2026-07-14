# Wired Brain Apps - Arquitectura de Microservicios

## Descripción del Proyecto

**Wired Brain Apps** es una aplicación de arquitectura de microservicios diseñada para gestionar un sistema de café con productos y control de inventario. La aplicación está compuesta por múltiples servicios que se comunican entre sí en un entorno containerizado con Docker.

### Características principales:
- Arquitectura de microservicios con 4 componentes principales
- Containerización con Docker y Docker Compose
- Monitoreo con Prometheus
- Escalabilidad mediante servicios desacoplados
- API REST para gestión de productos e inventario

---

## Arquitectura y Componentes

### Diagrama de Componentes

```
┌─────────────────────────────────────────────────────────┐
│                    WEB FRONTEND                          │
│  (ASP.NET Core - Puerto 8080)                           │
└──────────────┬──────────────────────────────┬───────────┘
               │                              │
               ▼                              ▼
        ┌──────────────┐             ┌──────────────┐
        │ PRODUCTS-API │             │  STOCK-API   │
        │ (Spring Boot)│             │   (Go Lang)  │
        │ Puerto 8081  │             │ Puerto 8082  │
        └──────┬───────┘             └──────┬───────┘
               │                            │
               └────────────┬───────────────┘
                            │
                            ▼
                    ┌──────────────────┐
                    │  PRODUCTS-DB     │
                    │ (PostgreSQL 11.6)│
                    │   Puerto 5432    │
                    └──────────────────┘
```

---

## Componentes del Sistema

### 1. products-db - Base de Datos PostgreSQL
**Rol:** Almacenamiento centralizado de productos e información

- **Imagen base:** `postgres:11.6-alpine`
- **Puerto expuesto:** 5432
- **Puerto interno:** 5432
- **Características:**
  - Inicialización automática de base de datos
  - Tablas predefinidas con datos de café
  - Volumen persistente de datos

---

### 2. products-api - API de Productos
**Rol:** Servicio REST para gestión de productos

- **Tecnología:** Java Spring Boot 2.4.3
- **Puerto expuesto:** 8081
- **Puerto interno:** 80
- **Características:**
  - API REST completa
  - Documentación OpenAPI/Swagger
  - Métricas Prometheus
  - ORM con JPA/Hibernate
  - Soporta secretos de Docker

---

### 3. stock-api - API de Inventario
**Rol:** Servicio REST para control de stock

- **Tecnología:** Go Lang 1.15.6
- **Puerto expuesto:** 8082
- **Puerto interno:** 8080
- **Características:**
  - API REST ligera y rápida
  - Conexión a PostgreSQL
  - Métricas Prometheus
  - Bajo consumo de recursos

---

### 4. web - Interfaz Web Frontal
**Rol:** Aplicación web para visualizar productos y stock

- **Tecnología:** ASP.NET Core 3.1
- **Puerto expuesto:** 8080
- **Características:**
  - Interfaz web interactiva
  - Integración con APIs
  - Múltiples versiones (v1, v2, v3)
  - CSS personalizable

---

## Guía de Construcción de Imágenes Docker

### Prerrequisitos

```bash
# Verificar que Docker está instalado
docker --version

# Versión mínima recomendada
# Docker: 19.03+
# Docker Compose: 1.25+
```

### Estructura del Proyecto

```
wired-brain-apps/
├── src/
│   ├── db/
│   │   ├── Dockerfile
│   │   └── init-products-db.sh
│   ├── products-api/
│   │   ├── Dockerfile
│   │   ├── pom.xml
│   │   └── src/
│   ├── stock-api/
│   │   ├── Dockerfile
│   │   ├── src/
│   │   └── go.mod
│   └── web/
│       ├── Dockerfile
│       └── src/
```

### Construcción Individual de Imágenes

#### 1. Construcción de Base de Datos (products-db)

```bash
# Cambiar al directorio raíz del proyecto
cd src/

# Construir directamente
docker build -t psdockerrun/products-db:latest db/

# Verificar la imagen
docker images | grep products-db
```

**Proceso de construcción:**
1. Descarga imagen base PostgreSQL 11.6-alpine
2. Copia script de inicialización
3. Script se ejecuta automáticamente al iniciar el contenedor

---

#### 2. Construcción de API de Productos (products-api)

```bash
# Cambiar al directorio de src
cd src/

# Construir directamente
docker build -t psdockerrun/products-api:latest products-api/

# Verificar la imagen
docker images | grep products-api
```

**Proceso de construcción (Multi-stage):**
1. **Stage 1 - Builder (Maven):**
   - Descarga JDK 11 y Maven
   - Descarga dependencias Maven
   - Compila código fuente con Maven
   - Genera JAR ejecutable
   
2. **Stage 2 - Runtime:**
   - Descarga OpenJDK 11 JRE slim
   - Copia JAR compilado
   - Expone puerto 80
   - Inicia la aplicación

---

#### 3. Construcción de API de Stock (stock-api)

```bash
# Cambiar al directorio de src
cd src/

# Construir directamente
docker build -t psdockerrun/stock-api:latest stock-api/

# Verificar la imagen
docker images | grep stock-api
```

**Proceso de construcción (Multi-stage):**
1. **Stage 1 - Builder (Go):**
   - Descarga Go 1.15.6
   - Descarga módulos de dependencias
   - Compila código Go a binario
   
2. **Stage 2 - Runtime:**
   - Descarga Alpine 3.13
   - Copia binario compilado
   - Expone puerto 8080
   - Inicia servidor

---

#### 4. Construcción de Web

```bash
# Cambiar al directorio de src
cd src/

# directamente:
docker build -t psdockerrun/web:latest web/

# Verificar las imágenes
docker images | grep web
```

**Proceso de construcción:**
1. **Stage 1 - Builder (SDK .NET):**
   - Descarga SDK .NET Core 3.1 Alpine
   - Restaura dependencias NuGet
   - Publica aplicación en modo Release
   
2. **Stage 2 - Runtime:**
   - Descarga ASP.NET Core runtime Alpine
   - Configura variables de entorno
   - Copia artefactos compilados
   - Inicia aplicación ASP.NET Core

---

## Variables de Entorno y Secretos

### 1. products-db - PostgreSQL

#### Variables de Entorno

| Variable | Valor | Descripción |
|----------|-------|-------------|
| `POSTGRES_PASSWORD` | `wired` | Contraseña del usuario postgres |
| `POSTGRES_USER` | `postgres` | Usuario por defecto (implícito) |
| `POSTGRES_DB` | `postgres` | Base de datos por defecto |

#### Inicialización
- **Script:** `/docker-entrypoint-initdb.d/init-products-db.sh`
- **Tablas creadas:**
  - `products`: Almacena productos con id, name, price, stock

#### Conexión
- **Host interno:** `products-db`
- **Puerto:** 5432
- **URL de conexión:** `postgres://postgres:wired@products-db:5432/postgres`

#### Ejemplo de uso en otros servicios
```properties
spring.datasource.url=jdbc:postgresql://products-db:5432/postgres
spring.datasource.username=postgres
spring.datasource.password=wired
```

---

### 2. products-api - Spring Boot

#### Variables de Entorno (docker-compose.yml)
```yaml
# Heredadas automáticamente del contenedor
# No se especifican en docker-compose
```

#### Archivo de Configuración: `application.properties`

```properties
# Logging
logging.level.wiredbrain.products=DEBUG

# Actuador y Prometheus
management.endpoints.web.exposure.include=prometheus

# Servidor
server.port=80

# Base de datos
spring.jpa.database=POSTGRESQL
spring.datasource.platform=postgres
spring.datasource.url=jdbc:postgresql://products-db:5432/postgres
spring.datasource.username=postgres
spring.datasource.password=wired
spring.jpa.show-sql=true
spring.jpa.generate-ddl=true
spring.jpa.hibernate.ddl-auto=update

# Soporte para secretos de Docker
spring.config.import=optional:file:/run/secrets/application.properties
```

#### Configuración en Dockerfile

```dockerfile
# Sin ENV explícitas - usa application.properties
```

#### Secretos de Docker Soportados
- Ruta:** `/run/secrets/application.properties`
- **Propósito:** Variables sensibles adicionales

#### Endpoints Disponibles
| Endpoint | Descripción |
|----------|-------------|
| `/products` | Lista todos los productos |
| `/products/{id}` | Obtiene producto por ID |
| `/actuator/prometheus` | Métricas Prometheus |
| `/swagger-ui.html` | Documentación OpenAPI |

#### Puertos
- **Expuesto:** 8081 (desde localhost)
- **Interno:** 80 (en contenedor)

---

### 3. stock-api - Go Lang

#### Variables de Entorno en Dockerfile

```dockerfile
ENV POSTGRES_CONNECTION_STRING="host=products-db port=5432 user=postgres ****** dbname=postgres sslmode=disable"
```

#### Configuración en Tiempo de Ejecución

| Variable | Valor | Descripción |
|----------|-------|-------------|
| `POSTGRES_CONNECTION_STRING` | `host=products-db port=5432 user=postgres password dbname=postgres sslmode=disable` | Cadena de conexión PostgreSQL |

#### Dependencias (go.mod)
```
github.com/gorilla/mux v1.8.0        # Router HTTP
github.com/lib/pq v1.9.0             # Driver PostgreSQL
github.com/prometheus/client_golang v1.9.0  # Métricas Prometheus
```

#### Endpoints Disponibles
| Endpoint | Descripción |
|----------|-------------|
| `/stock` | Información de stock |
| `/metrics` | Métricas Prometheus |

#### Puertos
- **Expuesto:** 8082 (desde localhost)
- **Interno:** 8080 (en contenedor)

#### Conexión a Base de Datos
```
host=products-db
port=5432
user=postgres
password=wired
dbname=postgres
sslmode=disable
```

---

### 4. web - ASP.NET Core

#### Variables de Entorno en Dockerfile

```dockerfile
ENV ProductsApi:Url="http://products-api/products" \
    StockApi:Url="http://stock-api:8080/stock"
```

#### Configuración de Servicios

| Variable | Valor | Descripción |
|----------|-------|-------------|
| `ProductsApi:Url` | `http://products-api/products` | URL de API de productos (red interna) |
| `StockApi:Url` | `http://stock-api:8080/stock` | URL de API de stock (red interna) |

#### Descubrimiento de Servicios
- **Host:** Nombres de servicios en docker-compose
  - `products-api` → Resuelve a IP del contenedor products-api
  - `stock-api` → Resuelve a IP del contenedor stock-api
  
#### Puertos
- **Expuesto:** 8080 (desde localhost)
- **Interno:** 80 (en contenedor)

## Verificación y Testing

### Verificar servicios activos

```bash
# Listar contenedores
docker ps

# Inspeccionar red
docker network inspect

# Ver logs
docker compose logs -f
```

### Testing de APIs

```bash
# Products API
curl http://localhost:8081/products
curl http://localhost:8081/actuator/prometheus

# Stock API
curl http://localhost:8082/stock
curl http://localhost:8082/metrics

# Web
curl http://localhost:8080/

# Acceder en navegador
# http://localhost:8080
```

### Verificar bases de datos

```bash
# Conectar a PostgreSQL
docker exec -it products-db psql -U postgres -d postgres

# Dentro de psql:
\dt                          # Listar tablas
SELECT * FROM products;      # Ver productos
\q                           # Salir
```

---

## Resolución de Problemas

### Errores Comunes

| Error | Causa | Solución |
|-------|-------|----------|
| Puerto ya en uso | Aplicación previa en puerto | `docker-compose down` o cambiar puerto en compose |
| Conexión a BD rechazada | BD no lista | Aguardar con `depends_on` |
| Red no encontrada | Problemas de composición | `docker network prune` y reintentar |
| Imagen no encontrada | Build incompleto | `docker-compose build --no-cache` |

### Limpiar ambiente

```bash
# Eliminar todos los contenedores detenidos
docker container prune

# Eliminar todas las redes no utilizadas
docker network prune

# Eliminar todas las imágenes no etiquetadas
docker image prune

# Limpiar todo (BE CAREFUL)
docker system prune -a
```

---

## Recursos Adicionales

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Spring Boot Docker](https://spring.io/guides/gs/spring-boot-docker/)
- [PostgreSQL Docker](https://hub.docker.com/_/postgres)
- [ASP.NET Core Docker](https://hub.docker.com/_/microsoft-dotnet)
- [Go Docker](https://hub.docker.com/_/golang)

---

## Licencia y Autoría

- **Proyecto:** Wired Brain Apps
- **Componentes:** Multi-stack (Java, Go, .NET Core)
- **Propósito:** Educativo - Arquitectura de Microservicios
- **Año:** 2024

---
