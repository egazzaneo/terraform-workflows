# Terraform Reusable Workflows 🚀

Colección de workflows reutilizables para automatizar pipelines de Terraform con GitHub Actions.

## 📋 Índice

- [Terraform Workflows](#terraform-workflows)
- [Configuración Inicial](#configuración-inicial)
- [Uso en Proyectos](#uso-en-proyectos)
- [Versionado y Mantenimiento](#versionado-y-mantenimiento)
- [Mejores Prácticas](#mejores-prácticas)

## 🏗️ Terraform Workflows

### Características

- **Multi-environment**: Soporte para dev, staging, prod
- **Validación automática**: Terraform validate + plan en cada push
- **Deploy manual**: Apply solo con confirmación manual
- **Destroy seguro**: Pipeline separado con doble confirmación
- **Detección automática**: Descubre environments dinámicamente

### Estructura del Repositorio

```
terraform-workflows/
├── .github/workflows/
│   ├── terraform-reusable.yml         # Pipeline principal
│   └── terraform-destroy-reusable.yml # Pipeline de destrucción
├── example-project/                   # Proyecto de ejemplo
│   ├── .github/workflows/
│   │   ├── terraform.yml
│   │   └── terraform-destroy.yml
│   └── environments/
│       ├── dev/
│       └── prod/
├── SETUP.md                          # Guía de configuración
└── README.md
```

## ⚙️ Configuración Inicial

### 1. Crear Repositorio Central

```bash
# Crear repositorio público para workflows reutilizables
gh repo create terraform-workflows --public
cd terraform-workflows

# Configurar remote y subir
git remote add origin https://github.com/TU-ORG/terraform-workflows.git
git add .
git commit -m "Initial terraform reusable workflows"
git branch -M main
git push -u origin main

# Crear tag para versionado
git tag v1.0.0
git push origin v1.0.0
```

### 2. Configurar Proyecto

```bash
# En tu proyecto de Terraform
cd mi-proyecto-terraform

# Crear estructura de workflows
mkdir -p .github/workflows

# Crear environments
mkdir -p environments/dev environments/prod
```

## 🔧 Uso en Proyectos

### Pipeline Principal

```yaml
# .github/workflows/terraform.yml
name: 'Terraform'

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:
    inputs:
      action:
        description: 'Action to perform'
        required: true
        default: 'plan'
        type: choice
        options: [plan, apply]

jobs:
  terraform:
    uses: TU-ORG/terraform-workflows/.github/workflows/terraform-reusable.yml@v1.0.0
    with:
      aws_region: 'us-east-1'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Pipeline Destroy

```yaml
# .github/workflows/terraform-destroy.yml
name: 'Terraform Destroy'

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to destroy'
        required: true
        type: choice
        options: [dev, prod]
      action:
        description: 'Action to perform'
        required: true
        type: choice
        options: [destroy-plan, destroy]
      confirm:
        description: 'Type "destroy" to confirm (only for destroy action)'
        required: false
        type: string

jobs:
  terraform-destroy:
    if: inputs.action == 'destroy-plan' || (inputs.action == 'destroy' && inputs.confirm == 'destroy')
    uses: TU-ORG/terraform-workflows/.github/workflows/terraform-destroy-reusable.yml@v1.0.0
    with:
      aws_region: 'us-east-1'
      environment: ${{ inputs.environment }}
      action: ${{ inputs.action }}
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Configurar Secrets

En GitHub → Settings → Secrets and variables → Actions:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

## 📊 Parámetros Disponibles

### Inputs

| Parámetro | Descripción | Requerido | Default |
|-----------|-------------|-----------|---------|
| `aws_region` | Región de AWS | ✅ | - |
| `terraform_version` | Versión de Terraform | ❌ | `latest` |
| `environments_dir` | Directorio de environments | ❌ | `environments` |
| `environment` | Environment específico | ❌ | `''` (todos) |
| `action` | Acción a realizar | ❌ | `plan` |

### Secrets

| Secret | Descripción | Requerido |
|--------|-------------|-----------|
| `AWS_ACCESS_KEY_ID` | Access Key de AWS | ✅ |
| `AWS_SECRET_ACCESS_KEY` | Secret Key de AWS | ✅ |

## 🔄 Versionado y Mantenimiento

### Actualizar Workflows

```bash
# 1. Hacer cambios en terraform-workflows
cd terraform-workflows
git add .
git commit -m "Update workflows with new features"
git push origin main

# 2. Crear/actualizar tag
git tag v1.1.0
git push origin v1.1.0

# 3. Para sobrescribir tag existente
git tag -f v1.0.0
git push origin v1.0.0 --force

# 4. Para eliminar y recrear tag
git tag -d v1.0.0
git push origin :refs/tags/v1.0.0
git tag v1.0.0
git push origin v1.0.0
```

### Versionado Semántico

- `v1.0.0` - Versión inicial estable
- `v1.1.0` - Nuevas características
- `v1.0.1` - Correcciones de bugs
- `v2.0.0` - Cambios que rompen compatibilidad

## 🎯 Mejores Prácticas

### Seguridad

- ✅ **Repositorio público** para workflows reutilizables
- ✅ **Proyectos privados** pueden usar workflows públicos
- ✅ **Apply manual** solo con confirmación
- ✅ **Destroy con doble confirmación**
- ✅ **Secrets centralizados** por proyecto

### Organización

- ✅ **Un repositorio** para todos los workflows reutilizables
- ✅ **Tags específicos** en producción (`@v1.0.0`)
- ✅ **Rama main** solo para desarrollo (`@main`)
- ✅ **Documentación actualizada** en cada cambio

### Estructura de Proyecto

```
mi-proyecto/
├── .github/workflows/
│   ├── terraform.yml          # Pipeline principal
│   └── terraform-destroy.yml  # Pipeline destroy
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── backend.tf
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── backend.tf
└── modules/                   # Módulos reutilizables
    └── s3/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

## 🚦 Flujo de Trabajo

### Desarrollo

1. **Push a main** → Ejecuta `validate` + `plan` automáticamente
2. **Pull Request** → Ejecuta `validate` + `plan` para revisión
3. **Manual Apply** → Ejecuta `apply` solo cuando sea necesario

### Destroy

1. **Destroy Plan** → Revisa qué se destruirá (seguro)
2. **Destroy** → Ejecuta destrucción real (requiere confirmación)

## 📞 Soporte

- **Issues**: Reportar problemas en el repositorio
- **Documentación**: Ver `SETUP.md` para configuración detallada
- **Ejemplos**: Revisar `example-project/` para referencia

---

**Creado por**: DevOps Team  
**Versión**: v1.0.0  
**Última actualización**: $(date +%Y-%m-%d)