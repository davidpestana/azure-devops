### Tareas de despliegue en distintos entornos

En un pipeline de entrega continua (CD), las tareas de despliegue varían según el entorno (desarrollo, pruebas, producción) y las tecnologías involucradas. Es fundamental garantizar que cada entorno esté configurado correctamente, incluyendo variables de configuración, permisos y herramientas específicas.

---

#### **1. Estructura del pipeline por entornos**

1. **Separación de etapas por entorno:**
   - Configurar una etapa (stage) para cada entorno (por ejemplo, `Development`, `Testing`, `Production`).
   - Utilizar condiciones para ejecutar etapas específicas basadas en ramas o aprobaciones manuales.

   Ejemplo en YAML:
   ```yaml
   stages:
     - stage: Development
       jobs:
         - job: DeployToDev
           steps:
             - script: echo "Desplegando en Development"
             - task: AzureWebApp@1
               inputs:
                 azureSubscription: '<Subscription>'
                 appName: '<AppName-Dev>'
                 package: '$(Pipeline.Workspace)/drop/*.zip'

     - stage: Testing
       dependsOn: Development
       jobs:
         - job: DeployToTest
           steps:
             - script: echo "Desplegando en Testing"
             - task: AzureWebApp@1
               inputs:
                 azureSubscription: '<Subscription>'
                 appName: '<AppName-Test>'
                 package: '$(Pipeline.Workspace)/drop/*.zip'

     - stage: Production
       dependsOn: Testing
       jobs:
         - deployment: DeployToProd
           environment: 'Production'
           strategy:
             runOnce:
               deploy:
                 steps:
                   - script: echo "Desplegando en Producción"
                   - task: AzureWebApp@1
                     inputs:
                       azureSubscription: '<Subscription>'
                       appName: '<AppName-Prod>'
                       package: '$(Pipeline.Workspace)/drop/*.zip'
   ```

---

#### **2. Configuración específica por entorno**

1. **Variables de entorno:**
   - Definir variables como `CONNECTION_STRING`, `API_KEY`, o `APPINSIGHTS_KEY` para cada entorno.
   - Utilizar **Variable Groups** en Azure DevOps para centralizar la configuración.

   Ejemplo de variables en YAML:
   ```yaml
   variables:
     - group: 'DevelopmentVariables'
     - name: Environment
       value: 'Development'
   ```

2. **Seguridad y secretos:**
   - Almacenar secretos en Azure Key Vault o en las variables protegidas del pipeline.
   - Configurar acceso mediante `Managed Identity` o una conexión segura.

3. **Configuración de infraestructura:**
   - Validar que los recursos necesarios (como bases de datos, cuentas de almacenamiento o clústeres) estén configurados en cada entorno.
   - Utilizar herramientas como Terraform o ARM templates para aprovisionamiento automatizado.

---

#### **3. Tipos de tareas de despliegue**

1. **Despliegue a App Service:**
   - Ideal para aplicaciones web en Azure.
   - Tarea: `AzureWebApp@1`.

2. **Despliegue a Kubernetes:**
   - Requiere manifiestos YAML o Helm charts.
   - Tarea: `Kubernetes@1`.

3. **Despliegue a máquinas virtuales (on-premises o cloud):**
   - Usar SSH para copiar artefactos y ejecutar comandos de despliegue.
   - Tarea: `SSH@0`.

4. **Despliegue a Azure Functions:**
   - Publicación directa desde un paquete `.zip` o `dll`.
   - Tarea: `AzureFunctionApp@1`.

5. **Ejecutar scripts específicos:**
   - Configurar scripts para ajustes adicionales, como reiniciar servicios o limpiar caché.
   - Tarea: `Bash@3` (Linux) o `PowerShell@2` (Windows).

---

#### **4. Ejemplo de flujo completo**

Un ejemplo de pipeline con tareas de despliegue en App Service y Kubernetes:

```yaml
stages:
  - stage: Development
    jobs:
      - job: DeployToAppService
        steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: '<DevSubscription>'
              appName: '<AppService-Dev>'
              package: '$(Pipeline.Workspace)/drop/*.zip'

  - stage: Testing
    dependsOn: Development
    jobs:
      - job: DeployToKubernetes
        steps:
          - task: Kubernetes@1
            inputs:
              connectionType: 'Azure Resource Manager'
              azureSubscription: '<TestSubscription>'
              kubernetesCluster: '<AKSCluster>'
              namespace: 'testing'
              command: 'apply'
              useConfigurationFile: true
              configuration: 'k8s/*.yaml'

  - stage: Production
    dependsOn: Testing
    jobs:
      - deployment: DeployToProd
        environment: 'Production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: '<ProdSubscription>'
                    appName: '<AppService-Prod>'
                    package: '$(Pipeline.Workspace)/drop/*.zip'
```

---

#### **5. Prácticas recomendadas**

1. **Aprobaciones manuales:**
   - Configurar aprobaciones obligatorias antes de ejecutar la etapa de producción.

2. **Pruebas automatizadas:**
   - Ejecutar pruebas de integración, rendimiento y regresión antes de pasar a entornos superiores.

3. **Estrategias de despliegue:**
   - Utilizar estrategias como Canary, Blue-Green o Rolling Updates para minimizar riesgos.

4. **Monitoreo post-despliegue:**
   - Configurar herramientas como Application Insights o Prometheus para identificar problemas tras el despliegue.

---

Estas tareas aseguran un despliegue confiable y repetible en múltiples entornos, reduciendo riesgos y facilitando la gestión de cambios en aplicaciones complejas.