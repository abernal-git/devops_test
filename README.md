# 🚀 DevOps Auto-Deploy Web App

![CI/CD Pipeline](https://github.com/abernal-git/devops_test/actions/workflows/deploy.yml/badge.svg)

Pipeline DevOps completo con Flask, Docker, Terraform, Ansible, Kubernetes y GitHub Actions desplegado en **Azure AKS**.

---

## 📐 Arquitectura

```
git push
  → GitHub Actions (CI/CD)
    → Docker build (linux/amd64)
      → Push a Azure Container Registry (ACR)
        → Ansible (configura entorno)
          → kubectl apply → Azure AKS
            → App pública en internet ✅
```

---

## 🛠️ Stack Tecnológico

| Tecnología | Versión | Rol |
|---|---|---|
| Python / Flask | 3.11 | App web con endpoints `/` y `/health` |
| Docker | - | Empaqueta la app en contenedor AMD64 |
| Terraform | 1.14.6 | Crea infraestructura en Azure (IaC) |
| Ansible | 2.20.2 | Configura el entorno y verifica el cluster |
| Kubernetes | 1.33.7 | Orquesta contenedores en AKS |
| Azure AKS | - | Cluster Kubernetes gestionado |
| Azure ACR | - | Registry privado de imágenes Docker |
| GitHub Actions | - | Pipeline CI/CD automático |

---

## 📁 Estructura del Proyecto

```
devops_test/
├── app.py                          # Flask app
├── Dockerfile                      # Imagen Docker AMD64
├── requirements.txt                # Dependencias Python
├── .gitignore
├── .github/
│   └── workflows/
│       └── deploy.yml              # Pipeline CI/CD
├── terraform/
│   ├── main.tf                     # Recursos Azure (RG, ACR, AKS)
│   ├── variables.tf                # Variables de configuración
│   └── outputs.tf
├── ansible/
│   ├── playbook.yml                # Configura entorno y verifica cluster
│   └── inventory.ini
└── k8s/
    ├── deployment.yaml             # 2 réplicas de la app
    └── service.yaml                # LoadBalancer con IP pública
```

---

## ⚙️ Pipeline CI/CD

Cada `git push` a `main` o `master` dispara automáticamente:

1. **Login a Azure** — autenticación con Service Principal
2. **Docker build & push** — construye imagen `linux/amd64` y la sube al ACR
3. **Conectar kubectl** — configura kubeconfig para el cluster AKS
4. **Deploy** — actualiza el deployment en Kubernetes y verifica el rollout

---

## 🚀 Setup desde Cero

### Prerequisitos
- Azure CLI (`brew install azure-cli`)
- Terraform (`brew install hashicorp/tap/terraform`)
- kubectl (`brew install kubectl`)
- Ansible (`pip install ansible`)
- Docker Desktop

### 1. Login a Azure
```bash
az login
```

### 2. Crear infraestructura
```bash
cd terraform
terraform init
terraform apply
```

Recursos creados:
- Resource Group: `devops-test-rg`
- Container Registry: `devopsacrabernal093`
- AKS Cluster: `devops-aks` (1 nodo Standard_D2s_v3, eastus2)

### 3. Conectar kubectl
```bash
az aks get-credentials --resource-group devops-test-rg --name devops-aks
```

### 4. Configurar entorno con Ansible
```bash
cd ansible
ansible-playbook -i inventory.ini playbook.yml
```

### 5. Build y push imagen Docker
```bash
az acr login --name devopsacrabernal093

docker buildx build --platform linux/amd64 \
  -t devopsacrabernal093.azurecr.io/devops-app:latest \
  --push .
```

### 6. Deploy en Kubernetes
```bash
kubectl apply -f k8s/
kubectl get service devops-webapp-svc
```

### 7. Probar la app
```bash
curl http://<EXTERNAL-IP>/health
# {"status":"ok"}
```

---

## 🔐 Secrets Requeridos en GitHub

| Secret | Descripción |
|---|---|
| `AZURE_CREDENTIALS` | JSON del Service Principal de Azure |

Generar con:
```bash
az ad sp create-for-rbac \
  --name "devops-github-actions" \
  --role contributor \
  --scopes /subscriptions/<SUBSCRIPTION_ID> \
  --sdk-auth
```

---

## 🧹 Limpiar Recursos

```bash
cd terraform && terraform destroy
```

> ⚠️ El cluster AKS cuesta ~$0.10/hr. Siempre destruye la infra cuando no la uses.

---

## 🐛 Troubleshooting

| Error | Causa | Solución |
|---|---|---|
| `exec format error` | Imagen ARM64 en servidor AMD64 | Usar `--platform linux/amd64` |
| `ErrImagePull 401` | AKS sin permisos en ACR | `az aks update --attach-acr` |
| `CrashLoopBackOff` | App falla al iniciar | `kubectl logs <pod>` |
| `Port 5000 in use` | AirPlay Receiver en macOS | Desactivar en System Settings |

---

## 📄 Licencia

MIT — libre para usar como base de aprendizaje.

---

*Proyecto desarrollado como hands-on de DevOps — Marzo 2026*
