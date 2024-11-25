### Laboratorio Práctico: Configurar un Pipeline de CI para ASP.NET Core

#### Objetivo
Configurar un pipeline de Integración Continua (CI) en Azure DevOps que realice las siguientes tareas:
1. Restaurar dependencias de un proyecto ASP.NET Core.
2. Compilar la aplicación.
3. Ejecutar pruebas unitarias.
4. Generar artefactos listos para el despliegue.

---

#### Requisitos previos
1. **Proyecto ASP.NET Core**:
   - Tener un proyecto ASP.NET Core disponible en un repositorio Git (Azure DevOps o GitHub).

2. **Azure DevOps**:
   - Contar con una organización y proyecto configurados en Azure DevOps.
   - Acceso a un agente de build (hosted o self-hosted).

---

#### Pasos

##### Paso 1: Crear un nuevo pipeline en Azure DevOps
1. **Acceder al portal de Azure DevOps**:
   - Navegar al proyecto correspondiente.
   - Ir a la sección **Pipelines** y seleccionar **New pipeline**.

2. **Seleccionar el repositorio**:
   - Si el proyecto está en Azure Repos, seleccionarlo.
   - Si está en GitHub, conectar la cuenta y elegir el repositorio.

3. **Elegir configuración YAML**:
   - Seleccionar la opción para crear un pipeline basado en un archivo YAML.
   - Si no existe un archivo `azure-pipelines.yml`, Azure DevOps sugerirá crearlo.

---

##### Paso 2: Definir el archivo YAML del pipeline

1. Crear un archivo llamado `azure-pipelines.yml` en la raíz del repositorio.
2. Añadir la configuración inicial para el pipeline:
   ```yaml
   trigger:
     - main

   pool:
     vmImage: 'windows-latest'

   steps:
     - task: UseDotNet@2
       inputs:
         packageType: 'sdk'
         version: '6.x' # Cambiar según la versión de .NET utilizada

     - script: dotnet restore
       displayName: 'Restaurar dependencias'

     - script: dotnet build --configuration Release
       displayName: 'Compilar la solución'

     - script: dotnet test --configuration Release --logger:trx
       displayName: 'Ejecutar pruebas unitarias'

     - task: PublishTestResults@2
       inputs:
         testResultsFormat: 'VSTest'
         testResultsFiles: '**/*.trx'
       displayName: 'Publicar resultados de pruebas'

     - script: dotnet publish --configuration Release --output $(Build.ArtifactStagingDirectory)
       displayName: 'Generar artefactos'

     - task: PublishBuildArtifacts@1
       inputs:
         pathToPublish: $(Build.ArtifactStagingDirectory)
         artifactName: 'drop'
       displayName: 'Publicar artefactos'
   ```

3. **Explicación del YAML**:
   - **Trigger**: Define que el pipeline se ejecutará en cada commit en la rama `main`.
   - **Pool**: Usa una máquina virtual hospedada con imagen de Windows.
   - **Steps**:
     - `UseDotNet@2`: Configura la versión del SDK de .NET.
     - `dotnet restore`: Restaura las dependencias del proyecto.
     - `dotnet build`: Compila el proyecto en modo Release.
     - `dotnet test`: Ejecuta las pruebas unitarias y genera un archivo de resultados.
     - `PublishTestResults@2`: Publica los resultados de las pruebas para visualización en Azure DevOps.
     - `dotnet publish`: Genera artefactos para despliegue.
     - `PublishBuildArtifacts@1`: Publica los artefactos como parte del pipeline.

---

##### Paso 3: Guardar y ejecutar el pipeline
1. Guardar el archivo `azure-pipelines.yml` en el repositorio.
2. Volver a la interfaz de Azure DevOps y ejecutar el pipeline.
   - Seleccionar **Run pipeline** y confirmar la ejecución.

---

##### Paso 4: Validar los resultados
1. **Resultados del pipeline**:
   - Verificar cada tarea en el pipeline para confirmar su éxito.
   - Revisar los resultados de las pruebas unitarias en la pestaña **Tests**.
   - Confirmar que los artefactos se generaron en la sección **Artifacts**.

2. **Descarga de artefactos**:
   - Descargar el artefacto generado y comprobar que contiene los binarios del proyecto listos para despliegue.

---

#### Resultado esperado
- Pipeline de CI configurado y funcionando para un proyecto ASP.NET Core.
- Artefactos generados y pruebas unitarias ejecutadas correctamente.