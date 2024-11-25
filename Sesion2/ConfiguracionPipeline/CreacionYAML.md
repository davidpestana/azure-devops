# Creación de Pipelines en YAML para Azure DevOps

Los pipelines en YAML permiten definir como código los flujos de integración continua (CI) y entrega continua (CD). Esto mejora la trazabilidad, portabilidad y capacidad de colaboración, ya que el archivo YAML se versiona junto con el código fuente del proyecto.

---

## **1. ¿Qué es un Pipeline en YAML?**

Un pipeline en YAML es un archivo de texto que contiene una descripción estructurada de los pasos necesarios para construir, probar y desplegar una aplicación. En Azure DevOps, estos archivos se colocan normalmente en la raíz del repositorio, típicamente con el nombre `azure-pipelines.yml`.

---

## **2. Estructura Básica de un Pipeline YAML**

Un pipeline en YAML tiene los siguientes componentes clave:

### **2.1. Triggers**
Determina cuándo debe activarse el pipeline.

- Ejemplo: Ejecutar en cada cambio en la rama `main`:
  ```yaml
  trigger:
    branches:
      include:
        - main
  ```

### **2.2. Variables**
Definen valores reutilizables en todo el pipeline.

- Ejemplo:
  ```yaml
  variables:
    buildConfiguration: Release
    dotnetVersion: '6.0'
  ```

### **2.3. Pool**
Especifica el agente donde se ejecutarán las tareas.

- Ejemplo: Usar un agente de Microsoft hospedado:
  ```yaml
  pool:
    vmImage: 'windows-latest'
  ```

### **2.4. Steps**
Define los pasos individuales que se ejecutarán en el pipeline.

- Ejemplo: Restaurar dependencias y compilar una aplicación .NET:
  ```yaml
  steps:
  - script: dotnet restore
    displayName: 'Restore Dependencies'

  - script: dotnet build --configuration $(buildConfiguration)
    displayName: 'Build Application'
  ```

---

## **3. Pipeline Completo para una Aplicación .NET Core**

Aquí tienes un pipeline básico para construir, probar y generar artefactos para una aplicación ASP.NET Core:

```yaml
trigger:
  branches:
    include:
      - main

variables:
  buildConfiguration: Release
  dotnetVersion: '6.0'

pool:
  vmImage: 'windows-latest'

steps:
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '$(dotnetVersion)'

- script: dotnet restore
  displayName: 'Restore Dependencies'

- script: dotnet build --configuration $(buildConfiguration)
  displayName: 'Build Application'

- script: dotnet test --configuration $(buildConfiguration)
  displayName: 'Run Tests'

- script: dotnet publish --configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)
  displayName: 'Publish Application'

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
  displayName: 'Publish Artifacts'
```

---

## **4. Componentes Avanzados de un Pipeline YAML**

### **4.1. Jobs**
Un pipeline puede contener múltiples jobs que se ejecutan en paralelo o en secuencia.

- Ejemplo: Dos jobs, uno para pruebas y otro para despliegue:
  ```yaml
  jobs:
  - job: BuildAndTest
    displayName: 'Build and Test Job'
    steps:
    - script: dotnet build
      displayName: 'Build Application'
    - script: dotnet test
      displayName: 'Run Tests'

  - job: Deploy
    displayName: 'Deployment Job'
    dependsOn: BuildAndTest
    steps:
    - script: echo "Deploying application..."
      displayName: 'Deploy Application'
  ```

---

### **4.2. Estrategias de Matriz**
Permiten ejecutar un conjunto de trabajos con diferentes configuraciones, como versiones de .NET o sistemas operativos.

- Ejemplo: Probar en diferentes versiones de .NET:
  ```yaml
  strategy:
    matrix:
      dotnet5:
        dotnetVersion: '5.0'
      dotnet6:
        dotnetVersion: '6.0'

  steps:
  - task: UseDotNet@2
    inputs:
      packageType: 'sdk'
      version: '$(dotnetVersion)'

  - script: dotnet test
    displayName: 'Run Tests'
  ```

---

### **4.3. Templates**
Los templates permiten reutilizar configuraciones comunes en múltiples pipelines.

1. **Crear un Template**:
   - Archivo `build-template.yml`:
     ```yaml
     parameters:
       dotnetVersion: '6.0'
     steps:
     - task: UseDotNet@2
       inputs:
         packageType: 'sdk'
         version: ${{ parameters.dotnetVersion }}

     - script: dotnet build
       displayName: 'Build Application'
     ```

2. **Usar el Template**:
   - Archivo `azure-pipelines.yml`:
     ```yaml
     jobs:
     - template: build-template.yml
       parameters:
         dotnetVersion: '7.0'
     ```

---

### **4.4. Condicionales**
Ejecuta pasos solo si se cumplen ciertas condiciones.

- Ejemplo: Desplegar solo si la rama es `main`:
  ```yaml
  steps:
  - script: echo "Deploying application..."
    condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
  ```

---

## **5. Pipeline para CI/CD**

Un pipeline completo que incluye construcción, pruebas y despliegue en **Azure App Service**.

```yaml
trigger:
  branches:
    include:
      - main

variables:
  buildConfiguration: Release
  dotnetVersion: '6.0'
  azureSubscription: 'MyAzureServiceConnection'
  appName: 'MyAspNetCoreApp'

pool:
  vmImage: 'windows-latest'

steps:
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '$(dotnetVersion)'

- script: dotnet restore
  displayName: 'Restore Dependencies'

- script: dotnet build --configuration $(buildConfiguration)
  displayName: 'Build Application'

- script: dotnet test --configuration $(buildConfiguration)
  displayName: 'Run Tests'

- script: dotnet publish --configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)
  displayName: 'Publish Application'

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'

- task: AzureWebApp@1
  inputs:
    azureSubscription: '$(azureSubscription)'
    appName: '$(appName)'
    package: '$(Build.ArtifactStagingDirectory)'
  displayName: 'Deploy to Azure App Service'
```

---

## **6. Validar y Depurar un Pipeline YAML**

### **Errores Comunes**
1. **Formato incorrecto**:
   - Usa un validador YAML (por ejemplo, en Visual Studio Code con la extensión YAML).
2. **Variables no definidas**:
   - Asegúrate de que todas las variables necesarias están definidas en el archivo o en la configuración del pipeline.

### **Pruebas y Simulaciones**
1. Usa la opción **"Editar y Guardar"** en Azure DevOps para probar el pipeline.
2. Ejecuta el pipeline en una rama separada antes de fusionarlo con `main`.

---

## **7. Buenas Prácticas**

1. **Dividir en Jobs y Steps**:
   - Mantén el pipeline modular para facilitar la depuración.
2. **Usar Variables**:
   - Centraliza valores como rutas, nombres de aplicaciones o versiones.
3. **Control de Versiones**:
   - Versiona los archivos YAML junto con el código fuente del proyecto.
4. **Automatización Progresiva**:
   - Comienza con un pipeline simple y agrégale complejidad iterativamente.

---

Con estas configuraciones, puedes implementar pipelines YAML robustos en Azure DevOps para automatizar la construcción, pruebas y despliegues de tus aplicaciones.