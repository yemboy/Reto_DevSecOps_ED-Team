# Reto Técnico DevSecOps
Implementación completa de flujo DevSecOps para el reto técnico de ED Team. Incluye aplicación Flask, pipelines CI/CD automatizados, escaneo de seguridad y despliegue multi-cloud (Azure / AWS).

**Funcionalidad**: API que responde `{"message": "Hola ED Team"}` en el endpoint raíz.

## Arquitectura Multi-Cloud

| Componente | Azure | AWS |
|------------|-------|-----|
| **Container Registry** | ACR | ECR |
| **Kubernetes** | AKS | EKS |
| **Ingress** | Traefik | Traefik |
| **CI/CD** | GitHub Actions | GitHub Actions |
| **Security** | Dependency Review + Grype + CodeQL | Dependency Review + Grype + CodeQL |

# Diagramas

## Arquitectura de Infraestructura

```mermaid
graph TB
    subgraph subGraph0["Traefik Namespace"]
            IC["Traefik Ingress Controller"]
            LB["Load Balancer"]
    end
    subgraph subGraph1["ED Team App"]
            P1["Pod 1<br>ed-team-app"]
    end
    subgraph subGraph2["Default Namespace"]
            subGraph1
            SVC["Service<br>ClusterIP"]
            HPA["HPA<br>2-10 replicas<br>📈 Can scale up"]
            ING["Ingress<br>ed-team-app"]
            P2["Pod 2<br>📈 On demand"]
            P3["Pod 3<br>📈 On demand"]
    end
    subgraph subGraph3["Kubernetes Cluster<br>AKS / EKS"]
            subGraph0
            subGraph2
    end
    subgraph subGraph4["Cloud Provider<br>Azure / AWS"]
            subGraph3
            REG["Container Registry<br>ACR / ECR"]
    end
    subgraph External["External"]
            USER["👤 User"]
            POSTMAN["📱 Postman"]
    end
    subgraph GitHub["GitHub"]
            REPO["📁 Repository"]
            ACTIONS["⚙️ GitHub Actions"]
            RELEASES["📦 Releases"]
    end
        USER --> LB
        POSTMAN --> LB
        LB --> IC
        IC --> ING
        ING --> SVC
        SVC --> P1
        HPA -.-> P1
        HPA -. Can create .-> P2 & P3
        ACTIONS --> RELEASES & REG
        REG --> P1

        style IC fill:#f3e5f5
        style LB fill:#f3e5f5
        style P1 fill:#e8f5e8
        style HPA fill:#fff3e0
        style P2 fill:#fff3e0,stroke-dasharray: 5 5
        style REG fill:#e1f5fe
        style P3 fill:#fff3e0,stroke-dasharray: 5 5
```

## Flujo CI/CD Completo

```mermaid
graph TB
    subgraph "👨‍💻 Developer Workflow"
        DEV[Developer]
        BRANCH[Feature Branch]
        PR[Pull Request]
    end

    subgraph "🔒 CI Pipeline (Pull Request)"
        SEC[Security Workflow]
        DEP_REV[Dependency Review <br/> 📊 Python packages]
        TRIVY[Grype Scan <br/> 🔍 Container vulnerabilities]
        BUILD_TEST[Build Test <br/> 🔨 Docker build validation]
    end

    subgraph "🚀 CD Pipeline (Push to Main)"
        CD[CD Workflow]
        BUILD[Build Image <br/> 🏗️ Docker build]
        PUSH[Push to Registry <br/> 📤 ACR / ECR]
        REL[Create Release <br/> 📦 GitHub release]
        DEPLOY[Deploy to K8s <br/> ☸️ AKS / EKS]
        VALIDATE[Health Validation <br/> ✅ App testing]
    end

    subgraph "☁️ Cloud Infrastructure"
        REG_INFRA[Container Registry<br/>ACR / ECR]
        K8S_INFRA[Kubernetes Cluster<br/>AKS / EKS]
        PODS[Running Pod 🏃‍♂️ <br/> ed-team-app]
        LB_INFRA[Load Balancer <br/> 🌐 Public endpoint]
    end

    subgraph "🧪 Testing & Validation"
        HEALTH[Health Check ❤️ <br/> /health endpoint]
        APP_TEST[App Test 📝 <br/> 'Hola ED Team']
        POSTMAN_TEST[Postman Testing 📱 <br/> Manual validation]
    end

    DEV --> BRANCH
    BRANCH --> PR
    PR --> SEC
    SEC --> DEP_REV
    SEC --> TRIVY
    SEC --> BUILD_TEST
    DEP_REV --> MERGE{✅ All Checks Pass?}
    TRIVY --> MERGE
    BUILD_TEST --> MERGE
    MERGE -->|Merge to Main| CD
    CD --> BUILD
    BUILD --> PUSH
    PUSH --> REL
    REL --> DEPLOY
    DEPLOY --> VALIDATE
    PUSH --> REG_INFRA
    DEPLOY --> K8S_INFRA
    K8S_INFRA --> PODS
    PODS --> LB_INFRA
    VALIDATE --> HEALTH
    VALIDATE --> APP_TEST
    LB_INFRA --> POSTMAN_TEST

    style SEC fill:#ffebee
    style CD fill:#e8f5e8
    style REG_INFRA fill:#e1f5fe
    style K8S_INFRA fill:#e1f5fe
    style PODS fill:#f1f8e9
    style HEALTH fill:#e8f5e8
    style APP_TEST fill:#e8f5e8
    style POSTMAN_TEST fill:#fff3e0
```

## Workflow de Infraestructura (Manual)

```mermaid
graph LR
    subgraph "🎯 Manual Trigger"
        ADMIN[👨‍💻 Admin]
        DISPATCH[Workflow Dispatch<br/>🔘 Select provider + action]
    end

    subgraph "🏗️ Azure Path"
        AZ_CREATE[Create ACR + AKS<br/>+ Traefik]
        AZ_DESTROY[Delete ACR + AKS]
    end

    subgraph "🏗️ AWS Path"
        AWS_CREATE[Create ECR + EKS<br/>+ Traefik]
        AWS_DESTROY[Delete ECR + EKS]
    end

    ADMIN --> DISPATCH
    DISPATCH -->|azure + create| AZ_CREATE
    DISPATCH -->|azure + destroy| AZ_DESTROY
    DISPATCH -->|aws + create| AWS_CREATE
    DISPATCH -->|aws + destroy| AWS_DESTROY

    style AZ_CREATE fill:#e1f5fe
    style AWS_CREATE fill:#fff3e0
    style AZ_DESTROY fill:#ffebee
    style AWS_DESTROY fill:#ffebee
```

## 📁 Estructura del Proyecto

```
├── .github/workflows/
│   ├── ci.yml              # Escaneo de seguridad en PRs
│   ├── cd.yml              # Pipeline de deployment (multi-cloud)
│   └── infraestructure.yml # Provisioning de infraestructura (Azure / AWS)
├── src/
│   ├── app.py              # Aplicación Flask
│   ├── requirements.txt    # Dependencias Python
│   └── Dockerfile          # Imagen de contenedor
├── k8s/
│   ├── deployment.yml      # Pods de la aplicación
│   ├── service.yml         # Servicio interno
│   ├── hpa.yml             # Auto-scaling
│   └── ingress.yml         # Acceso externo (Traefik)
└── README.md
```

## ⚙️ Setup e Instalación

### 1. **Configurar Secretos en GitHub**

Settings → Secrets → Actions:

**Para Azure:**
```
AZURE_CREDENTIALS = { "clientId": "...", "clientSecret": "...", "subscriptionId": "...", "tenantId": "..." }
```

**Para AWS:**
```
AWS_ACCESS_KEY_ID = tu-access-key
AWS_SECRET_ACCESS_KEY = tu-secret-key
```

### 2. **Configurar Variables en GitHub**

Settings → Variables → Actions:

| Variable | Azure | AWS |
|----------|-------|-----|
| `CLOUD_PROVIDER` | `azure` | `aws` |
| `RESOURCE_GROUP` | `mi-resource-group` | - |
| `AKS_NAME` | `mi-aks-cluster` | - |
| `CONTAINER_NAME` | `ed-team-app` | - |
| `REGISTRY_NAME` | `acredteamdevsecops` | - |
| `AZURE_LOCATION` | `eastus` | - |
| `AWS_REGION` | - | `us-east-1` |
| `EKS_CLUSTER_NAME` | - | `eks-ed-team-cluster` |
| `ECR_REPOSITORY_NAME` | - | `ed-team-app` |

### 3. **Crear Infraestructura**

1. Actions → **Provision Infrastructure**
2. Seleccionar **Cloud Provider**: `azure` o `aws`
3. Seleccionar **Action**: `create`
4. Run workflow

### 4. **Activar Branch Protection**

Settings → Rules → New Ruleset:
- Require pull request reviews (1)
- Restrict deletions
- Block force pushes

## Workflows

### **CI** (Pull Requests)
- **CodeQL SAST**: Análisis estático de seguridad
- **Dependency Review**: Vulnerabilidades en dependencias
- **Grype Scan**: Vulnerabilidades en contenedores

### **CD** (Push a main)
1. **Build**: Construye y sube imagen a ACR/ECR (segun `CLOUD_PROVIDER`)
2. **Release**: Crea GitHub release automatico
3. **Deploy**: Despliega a AKS/EKS con manifiestos K8s
4. **Validate**: Verifica health de la aplicación

### **Infrastructure** (Manual)
- Selector de **Cloud Provider**: Azure / AWS
- `create`: Provisiona registry + cluster + Traefik
- `destroy`: Elimina recursos

## Testing

### **Obtener endpoint externo**
```bash
kubectl get service -n traefik traefik
```

### **Test de la Aplicación**
```bash
# Main endpoint
GET http://[EXTERNAL-ADDR]/
Response: {"message": "Hola ED Team"}

# Health check
GET http://[EXTERNAL-ADDR]/health
Response: {"status": "healthy"}
```

## Seguridad Implementada

- ✅ **SAST** con CodeQL
- ✅ **Dependency scanning** en PRs
- ✅ **Container vulnerability scanning** con Grype
- ✅ **Non-root containers** con security contexts
- ✅ **Rulesets** con PR approvals
- ✅ **Resource limits** y security policies

## Manifiestos K8s (Cloud-agnostic)

- **deployment.yml**: 2 replicas con health checks, security contexts y zero-downtime deploys
- **service.yml**: ClusterIP para comunicación interna
- **hpa.yml**: Auto-scaling 2-10 pods basado en CPU/Memory
- **ingress.yml**: Traefik para acceso externo

## Comandos Útiles

### Azure
```bash
az aks get-credentials --resource-group [RG] --name [AKS_NAME]
kubectl get pods -l app=ed-team-app
kubectl get service -n traefik traefik
```

### AWS
```bash
aws eks update-kubeconfig --name [EKS_CLUSTER_NAME] --region [REGION]
kubectl get pods -l app=ed-team-app
kubectl get service -n traefik traefik
```

## ✅ Requisitos Cumplidos

### **Funcionales**
- [x] Aplicación "Hola ED Team"
- [x] Automatización con GitHub Actions
- [x] Trunk-based development
- [x] Recursos cloud automatizados (Azure + AWS)

### **Técnicos**
- [x] Multi-cloud: Azure (AKS + ACR) / AWS (EKS + ECR)
- [x] Traefik Ingress Controller
- [x] Build y push automatizado
- [x] Deploy con manifiestos K8s completos
- [x] Validación de pods y servicios
- [x] Rulesets con PR approvals
- [x] Escaneo de seguridad

## 🎯 Características Destacadas

- **Multi-Cloud**: Selector Azure / AWS en un solo workflow
- **GitOps**: Trunk-based development con PR obligatorios
- **Security**: Scanning automatizado en pipeline (SAST + SCA + Container)
- **HA**: 2 replicas minimas con auto-scaling hasta 10
- **Zero-Downtime**: RollingUpdate con maxUnavailable: 0
- **Monitoring**: Health checks y resource monitoring
- **Automation**: Infrastructure as Code completo
