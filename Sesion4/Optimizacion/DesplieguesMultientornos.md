### Automatización de despliegues en múltiples entornos

La automatización de despliegues en múltiples entornos es fundamental en los pipelines de CI/CD para garantizar la entrega continua de aplicaciones de manera segura, consistente y eficiente. En Azure DevOps, se pueden automatizar despliegues a entornos como **Desarrollo**, **Testing**, **Producción**, entre otros, utilizando **Stages**, **Environments** y **Variables**.

---

#### 1. **Definición de Stages para múltiples entornos**  

Azure Pipelines permite definir múltiples etapas (**stages**) dentro de un pipeline para desplegar en distintos entornos de forma secuencial o paralela.

- **Ejemplo básico con múltiples stages**:  
   ```yaml
   trigger:
     - main

   stages:
     - stage: Development
       displayName: "Despliegue en Desarrollo"
       jobs:
         - job: DeployDev
           pool:
             vmImage: 'ubuntu-latest'
           steps:
             - script: echo "Desplegando en el entorno de Desarrollo"
               displayName: "Deploy en Desarrollo"

     - stage: Testing
       displayName: "Despliegue en Testing"
       dependsOn: Development
       jobs:
         - job: DeployTest
           pool:
             vmImage: 'ubuntu-latest'
           steps:
             - script: echo "Desplegando en el entorno de Testing"
               displayName: "Deploy en Testing"

     - stage: Production
       displayName: "Despliegue en Producción"
       dependsOn: Testing
       jobs:
         - job: DeployProd
           pool:
             vmImage: 'ubuntu-latest'
           steps:
             - script: echo "Desplegando en el entorno de Producción"
               displayName: "Deploy en Producción"
   ```

   **Explicación**:  
   - Se definen tres stages: `Development`, `Testing` y `Production`.  
   - El stage de Testing depende del stage de Development, y Production depende de Testing.  
   - Los trabajos (`jobs`) dentro de cada stage contienen los pasos para ejecutar el despliegue.

---

#### 2. **Uso de Environments en Azure DevOps**  

Los **Environments** en Azure DevOps permiten gestionar y proteger los despliegues a entornos específicos, además de habilitar aprobaciones manuales.

- **Configuración de un Environment en un pipeline**:  
   ```yaml
   stages:
     - stage: Development
       jobs:
         - deployment: DeployDev
           environment: 'Development'
           strategy:
             runOnce:
               deploy:
                 steps:
                   - script: echo "Desplegando en Desarrollo"

     - stage: Production
       jobs:
         - deployment: DeployProd
           environment: 'Production'
           strategy:
             runOnce:
               deploy:
                 steps:
                   - script: echo "Desplegando en Producción"
   ```

   **Explicación**:  
   - La clave `environment` define el nombre del entorno (`Development` y `Production`).  
   - La estrategia `runOnce` se utiliza para realizar un despliegue único.  
   - Los Environments permiten configurar aprobaciones manuales antes de realizar un despliegue, asegurando mayor control.

---

#### 3. **Gestión de variables por entorno**  

Para configurar parámetros específicos según el entorno, se pueden utilizar **variables** o **variable groups**.

- **Definición de variables por stage**:  
   ```yaml
   variables:
     environment: 'default'

   stages:
     - stage: Development
       variables:
         environment: 'dev'
       jobs:
         - script: echo "Desplegando en ${{ variables.environment }}"

     - stage: Production
       variables:
         environment: 'prod'
       jobs:
         - script: echo "Desplegando en ${{ variables.environment }}"
   ```

- **Uso de grupos de variables** (Azure DevOps Portal):  
   1. Crea un grupo de variables en el portal.  
   2. Enlázalo en el pipeline:  
      ```yaml
      variables:
        - group: 'Production_Variables'
      ```

---

#### 4. **Despliegue condicional**  

Es posible configurar **condiciones** para ejecutar despliegues en entornos específicos, por ejemplo, desplegar solo en Producción cuando el branch es `main`.

- **Ejemplo de despliegue condicional**:  
   ```yaml
   stages:
     - stage: Production
       condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
       jobs:
         - script: echo "Desplegando solo en Producción desde main"
   ```

   **Explicación**:  
   - La condición utiliza `Build.SourceBranch` para verificar que el branch es `main`.  
   - El despliegue solo se realiza si la condición se cumple.

---

#### 5. **Aprobaciones y gates (puertas)**  

Azure DevOps permite configurar **aprobaciones manuales** y **gates** para garantizar que los despliegues solo se ejecuten cuando se cumplan ciertos criterios.

- **Configuración de aprobaciones**:  
   1. Accede a los **Environments** en Azure DevOps.  
   2. Configura aprobaciones manuales para el entorno de Producción.  

---

#### 6. **Despliegues automáticos en IIS**  

Para entornos Windows con IIS, se puede automatizar el despliegue utilizando la tarea `IIS Web App Deployment`.

- **Ejemplo en YAML**:  
   ```yaml
   steps:
     - task: IISWebAppDeploymentOnMachineGroup@0
       inputs:
         WebSiteName: 'MyWebsite'
         Package: '$(System.DefaultWorkingDirectory)/**/*.zip'
         RemoveAdditionalFilesFlag: true
         TakeAppOfflineFlag: true
   ```

   **Parámetros clave**:  
   - `WebSiteName`: Nombre del sitio en IIS.  
   - `Package`: Ruta del paquete de despliegue (ZIP).  
   - `RemoveAdditionalFilesFlag`: Elimina archivos adicionales del destino.  

---

#### 7. **Buenas prácticas para despliegues en múltiples entornos**  

1. **Separación clara de stages**: define etapas independientes para cada entorno.  
2. **Aprobaciones manuales**: habilita aprobaciones en entornos críticos como Producción.  
3. **Uso de variables**: centraliza configuraciones específicas en grupos de variables.  
4. **Monitoreo de despliegues**: implementa herramientas como **Application Insights** para monitorear el desempeño.  
5. **Rollback automático**: configura pasos de rollback en caso de fallos en el despliegue.  

---

Con estas prácticas y configuraciones, los despliegues automatizados en múltiples entornos se realizan de forma controlada, segura y eficiente, mejorando el flujo de CI/CD en Azure DevOps.