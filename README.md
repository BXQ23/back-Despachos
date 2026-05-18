# Back Despachos — Springboot API REST

Microservicio de gestión de **despachos** para Innovatech Chile.  
Spring Boot 3.4.4 · Java 17 · MySQL 8 · Docker · GitHub Actions CI/CD

---

## Tabla de contenidos

1. [Descripción del servicio](#descripción-del-servicio)
2. [Requisitos previos](#requisitos-previos)
3. [Variables de entorno](#variables-de-entorno)
4. [Ejecutar con Docker Compose](#ejecutar-con-docker-compose)
5. [Endpoints disponibles](#endpoints-disponibles)
6. [Dockerfile (multi-stage)](#dockerfile-multi-stage)
7. [Pipeline CI/CD](#pipeline-cicd)
8. [Secrets de GitHub Actions](#secrets-de-github-actions)

---

## Descripción del servicio

Expone una API REST para crear, listar, actualizar y eliminar **despachos**.  
Se conecta a la misma instancia MySQL del stack del backend, a la base de datos `despachos_db`.

**Puerto:** `8081`  
**Base path:** `/api/v1/despachos`  
**Swagger UI:** `http://<HOST>:8081/swagger-ui.html`

> Este servicio se despliega **junto a back-ventas** en el mismo EC2 backend,  
> usando el `docker-compose.yml` ubicado en el repositorio `back-ventas`.

---

## Requisitos previos

| Herramienta | Versión mínima |
|---|---|
| Docker | 24.x |
| Docker Compose | 2.x |
| Git | 2.x |

---

## Variables de entorno

| Variable | Descripción | Ejemplo |
|---|---|---|
| `DB_ENDPOINT` | Hostname del servidor MySQL | `mysql` |
| `DB_PORT` | Puerto MySQL | `3306` |
| `DB_NAME` | Nombre de la base de datos | `despachos_db` |
| `DB_USERNAME` | Usuario MySQL | `root` |
| `DB_PASSWORD` | Contraseña MySQL | `*****` |

---

## Ejecutar con Docker Compose

Este servicio forma parte del stack del backend. Para levantarlo:

```bash
# Desde el repositorio back-ventas (donde vive el docker-compose.yml del stack):
docker compose up -d back-despachos

# Ver logs
docker compose logs -f back-despachos

# Verificar que responde
curl http://localhost:8081/api/v1/despachos
```

Para ejecutar solo este servicio de forma aislada (desarrollo local):

```bash
# Desde este repositorio, construir la imagen localmente
docker build -t back-despachos:local .

# Correr apuntando a una MySQL local
docker run -p 8081:8081 \
  -e DB_ENDPOINT=host.docker.internal \
  -e DB_PORT=3306 \
  -e DB_NAME=despachos_db \
  -e DB_USERNAME=root \
  -e DB_PASSWORD=tu_password \
  back-despachos:local
```

---

## Endpoints disponibles

| Método | Ruta | Descripción |
|---|---|---|
| `GET` | `/api/v1/despachos` | Listar todos los despachos |
| `GET` | `/api/v1/despachos/{id}` | Obtener despacho por ID |
| `POST` | `/api/v1/despachos` | Crear nuevo despacho |
| `PUT` | `/api/v1/despachos/{id}` | Actualizar despacho |
| `DELETE` | `/api/v1/despachos/{id}` | Eliminar despacho |

Documentación interactiva: `http://<HOST>:8081/swagger-ui.html`

---

## Dockerfile (multi-stage)

Mismo patrón que back-ventas, adaptado al proyecto Despachos:

```
Stage 1 (builder)  →  maven:3.9.6-eclipse-temurin-17
  - Compila Springboot-API-REST-DESPACHO

Stage 2 (runtime)  →  eclipse-temurin:17-jre-alpine
  - Solo JRE, usuario appuser (no root)
  - Puerto 8081
```

---

## Pipeline CI/CD

```
Push a rama deploy
        │
        ▼
┌─────────────────────────┐
│  JOB 1: build-push      │
│  Build imagen Docker    │
│  Push a Docker Hub      │
│  :latest + :sha-XXXXXX  │
└──────────┬──────────────┘
           │
           ▼
┌──────────────────────────┐
│  JOB 2: deploy           │
│  SSH a EC2 Backend       │
│  docker compose pull     │
│  compose up --no-deps    │
└──────────────────────────┘
```

---

## Secrets de GitHub Actions

Los mismos secrets que back-ventas (mismo EC2):

| Secret | Descripción |
|---|---|
| `DOCKERHUB_USERNAME` | Usuario Docker Hub |
| `DOCKERHUB_TOKEN` | Token de acceso Docker Hub |
| `EC2_BACKEND_HOST` | IP pública EC2 backend |
| `EC2_USERNAME` | Usuario SSH (`ec2-user` / `ubuntu`) |
| `EC2_SSH_KEY` | Clave privada `.pem` completa |

---

## Estructura del repositorio

```
back-despachos/
├── .github/
│   └── workflows/
│       └── ci-cd.yml                    # Pipeline CI/CD
├── Springboot-API-REST-DESPACHO/
│   ├── src/                             # Código fuente Java
│   ├── pom.xml
│   └── ...
├── Dockerfile                           # Multi-stage build
├── .gitignore
└── README.md
```
