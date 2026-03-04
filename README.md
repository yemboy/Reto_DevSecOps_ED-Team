# Reto DevSecOps - ED Team

APIs en Flask (Python) y Go con pipeline DevSecOps completo, despliegue automatizado en Kubernetes (Azure AKS o AWS EKS) y escaneo de seguridad integrado.

## ¿Qué hace?

Dos APIs con los mismos endpoints:

| Endpoint | Respuesta |
|----------|-----------|
| `GET /` | `{"message": "Hola ED Team"}` |
| `GET /health` | `{"status": "healthy"}` |

| App | Lenguaje | Puerto | Directorio |
|-----|----------|--------|------------|
| Python (Flask) | Python 3.13 | 5000 | `src/` |
| Go | Go 1.23 | 8080 | `src-go/` |

## Estructura del proyecto

```
src/
  app.py              # Aplicación Flask (Python)
  Dockerfile          # Imagen Docker (Python 3.13 + Gunicorn)
  requirements.txt    # Dependencias (Flask, Gunicorn)
src-go/
  main.go             # Aplicación Go
  main_test.go        # Tests unitarios
  go.mod              # Módulo Go
  Dockerfile          # Imagen Docker (Go 1.23, multi-stage)
k8s/
  deployment.yml      # Deployment con 2 réplicas, probes y security context
  service.yml         # Service ClusterIP (puerto 80 → 5000)
  ingress.yml         # Ingress con Traefik
  hpa.yml             # Autoescalado (2-10 pods, CPU 70% / RAM 80%)
.github/workflows/
  ci.yml              # Pipeline CI Python (SAST, SCA, escaneo de imagen)
  ci-go.yml           # Pipeline CI Go (tests, SAST, SCA, escaneo de imagen)
  cd.yml              # Pipeline CD (build, release, deploy)
  infraestructure.yml # Provisión de infraestructura (AKS/EKS)
```

## Pipelines

### CI Python (`ci.yml`) — Se ejecuta en push/PR a `main` (cambios en `src/`)
1. **SAST** — Análisis estático con CodeQL (Python)
2. **SCA** — Revisión de dependencias (solo en PRs)
3. **Container Security** — Escaneo de imagen con Grype/Anchore

### CI Go (`ci-go.yml`) — Se ejecuta en push/PR a `main` (cambios en `src-go/`)
1. **Test** — Tests unitarios con `go test -race`
2. **SAST** — Análisis estático con CodeQL (Go)
3. **SCA** — Revisión de dependencias (solo en PRs)
4. **Container Security** — Escaneo de imagen con Grype/Anchore

### CD (`cd.yml`) — Se ejecuta en push a `main`
1. **Build** — Construye la imagen Docker y la sube al registry (ACR o ECR)
2. **Release** — Crea un GitHub Release con tag automático
3. **Deploy** — Despliega en Kubernetes (AKS o EKS)

### Infraestructura (`infraestructure.yml`) — Ejecución manual
Crea o destruye la infraestructura cloud (registry + cluster Kubernetes).

## Configuración de secretos y variables

Configura todo en **GitHub → Settings → Secrets and variables → Actions**.

### Secrets (obligatorios)

#### Para Azure:
| Secret | Descripción |
|--------|-------------|
| `AZURE_CREDENTIALS` | JSON con las credenciales del Service Principal de Azure |

#### Para AWS:
| Secret | Descripción |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | Access Key ID de IAM |
| `AWS_SECRET_ACCESS_KEY` | Secret Access Key de IAM |

> `GITHUB_TOKEN` se genera automáticamente, no necesitas configurarlo.

### Variables de repositorio (obligatorias)

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `CLOUD_PROVIDER` | Proveedor cloud a usar | `azure` o `aws` |

#### Azure:
| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `RESOURCE_GROUP` | Resource Group existente | `rg-edteam` |
| `AKS_NAME` | Nombre del cluster AKS | `aks-edteam` |
| `CONTAINER_NAME` | Nombre de la imagen | `ed-team-app` |
| `REGISTRY_NAME` | Nombre del ACR | `acredteamdevsecops` |
| `AZURE_LOCATION` | Región de Azure | `eastus` |

#### AWS:
| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `AWS_REGION` | Región de AWS | `us-east-1` |
| `EKS_CLUSTER_NAME` | Nombre del cluster EKS | `eks-edteam` |
| `ECR_REPOSITORY_NAME` | Nombre del repositorio ECR | `ed-team-app` |

## Ejecución local

### Requisitos previos

| Herramienta | macOS | Windows |
|-------------|-------|---------|
| Python 3.x | `brew install python` | [python.org/downloads](https://www.python.org/downloads/) (marcar "Add to PATH") |
| pip | Incluido con Python | Incluido con Python |
| Go 1.23+ | `brew install go` | [go.dev/dl](https://go.dev/dl/) |
| Docker | [Docker Desktop for Mac](https://docs.docker.com/desktop/install/mac-install/) | [Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/) |
| Git | `brew install git` | [git-scm.com](https://git-scm.com/download/win) |

### Con Python

**macOS / Linux (Terminal):**
```bash
cd src
pip3 install -r requirements.txt
python3 app.py
# Disponible en http://localhost:5000
```

**Windows (PowerShell o CMD):**
```powershell
cd src
pip install -r requirements.txt
python app.py
# Disponible en http://localhost:5000
```

> **Nota Windows:** Si `python` no es reconocido, usa `py` en su lugar:
> ```powershell
> py app.py
> ```

### Con Go

```bash
cd src-go
go run main.go
# Disponible en http://localhost:8080
```

Tests:
```bash
cd src-go
go test -v ./...
```

### Con Docker (multiplataforma)

Funciona igual en macOS, Windows y Linux (requiere Docker Desktop):

**Python:**
```bash
docker build -t ed-team-app src/
docker run -p 5000:5000 ed-team-app
# Disponible en http://localhost:5000
```

**Go:**
```bash
docker build -t ed-team-app-go src-go/
docker run -p 8080:8080 ed-team-app-go
# Disponible en http://localhost:8080
```
