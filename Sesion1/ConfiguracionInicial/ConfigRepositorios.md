# Configuración de Repositorios para .NET en Azure DevOps

Una correcta configuración de repositorios es clave para gestionar eficazmente el código fuente de aplicaciones .NET en **Azure DevOps**. Este proceso incluye la elección del sistema de control de versiones, la organización de ramas, la configuración de políticas y la integración con herramientas de desarrollo.

---

## Pasos para Configurar un Repositorio para .NET en Azure DevOps

### **1. Crear el Repositorio**

1. **Accede al proyecto**:
   - Entra al proyecto en Azure DevOps donde deseas configurar el repositorio.
2. **Navega a "Repos"**:
   - En el menú lateral, selecciona **"Repos"**.
3. **Crear un nuevo repositorio**:
   - Haz clic en **"Nuevo repositorio"**.
   - Selecciona las opciones:
     - **Tipo**: Git (recomendado) o TFVC.
     - **Nombre**: Asigna un nombre descriptivo como `MyDotNetApp`.
   - Crea el repositorio.

---

### **2. Clonar el Repositorio en el Entorno Local**

1. Copia la URL del repositorio:
   - En la pestaña **"Clonar"**, copia la URL HTTPS o SSH.
2. Abre tu terminal o Visual Studio.
3. Clona el repositorio usando Git:
   ```bash
   git clone <URL del repositorio>
   ```

---

### **3. Configurar un Proyecto .NET en el Repositorio**

1. **Crea un Proyecto en Visual Studio**:
   - Abre Visual Studio y selecciona **"Crear un nuevo proyecto"**.
   - Elige una plantilla adecuada (por ejemplo, **Aplicación web ASP.NET Core**).
   - Configura el nombre del proyecto y su ubicación en la carpeta del repositorio clonado.
   
2. **Inicializa el Proyecto**:
   - Visual Studio generará los archivos del proyecto, incluyendo:
     - `Program.cs` o `Startup.cs` (según la versión de .NET).
     - `appsettings.json` para configuraciones.

3. **Añade Archivos al Repositorio**:
   - Usa los siguientes comandos en la terminal para añadir y commitear los archivos iniciales:
     ```bash
     git add .
     git commit -m "Initial commit: Add .NET project"
     git push origin main
     ```

---

### **4. Configurar la Estrategia de Ramas**

#### **Rama Principal (Main)**
- Usada para código listo para producción.

#### **Ramas de Desarrollo (Dev)**
- Crea una rama `develop` para integrar características antes de fusionarlas en `main`.
  ```bash
  git checkout -b develop
  git push origin develop
  ```

#### **Ramas de Características (Feature Branches)**
- Para cada nueva funcionalidad o corrección de errores, crea ramas específicas:
  ```bash
  git checkout -b feature/new-feature
  ```

---

### **5. Configurar Políticas de Ramas**

1. **Accede a la Configuración de la Rama**:
   - Ve a **"Repos" → "Ramas"** en Azure DevOps.
   - Selecciona la rama sobre la que deseas configurar políticas (por ejemplo, `main`).

2. **Define las Políticas**:
   - **Aprobar Pull Requests**:
     - Requiere que las Pull Requests tengan revisores asignados.
   - **Compilaciones Válidas**:
     - Exige que las Pull Requests pasen las validaciones del pipeline antes de fusionarse.
   - **Protección contra Fuerza (Force Push)**:
     - Evita sobrescribir accidentalmente el historial de la rama.

---

### **6. Configurar el Pipeline para CI/CD**

1. **Añade un Pipeline YAML**:
   - En el repositorio, crea un archivo llamado `azure-pipelines.yml`.
   - Ejemplo básico para construir y probar una aplicación .NET Core:
     ```yaml
     trigger:
       branches:
         include:
           - main
           - develop

     pool:
       vmImage: 'windows-latest'

     steps:
     - task: UseDotNet@2
       inputs:
         packageType: 'sdk'
         version: '6.0.x'

     - script: dotnet restore
       displayName: 'Restore Dependencies'

     - script: dotnet build --configuration Release
       displayName: 'Build Application'

     - script: dotnet test --configuration Release
       displayName: 'Run Tests'
     ```

2. **Configura el Pipeline**:
   - Ve a **"Pipelines" → "Crear pipeline"**.
   - Selecciona el repositorio y confirma la configuración del archivo YAML.

---

### **7. Configurar Herramientas de Desarrollo**

#### **Visual Studio**
- **Conexión con Azure DevOps**:
  - Vincula tu cuenta de Azure DevOps en Visual Studio.
  - Accede al menú **"Equipo" → "Conectar a repositorio"**.

#### **Integración con Secretos**:
- Usa herramientas como **Azure Key Vault** para gestionar secretos como cadenas de conexión a bases de datos.
- Configura el acceso al Key Vault desde el código en `appsettings.json`:
  ```json
  {
    "KeyVault": {
      "VaultName": "mykeyvault",
      "ClientId": "your-client-id",
      "ClientSecret": "your-client-secret"
    }
  }
  ```

---

### **8. Monitoreo y Validación del Código**

#### **Integración con SonarCloud**
- Agrega análisis de calidad y cobertura del código.
- Configura un paso en el pipeline:
  ```yaml
  - task: SonarCloudPrepare@1
    inputs:
      SonarCloud: 'Sonar Service Connection'
      organization: 'your-org'
      scannerMode: 'CLI'
      configMode: 'manual'
      projectKey: 'your-project-key'
  ```

#### **Notificaciones Automáticas**:
- Configura alertas para cambios en ramas, fallos de compilación o conflictos.

---

## Buenas Prácticas en la Configuración de Repositorios .NET

1. **Estructurar el Código**:
   - Mantén carpetas claras (`Controllers`, `Services`, `Models`).
2. **Proteger la Rama Principal**:
   - Asegúrate de que solo los administradores puedan realizar cambios directos.
3. **Automatización**:
   - Usa pipelines para pruebas y despliegues automáticos.
4. **Documentación**:
   - Incluye un archivo `README.md` con instrucciones para configurar y ejecutar el proyecto.

---

Con estas configuraciones, el equipo tendrá un entorno eficiente para colaborar en proyectos .NET utilizando Azure DevOps, maximizando la calidad y la automatización en el flujo de trabajo.