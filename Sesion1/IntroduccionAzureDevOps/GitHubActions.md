# Alternativa con GitHub Actions

**GitHub Actions** es una plataforma de automatización integrada directamente en GitHub que permite configurar flujos de trabajo para CI/CD y otros procesos automatizados. Como alternativa a **Azure DevOps**, GitHub Actions destaca por su flexibilidad, su integración nativa con repositorios de GitHub y su simplicidad para empezar.

---

## ¿Qué es GitHub Actions?

GitHub Actions es un servicio que permite crear flujos de trabajo mediante **archivos YAML**, definidos directamente en el repositorio. Los flujos de trabajo pueden desencadenarse en respuesta a eventos como:

- Commits en el código.
- Creación o fusión de Pull Requests.
- Lanzamientos de versiones (tags).
- Acciones manuales o programadas (cron jobs).

Es compatible con cualquier lenguaje o plataforma, incluyendo **.NET**, y puede integrarse con servicios externos para tareas avanzadas.

---

## Características Clave de GitHub Actions

1. **Integración Nativa con GitHub**:
   - Funciona directamente con los repositorios en GitHub, sin necesidad de configuración adicional.

2. **Pipeline como Código**:
   - Los flujos de trabajo se definen en archivos YAML ubicados en `.github/workflows`.

3. **Soporte Multiplataforma**:
   - Permite ejecutar trabajos en runners para **Windows**, **Linux** y **macOS**.

4. **Marketplace de Acciones**:
   - Dispone de un catálogo de acciones predefinidas (plugins) que simplifican tareas como despliegues, construcción y pruebas.

5. **Ejecución Gratuita en Repositorios Públicos**:
   - Ofrece minutos gratuitos en runners hospedados por GitHub para proyectos de código abierto.

---

## Componentes de GitHub Actions

1. **Eventos**:
   - Determinan cuándo se activa el flujo de trabajo.
   - Ejemplos:
     - `push`: Cada vez que se realiza un push al repositorio.
     - `pull_request`: Al abrir, actualizar o cerrar una Pull Request.
     - `schedule`: Desencadenadores basados en cron.

2. **Jobs**:
   - Tareas independientes que se ejecutan como parte del flujo de trabajo.
   - Se pueden ejecutar en paralelo o de forma secuencial.

3. **Steps**:
   - Comandos individuales que se ejecutan dentro de un job.
   - Pueden incluir acciones predefinidas o comandos personalizados.

4. **Runners**:
   - Máquinas virtuales que ejecutan los trabajos.
   - GitHub proporciona runners hospedados, pero también se pueden usar runners auto-hospedados.

---

## Flujo de Trabajo Típico con GitHub Actions

Un pipeline básico de CI/CD con GitHub Actions para una aplicación .NET podría seguir los siguientes pasos:

### **1. Crear el Archivo del Flujo de Trabajo**
   - Ubicar el archivo en `.github/workflows/ci-cd.yml`.

```yaml
name: CI/CD for .NET App

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 6.0

    - name: Restore Dependencies
      run: dotnet restore

    - name: Build Application
      run: dotnet build --no-restore --configuration Release

    - name: Run Tests
      run: dotnet test --no-build --configuration Release
```

---

### **2. Añadir Despliegue al Flujo de Trabajo**
   - Para despliegues en **Azure App Service**:

```yaml
deploy:
  needs: build
  runs-on: ubuntu-latest

  steps:
  - name: Checkout Code
    uses: actions/checkout@v3

  - name: Setup Azure CLI
    uses: azure/login@v1
    with:
      creds: ${{ secrets.AZURE_CREDENTIALS }}

  - name: Deploy to Azure App Service
    run: az webapp deploy --name MyAppService --resource-group MyResourceGroup --src-path ./bin/Release/net6.0
```

---

## Beneficios de GitHub Actions

1. **Integración Simple**:
   - Todo el flujo de trabajo está vinculado directamente al repositorio de GitHub.

2. **Flexibilidad**:
   - Define cualquier tipo de flujo de trabajo para construcción, pruebas, despliegue, y más.

3. **Marketplace de Acciones**:
   - Permite reutilizar acciones creadas por la comunidad, reduciendo el esfuerzo de configuración.

4. **Ecosistema Global**:
   - Compatible con servicios externos como AWS, Azure, Docker Hub, entre otros.

---

## Casos de Uso de GitHub Actions

1. **Integración Continua (CI)**:
   - Automatizar la construcción y pruebas para asegurar la calidad del código.

2. **Entrega Continua (CD)**:
   - Desplegar automáticamente aplicaciones a servicios como Azure, AWS o Kubernetes.

3. **Mantenimiento del Repositorio**:
   - Automatizar tareas como análisis de dependencias o ejecución de herramientas de linting.

4. **Automatización de Operaciones**:
   - Configuración de scripts para migraciones de bases de datos, limpieza de recursos, etc.

---

## Comparativa: Azure DevOps vs. GitHub Actions

| **Característica**         | **Azure DevOps**              | **GitHub Actions**             |
|----------------------------|-------------------------------|---------------------------------|
| **Integración con GitHub** | Parcial (requiere conexión)   | Total (nativa)                 |
| **Configuración**          | Más compleja, ideal para empresas | Más sencilla, ideal para equipos pequeños |
| **Ejecución**              | Pipelines robustos y extensibles | Flujos de trabajo ligeros y flexibles |
| **Costo**                  | Pago por uso (minutos de CI/CD) | Gratuito para repositorios públicos |

---

GitHub Actions es una alternativa sólida a Azure DevOps, especialmente para proyectos que ya están alojados en GitHub. Su flexibilidad y facilidad de uso lo convierten en una excelente opción para equipos pequeños o para flujos de trabajo simples. Para proyectos más complejos, Azure DevOps sigue siendo una solución más completa y escalable.