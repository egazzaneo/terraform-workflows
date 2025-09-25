# Terraform Reusable Workflows

Workflows centralizados y reutilizables para pipelines de Terraform multi-environment.

## 🚀 Características

- **Workflows reutilizables**: Centraliza la lógica en un solo repositorio
- **Configuración flexible**: Personalizable por proyecto
- **Pipeline resiliente**: Continúa ejecutándose aunque fallen algunos environments
- **Pipelines separados**: Pipeline principal y destroy independientes
- **Selección de environments**: Ejecuta en environment específico o todos

## 📁 Estructura

```
terraform-workflows/
├── .github/
│   ├── workflows/
│   │   ├── terraform-reusable.yml         # Pipeline principal reutilizable
│   │   └── terraform-destroy-reusable.yml # Pipeline destroy reutilizable
│   └── actions/
│       └── discover-environments/         # Action para detectar environments
│           └── action.yml
└── README.md
```

## 🔧 Uso en Proyectos

### 1. Pipeline Principal

```yaml
# proyecto/.github/workflows/terraform.yml
name: 'Terraform Pipeline'

on:
  push:
    branches: ["main"]
    paths: ["environments/**"]
  pull_request:
    branches: ["main"]
    paths: ["environments/**"]
  workflow_dispatch:
    inputs:
      action:
        description: 'Action to perform'
        required: true
        default: 'plan'
        type: choice
        options: [plan, apply]
      environment:
        description: 'Environment to target (leave empty for all)'
        required: false
        type: string

jobs:
  terraform:
    uses: org/terraform-workflows/.github/workflows/terraform-reusable.yml@main
    with:
      environments_dir: 'environments'
      terraform_version: 'latest'
      aws_region: 'us-east-1'
      action: ${{ github.event.inputs.action || 'plan' }}
      environment: ${{ github.event.inputs.environment || '' }}
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### 2. Pipeline Destroy

```yaml
# proyecto/.github/workflows/terraform-destroy.yml
name: 'Terraform Destroy Pipeline'

on:
  workflow_dispatch:
    inputs:
      action:
        description: 'Destroy action to perform'
        required: true
        default: 'destroy-plan'
        type: choice
        options: [destroy-plan, destroy]
      environment:
        description: 'Environment to destroy (leave empty for all)'
        required: false
        type: string

jobs:
  terraform-destroy:
    uses: org/terraform-workflows/.github/workflows/terraform-destroy-reusable.yml@main
    with:
      environments_dir: 'environments'
      terraform_version: 'latest'
      aws_region: 'us-east-1'
      action: ${{ github.event.inputs.action }}
      environment: ${{ github.event.inputs.environment || '' }}
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## ⚙️ Parámetros

### Inputs

| Parámetro | Descripción | Requerido | Default |
|-----------|-------------|-----------|---------|
| `environments_dir` | Directorio de environments | No | `environments` |
| `terraform_version` | Versión de Terraform | No | `latest` |
| `aws_region` | Región de AWS | Sí | - |
| `action` | Acción a realizar | Varía | `plan` |
| `environment` | Environment específico | No | `''` (todos) |

### Secrets

| Secret | Descripción | Requerido |
|--------|-------------|-----------|
| `AWS_ACCESS_KEY_ID` | Access Key de AWS | Sí |
| `AWS_SECRET_ACCESS_KEY` | Secret Key de AWS | Sí |

## 🎯 Ventajas

- **Mantenimiento centralizado**: Un solo lugar para actualizar workflows
- **Consistencia**: Misma lógica en todos los proyectos
- **Reutilización**: No duplicar código entre proyectos
- **Versionado**: Usar tags para versiones específicas
- **Flexibilidad**: Personalizable por proyecto

## 📝 Ejemplo de Proyecto

```
mi-proyecto/
├── .github/workflows/
│   ├── terraform.yml          # Usa terraform-reusable.yml
│   └── terraform-destroy.yml  # Usa terraform-destroy-reusable.yml
├── environments/
│   ├── dev/
│   └── prod/
└── .github/actions/
    └── discover-environments/ # Copia del action (si es necesario)
```

## 🔄 Actualización y Versionado

### Pasos para actualizar workflows:

```bash
# 1. Hacer cambios en terraform-workflows
cd terraform-workflows
git add .
git commit -m "Update workflows with new features"

# 2. Pushear cambios
git push origin main

# 3. Crear/actualizar tag
git tag v1.0.0
git push origin v1.0.0

# 4. Para sobrescribir tag existente (si es necesario)
git tag -f v1.0.0                    # Forzar tag local
git push origin v1.0.0 --force       # Forzar push del tag

# 5. Para eliminar tag (si necesitas recrearlo)
git tag -d v1.0.0                    # Eliminar tag local
git push origin :refs/tags/v1.0.0    # Eliminar tag remoto
git tag v1.0.0                       # Crear nuevo tag
git push origin v1.0.0               # Pushear nuevo tag
```

### Versionado recomendado:
- `v1.0.0` - Versión inicial estable
- `v1.1.0` - Nuevas características
- `v1.0.1` - Correcciones de bugs
- `v2.0.0` - Cambios que rompen compatibilidad

### Uso en proyectos:
```yaml
# Usar versión específica (recomendado)
uses: egazzaneo/terraform-workflows/.github/workflows/terraform-reusable.yml@v1.0.0

# Usar rama main (no recomendado para producción)
uses: egazzaneo/terraform-workflows/.github/workflows/terraform-reusable.yml@main
```

## 🛡️ Seguridad

- Apply y destroy solo manuales
- Pipelines separados para mayor seguridad
- Validación previa obligatoria
- Credenciales via GitHub Secrets