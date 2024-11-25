# Uso de Agentes de Build para Windows y Azure

Los **agentes de build** son componentes fundamentales en el proceso de integración y entrega continua (CI/CD) en **Azure DevOps**. Son responsables de ejecutar los trabajos definidos en los pipelines, como la construcción, pruebas y despliegue de aplicaciones. Dependiendo del entorno, puedes usar **agentes hospedados por Microsoft** (para tareas en la nube) o **agentes auto-hospedados** (para tareas locales o personalizadas).

---

## **Tipos de Agentes de Build**

### **1. Agentes Hospedados por Microsoft**
- Son agentes preconfigurados en la infraestructura de Microsoft.
- Compatibles con múltiples sistemas operativos: Windows, Linux, y macOS.
- Recomendados para proyectos en la nube, como aplicaciones en **Azure App Service** o **Azure Kubernetes Service**.

### **2. Agentes Auto-Hospedados**
- Son agentes configurados manualmente por los usuarios en sus propios servidores o máquinas virtuales.
- Ideales para tareas que requieren acceso a recursos locales, como bases de datos on-premises o servidores IIS.

---

## **Uso de Agentes Hospedados por Microsoft**

### **Ventajas**
1. Configuración rápida: No necesitas instalar ni mantener software.
2. Compatibilidad preinstalada: Vienen con herramientas como .NET SDK, Node.js, Python, y Docker.
3. Escalabilidad: Microsoft proporciona agentes adicionales automáticamente si tienes múltiples builds.

### **Configuración en el Pipeline**

1. **Seleccionar un Agente Windows Hospedado**
   - En el archivo YAML, especifica el pool `windows-latest` o una versión específica.
   ```yaml
   pool:
     vmImage: 'windows-latest'
   ```

2. **Pipeline Básico con un Agente Windows**
   - Ejemplo para construir una aplicación .NET Core:
     ```yaml
     trigger:
       branches:
         include:
           - main

     pool:
       vmImage: 'windows-latest'

     steps:
     - task: UseDotNet@2
       inputs:
         packageType: 'sdk'
         version: '6.0'

     - script: dotnet restore
       displayName: 'Restore Dependencies'

     - script: dotnet build --configuration Release
       displayName: 'Build Application'
     ```

---

## **Uso de Agentes Auto-Hospedados**

### **Ventajas**
1. Control total: Puedes instalar herramientas personalizadas y ajustar configuraciones específicas.
2. Acceso a recursos locales: Conectan Azure DevOps con servidores, bases de datos y archivos en tu red.
3. Reducción de costos: Útil si ya tienes infraestructura disponible y quieres evitar costos por minutos adicionales de agentes hospedados.

### **Pasos para Configurar un Agente Auto-Hospedado**

#### **1. Crear un Pool de Agentes**
1. Ve a **Configuración de la Organización** en Azure DevOps.
2. Haz clic en **Pools de Agentes** → **Nuevo Pool de Agentes**.
3. Asigna un nombre descriptivo, como `OnPremisesWindows`.

#### **2. Descargar el Software del Agente**
1. Ve a **Configuración del Proyecto** → **Agentes**.
2. Selecciona **Nuevo Agente** y descarga el instalador para Windows.

#### **3. Instalar y Registrar el Agente**
1. Extrae el archivo descargado en tu servidor local.
2. Abre una terminal como administrador y ejecuta el script de configuración:
   ```bash
   config.cmd --unattended --url https://dev.azure.com/your-organization --auth pat --token <personal-access-token> --pool OnPremisesWindows --agent MyAgent
   ```

#### **4. Verificar el Agente**
- Una vez registrado, el agente aparecerá en el pool configurado en Azure DevOps.

---

### **Usar un Agente Auto-Hospedado en el Pipeline**

1. Configura el pipeline para usar el pool personalizado:
   ```yaml
   pool:
     name: OnPremisesWindows
   ```

2. Ejemplo de pipeline:
   ```yaml
   trigger:
     branches:
       include:
         - main

   pool:
     name: OnPremisesWindows

   steps:
   - script: echo "Building application on an auto-hosted agent"
     displayName: 'Build Step'
   ```

---

## **Comparativa: Agentes Hospedados vs. Auto-Hospedados**

| Característica               | Agentes Hospedados por Microsoft | Agentes Auto-Hospedados         |
|------------------------------|-----------------------------------|----------------------------------|
| **Configuración**            | Preconfigurados                  | Requiere instalación manual     |
| **Acceso a Recursos Locales**| No                               | Sí                              |
| **Flexibilidad**             | Limitada a las herramientas preinstaladas | Completa (puedes instalar software personalizado) |
| **Costo**                    | Pago por minuto adicional        | Costos locales                  |
| **Escalabilidad**            | Automática                       | Manual                          |

---

## **Buenas Prácticas para Usar Agentes de Build**

1. **Optimizar Tiempos de Ejecución**:
   - En agentes hospedados, selecciona la imagen adecuada (`windows-latest`, `ubuntu-latest`, etc.) para reducir el tiempo de inicialización.
   - Usa `caching` para restaurar dependencias más rápido:
     ```yaml
     - task: Cache@2
       inputs:
         key: 'nuget | "$(Agent.OS)" | **/packages.lock.json'
         restoreKeys: 'nuget | "$(Agent.OS)"'
         path: ~/.nuget/packages
     ```

2. **Seguridad en Agentes Auto-Hospedados**:
   - Asegúrate de que las máquinas tengan actualizaciones y parches aplicados.
   - Limita los permisos de red y acceso solo a lo estrictamente necesario.

3. **Revisar Logs del Agente**:
   - Inspecciona los logs para detectar problemas recurrentes.
   - Logs locales en agentes auto-hospedados: `Agent.WorkDirectory/_diag/`.

---

## **Ejemplo: Pipeline Híbrido para Windows y Azure**

Este pipeline usa un agente hospedado para construcción y pruebas, y un agente auto-hospedado para despliegues on-premises.

```yaml
jobs:
- job: BuildAndTest
  pool:
    vmImage: 'windows-latest'
  steps:
  - script: dotnet restore
    displayName: 'Restore Dependencies'
  - script: dotnet build --configuration Release
    displayName: 'Build Application'
  - script: dotnet test --configuration Release
    displayName: 'Run Tests'

- job: DeployOnPremises
  dependsOn: BuildAndTest
  pool:
    name: OnPremisesWindows
  steps:
  - script: echo "Deploying application on local IIS server..."
    displayName: 'Deploy to IIS'
  ```

---

Con estos conceptos y ejemplos, puedes configurar agentes de build en Azure DevOps tanto para entornos en la nube como locales, optimizando tu flujo de trabajo CI/CD.