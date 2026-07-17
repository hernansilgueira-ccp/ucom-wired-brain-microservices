# Wired Brain Apps — Microservicios con Docker Compose

Implementación académica de una arquitectura de microservicios para consultar productos y controlar inventario de una cafetería. La solución integra servicios desarrollados en Java, Go y ASP.NET Core, una base PostgreSQL y monitoreo centralizado con Prometheus.

## Arquitectura

```text
Usuario (puerto 8080)
        |
        v
Web ASP.NET Core
   |            |
   v            v
Products API   Stock API
Spring Boot    Go
puerto 8081    puerto 8082
   |            |
   +------v-----+
       PostgreSQL
       red interna

Prometheus (puerto 9090) recopila métricas de Products API, Stock API y Web.
```

## Componentes

| Servicio | Tecnología | Puerto host | Responsabilidad |
|---|---|---:|---|
| `web` | ASP.NET Core 3.1 | 8080 | Interfaz web y consumo de APIs |
| `products-api` | Spring Boot 2.4.3 / Java 11 | 8081 | Consulta de productos y métricas |
| `stock-api` | Go 1.15.6 | 8082 | Consulta y actualización de stock |
| `products-db` | PostgreSQL 11.6 | No publicado | Persistencia en red interna |
| `prometheus` | Prometheus | 9090 | Recolección y consulta de métricas |

## Requisitos

- Windows 10/11, Linux o macOS.
- Docker Desktop o Docker Engine con Docker Compose.
- Git, si se desea clonar el repositorio.

## Ejecución

Desde la raíz del proyecto:

```bash
docker compose config
docker compose up --build -d
docker compose ps
```

El primer arranque descarga imágenes y construye los servicios. La base se considera saludable únicamente después de crear la tabla `products`, incluida la columna `stock` y los datos iniciales.

## Accesos

| Recurso | URL |
|---|---|
| Aplicación web | http://localhost:8080 |
| Products API | http://localhost:8081/products |
| Stock API | http://localhost:8082/stock/1 |
| Métricas Products API | http://localhost:8081/actuator/prometheus |
| Métricas Stock API | http://localhost:8082/metrics |
| Prometheus | http://localhost:9090 |
| Estado de objetivos | http://localhost:9090/targets |

> La API de stock requiere un identificador: `GET /stock/{id}`. La ruta general `/stock` no está implementada.

## Pruebas en PowerShell

### Productos

```powershell
Invoke-RestMethod http://localhost:8081/products |
ConvertTo-Json -Depth 5
```

### Inventario

```powershell
1..3 | ForEach-Object {
    Invoke-RestMethod "http://localhost:8082/stock/$_"
} | ConvertTo-Json
```

### Base de datos

```powershell
docker exec products-db psql -U postgres -d postgres -c "SELECT * FROM products;"
```

### Monitoreo

En Prometheus, ejecutar la consulta:

```promql
up
```

El valor `1` indica que el objetivo está disponible.

## Operación

```bash
# Logs generales
docker compose logs -f

# Logs de un servicio
docker compose logs stock-api --tail=50

# Detener sin eliminar datos
docker compose down

# Detener y eliminar volúmenes del laboratorio
docker compose down -v
```

## Decisiones y correcciones implementadas

- Se creó `docker-compose.yml` para construir y orquestar todo el sistema.
- Se incorporó una red bridge privada para la comunicación por nombre de servicio.
- PostgreSQL no publica el puerto 5432 al equipo anfitrión; solo es accesible dentro de la red Docker.
- Se agregó persistencia mediante volúmenes nombrados.
- Se agregó un healthcheck que valida la estructura y los datos requeridos antes de iniciar las APIs.
- Se normalizan los finales de línea del script de inicialización para permitir su ejecución desde proyectos clonados en Windows.
- Se reemplazó la etiqueta retirada `openjdk:11-jre-slim` por Eclipse Temurin 11 y se actualizó la imagen de Maven.
- Se añadió Prometheus y su configuración de scraping para los tres servicios instrumentados.

## Estructura relevante

```text
.
├── docker-compose.yml
├── monitoring/
│   └── prometheus.yml
├── capturas/
├── src/
│   ├── db/
│   ├── products-api/
│   ├── stock-api/
│   └── web/
└── README.md
```

## Evidencias

Las capturas de validación, ejecución, APIs, base de datos, interfaz web y Prometheus se encuentran en `capturas/`.

## Informe técnico

El informe completo del proyecto se encuentra disponible en:

- [Informe técnico — Wired Brain Apps](docs/Informe_Tecnico_Wired_Brain_Apps.docx)

## Autores
Antonio Aguero
Victor Martinez
Hernan Silgueira
UCOM, Integración de Sistemas I, 2026.
