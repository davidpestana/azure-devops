### Laboratorio Práctico: Crear un Pipeline de CD para ASP.NET Core

En este laboratorio, aprenderás a crear un pipeline de Entrega Continua (CD) para una aplicación ASP.NET Core utilizando **Azure DevOps**. Implementaremos despliegues iterativos en diferentes destinos: **IIS on-premises**, **Azure App Service**, y **Kubernetes**. Este laboratorio incluye pasos detallados y pruebas en cada etapa.

---

#### **1. Preparativos**

1. **Requisitos previos:**
   - Una cuenta en **Azure DevOps**.
   - Acceso a un servidor Windows con IIS configurado.
   - Una suscripción activa en **Microsoft Azure**.
   - Un clúster de Kubernetes (puede ser Azure Kubernetes Service o Minikube para pruebas locales).

2. **Configuración del entorno:**
   - Crear un repositorio en Azure DevOps y subir el código fuente de la aplicación ASP.NET Core.
   - Configurar el agente de compilación:
     - Para IIS: Un agente autoalojado en un servidor Windows.
     - Para otros despliegues: Usar agentes Microsoft-hosted.

3. **Dependencias:**
   - Instalar el SDK de .NET Core/ASP.NET Core.
   - Configurar Docker si se usará en el despliegue a Kubernetes.

---

#### **2. Crear el pipeline de CD**

##### **Paso 1: Definir el pipeline en Azure DevOps**

1. En Azure DevOps, navega a tu proyecto y selecciona **Pipelines > Create Pipeline**.
2. Selecciona la ubicación del repositorio (por ejemplo, Azure Repos Git).
3. Escoge la opción de **pipeline YAML**.
4. Crea un archivo `azure-pipelines.yml` en la raíz del repositorio con el siguiente contenido inicial:

   ```yaml
   trigger:
     - main

   pool:
     vmImage: 'windows-latest'

   stages:
     - stage: Build
       displayName: Build Stage
       jobs:
         - job: BuildJob
           displayName: Compile and Package
           steps:
             - task: UseDotNet@2
               inputs:
                 packageType: 'sdk'
                 version: '7.x'
             - task: DotNetCoreCLI@2
               inputs:
                 command: 'publish'
                 arguments: '--configuration Release --output $(Build.ArtifactStagingDirectory)'
                 projects: '**/*.csproj'
             - task: PublishBuildArtifacts@1
               inputs:
                 PathtoPublish: '$(Build.ArtifactStagingDirectory)'
                 ArtifactName: 'drop'
   ```

**Resultado esperado:**
- El pipeline compila el código y genera artefactos en un directorio llamado `drop`.

---

##### **Paso 2: Agregar despliegue a IIS**

1. Modifica el archivo YAML para incluir una etapa de despliegue a IIS:

   ```yaml
   - stage: DeployIIS
     displayName: Deploy to IIS
     dependsOn: Build
     jobs:
       - job: DeployIISJob
         displayName: Deploy ASP.NET Core to IIS
         pool:
           name: 'OnPremWindowsAgent'
         steps:
           - task: IISWebAppDeploy@1
             inputs:
               WebSiteName: 'MyIISWebsite'
               Package: '$(Pipeline.Workspace)/drop/*.zip'
               TakeAppOfflineFlag: true
               RemoveAdditionalFilesFlag: true
               AdditionalArguments: '-verbose'
   ```

2. Configura el agente `OnPremWindowsAgent` para conectarse al servidor IIS:
   - Instala un agente autoalojado en el servidor Windows.
   - Registra el agente en Azure DevOps.

3. Configura el servidor IIS:
   - Habilita Web Deploy y asegura la conectividad.
   - Crea un sitio web en IIS con el nombre `MyIISWebsite`.

**Resultado esperado:**
- La aplicación ASP.NET Core se despliega en el servidor IIS.

---

##### **Paso 3: Agregar despliegue a Azure App Service**

1. Extiende el archivo YAML para incluir un despliegue a Azure App Service:

   ```yaml
   - stage: DeployAzureAppService
     displayName: Deploy to Azure App Service
     dependsOn: Build
     jobs:
       - job: DeployAppServiceJob
         displayName: Deploy ASP.NET Core to Azure App Service
         steps:
           - task: AzureWebApp@1
             inputs:
               azureSubscription: '<AzureSubscription>'
               appType: 'webApp'
               appName: '<AppServiceName>'
               package: '$(Pipeline.Workspace)/drop/*.zip'
   ```

2. Configura un App Service en Azure:
   - Navega al portal de Azure y crea un App Service.
   - Configura un plan de App Service con el runtime de .NET.

3. Configura la conexión de servicio en Azure DevOps:
   - En **Project Settings > Service Connections**, crea una conexión de tipo Azure Resource Manager.

**Resultado esperado:**
- La aplicación se despliega en el App Service de Azure.

---

##### **Paso 4: Agregar despliegue a Kubernetes**

1. Extiende el archivo YAML para incluir una etapa de despliegue a Kubernetes:

   ```yaml
   - stage: DeployKubernetes
     displayName: Deploy to Kubernetes
     dependsOn: Build
     jobs:
       - job: DeployK8sJob
         displayName: Deploy ASP.NET Core to Kubernetes
         steps:
           - task: Kubernetes@1
             inputs:
               connectionType: 'Azure Resource Manager'
               azureSubscription: '<AzureSubscription>'
               kubernetesCluster: '<AKSClusterName>'
               namespace: 'default'
               command: 'apply'
               useConfigurationFile: true
               configuration: 'manifests/*.yaml'
   ```

2. Crear los manifiestos de Kubernetes en una carpeta `manifests`:

   **Deployment:**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: myapp
     template:
       metadata:
         labels:
           app: myapp
       spec:
         containers:
         - name: myapp
           image: myregistry.azurecr.io/myapp:latest
           ports:
           - containerPort: 80
   ```

   **Service:**
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: myapp-service
   spec:
     selector:
       app: myapp
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
     type: LoadBalancer
   ```

3. Configura un Azure Kubernetes Service (AKS) o usa Minikube:
   - Para AKS: Asegúrate de que esté configurado con un Azure Container Registry (ACR).
   - Para Minikube: Usa un registro Docker local.

**Resultado esperado:**
- La aplicación se despliega en un clúster de Kubernetes.

---

#### **3. Pruebas y validación**

1. **Pruebas en IIS:**
   - Accede a la URL del sitio web configurado en IIS.
   - Verifica que la aplicación funcione correctamente.

2. **Pruebas en Azure App Service:**
   - Accede a la URL del App Service proporcionada en el portal de Azure.
   - Valida que la aplicación responda correctamente.

3. **Pruebas en Kubernetes:**
   - Obtén la IP del `Service` de Kubernetes:
     ```bash
     kubectl get service myapp-service
     ```
   - Accede a la aplicación utilizando la IP pública.

---

#### **4. Iteraciones**

1. Realiza un cambio menor en el código de la aplicación (por ejemplo, modificar el texto de una vista).
2. Haz un commit en la rama principal para activar el pipeline.
3. Valida que el cambio se propague correctamente en los tres destinos.

---

#### **Conclusión**

Este laboratorio práctico te guía paso a paso para crear un pipeline de CD funcional y versátil para ASP.NET Core. Con despliegues a **IIS on-premises**, **Azure App Service**, y **Kubernetes**, estás preparado para manejar una variedad de escenarios modernos en CI/CD.