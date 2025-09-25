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

## 🔄 Actualización

Para actualizar todos los proyectos:
1. Modifica los workflows en `terraform-workflows`
2. Crea un nuevo tag: `git tag v1.1.0`
3. Los proyectos usan `@v1.1.0` en lugar de `@main`

## 🛡️ Seguridad

- Apply y destroy solo manuales
- Pipelines separados para mayor seguridad
- Validación previa obligatoria
- Credenciales via GitHub Secrets