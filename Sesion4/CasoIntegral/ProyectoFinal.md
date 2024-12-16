### Proyecto final: Pipeline completo de CI/CD

En esta sección, construiremos un **pipeline completo de CI/CD** en **Azure DevOps** que implementa las mejores prácticas aprendidas durante el curso. El pipeline incluirá las etapas de **compilación**, **pruebas**, **análisis de seguridad** y **despliegue automatizado en múltiples entornos**, integrando servicios clave como **Azure Key Vault**, **Application Insights** y **análisis de vulnerabilidades**.

---

#### 1. **Objetivos del proyecto**  

- Implementar un pipeline funcional que cubra un flujo completo de CI/CD.
- Automatizar la compilación, pruebas, análisis de seguridad y despliegues.
- Aplicar herramientas y configuraciones seguras como Key Vault y Application Insights.
- Desplegar la aplicación en entornos **Desarrollo**, **Testing** y **Producción**.

---

#### 2. **Estructura del pipeline**

El pipeline se dividirá en las siguientes etapas (**stages**):

1. **Build**: Compilación de la aplicación y generación de artefactos.
2. **Test**: Ejecución de pruebas unitarias y funcionales.
3. **Security Analysis**: Análisis de vulnerabilidades en el código y dependencias.
4. **Deploy**: Despliegue automatizado en múltiples entornos (Dev, Test, Prod).
5. **Monitoring**: Configuración de monitoreo con **Application Insights**.

---

#### 3. **Pipeline completo en YAML**

El siguiente archivo YAML describe el pipeline completo de CI/CD:

```yaml
trigger:
  - main

variables:
  azureSubscription: 'MyAzureSubscription'
  keyVaultName: 'MyKeyVault'
  appInsightsName: 'MyAppInsights'
  webAppName: 'MyWebApp'
  buildConfiguration: 'Release'

stages:
  # 1. Build Stage
  - stage: Build
    displayName: "Compilación de la Aplicación"
    jobs:
      - job: BuildJob
        displayName: "Compilar y Generar Artefactos"
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: UseDotNet@2
            inputs:
              version: '6.x'
            displayName: "Instalar .NET SDK"

          - script: |
              dotnet build --configuration $(buildConfiguration)
              dotnet publish --configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)
            displayName: "Compilar y Publicar Artefactos"

          - publish: $(Build.ArtifactStagingDirectory)
            artifact: drop
            displayName: "Publicar Artefactos"

  # 2. Test Stage
  - stage: Test
    displayName: "Ejecución de Pruebas"
    dependsOn: Build
    jobs:
      - job: TestJob
        displayName: "Pruebas Unitarias"
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - download: current
            artifact: drop
            displayName: "Descargar Artefactos"

          - script: |
              dotnet test --configuration $(buildConfiguration)
            displayName: "Ejecutar Pruebas Unitarias"

  # 3. Security Analysis Stage
  - stage: SecurityAnalysis
    displayName: "Análisis de Seguridad"
    dependsOn: Build
    jobs:
      - job: SecurityJob
        displayName: "Escanear Vulnerabilidades"
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - script: |
              echo "Escaneando dependencias..."
              curl -L https://github.com/jeremylong/DependencyCheck/releases/download/v7.0.0/dependency-check-7.0.0-release.zip -o dc.zip
              unzip dc.zip -d dependency-check
              ./dependency-check/bin/dependency-check.sh --project "MyProject" --scan "$(Build.SourcesDirectory)" --format "HTML" --out "dependency-report"
            displayName: "Escaneo con OWASP Dependency-Check"

          - publish: 'dependency-report'
            artifact: 'SecurityReport'
            displayName: "Publicar Reporte de Seguridad"

  # 4. Deploy Stage
  - stage: Deploy
    displayName: "Despliegue en Entornos"
    dependsOn: [Test, SecurityAnalysis]
    jobs:
      - deployment: DeployDev
        displayName: "Desplegar en Desarrollo"
        environment: Development
        pool:
          vmImage: 'windows-latest'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureKeyVault@2
                  inputs:
                    azureSubscription: $(azureSubscription)
                    KeyVaultName: $(keyVaultName)
                    SecretsFilter: '*'
                  displayName: "Obtener Secretos de Key Vault"

                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: $(azureSubscription)
                    appType: 'webApp'
                    appName: $(webAppName)
                    package: $(Pipeline.Workspace)/drop/**/*.zip
                  displayName: "Desplegar en Azure WebApp"

  # 5. Monitoring Stage
  - stage: Monitoring
    displayName: "Configurar Monitoreo"
    dependsOn: Deploy
    jobs:
      - job: ConfigureAppInsights
        displayName: "Configurar Application Insights"
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: AzureCLI@2
            inputs:
              azureSubscription: $(azureSubscription)
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az monitor app-insights component update \
                --app $(appInsightsName) \
                --resource-group MyResourceGroup \
                --application-type web
            displayName: "Configurar Application Insights"
```

---

#### 4. **Descripción del Pipeline**

- **Build**: Compila el código fuente y publica los artefactos necesarios.  
- **Test**: Ejecuta pruebas unitarias sobre los artefactos generados.  
- **Security Analysis**: Realiza un escaneo de seguridad con **OWASP Dependency-Check** y publica el reporte.  
- **Deploy**: Despliega la aplicación en el entorno **Desarrollo** utilizando Azure Web Apps y recuperando secretos desde **Key Vault**.  
- **Monitoring**: Configura **Application Insights** para monitorear la aplicación en tiempo real.

---

#### 5. **Objetivos del laboratorio práctico**

1. Implementar el pipeline YAML en Azure DevOps.
2. Configurar Azure Key Vault y Application Insights.
3. Desplegar la aplicación en un entorno de Desarrollo.
4. Analizar los reportes de seguridad y los resultados del monitoreo.

---

Este pipeline completo proporciona una solución práctica y funcional de CI/CD, integrando herramientas clave y aplicando buenas prácticas para la automatización de despliegues en múltiples entornos.