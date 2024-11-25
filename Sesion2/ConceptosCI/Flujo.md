# Flujo de Trabajo para ASP.NET Core y ASP.NET Framework en Azure DevOps

El desarrollo de aplicaciones ASP.NET Core y ASP.NET Framework sigue un flujo de trabajo estructurado que abarca desde la escritura del código hasta el despliegue en entornos de producción. Azure DevOps proporciona herramientas integradas para gestionar este flujo, incluyendo integración continua (CI), entrega continua (CD) y pruebas automatizadas.

Este documento explica el flujo para ambas tecnologías, destacando similitudes y diferencias.

---

## **ASP.NET Core vs. ASP.NET Framework: Breve Introducción**
1. **ASP.NET Core**:
   - Multiplataforma (Windows, macOS, Linux).
   - Diseñado para aplicaciones modernas y contenerizadas.
   - Basado en .NET Core, con mejoras en rendimiento y flexibilidad.

2. **ASP.NET Framework**:
   - Limitado a Windows.
   - Ideal para aplicaciones empresariales existentes.
   - Utiliza tecnologías como Web Forms, MVC y Web API.

---

## **Flujo de Trabajo General**

El flujo para ASP.NET Core y ASP.NET Framework incluye las siguientes etapas:

### **1. Escribir el Código**
   - **ASP.NET Core**:
     - Utiliza un enfoque modular basado en dependencias específicas.
     - Organización típica:
       ```
       /Controllers
       /Views
       /Models
       appsettings.json
       Program.cs
       ```
   - **ASP.NET Framework**:
     - Utiliza estructuras tradicionales como MVC o Web Forms.
     - Organización típica:
       ```
       /Controllers
       /Views
       /Models
       Web.config
       Global.asax
       ```

**Diferencia clave**: ASP.NET Core utiliza `appsettings.json` para configuración, mientras que ASP.NET Framework usa `Web.config`.

---

### **2. Configuración del Proyecto**

- **ASP.NET Core**:
  - Configura servicios en `Program.cs` y `Startup.cs`.
  - Ejemplo de configuración:
    ```csharp
    var builder = WebApplication.CreateBuilder(args);
    builder.Services.AddControllersWithViews();
    var app = builder.Build();
    app.UseStaticFiles();
    app.MapDefaultControllerRoute();
    app.Run();
    ```

- **ASP.NET Framework**:
  - Configura rutas y dependencias en `Global.asax`.
  - Ejemplo:
    ```csharp
    protected void Application_Start()
    {
        AreaRegistration.RegisterAllAreas();
        RouteConfig.RegisterRoutes(RouteTable.Routes);
    }
    ```

---

### **3. Gestión del Código Fuente**

1. **Repositorio Git**:
   - Usa Azure Repos o GitHub para gestionar el código.
   - Crea ramas para desarrollo (`develop`) y características (`feature/feature-name`).
   - Realiza commits frecuentes.

   **Por qué**:
   - Permite a los equipos colaborar sin conflictos y mantener un historial de cambios.

---

### **4. Construcción (Build)**

1. **ASP.NET Core**:
   - Usa el comando `dotnet build` para compilar el código.
   - Ejemplo en Azure Pipelines:
     ```yaml
     - script: dotnet build --configuration Release
       displayName: 'Build ASP.NET Core Application'
     ```

2. **ASP.NET Framework**:
   - Utiliza MSBuild para compilar el proyecto.
   - Ejemplo en Azure Pipelines:
     ```yaml
     - task: VSBuild@1
       inputs:
         solution: '**/*.sln'
         msbuildArgs: '/p:Configuration=Release'
       displayName: 'Build ASP.NET Framework Application'
     ```

**Diferencia clave**: ASP.NET Core utiliza `dotnet CLI`, mientras que ASP.NET Framework depende de Visual Studio y MSBuild.

---

### **5. Pruebas Automatizadas**

- **ASP.NET Core**:
  - Compatible con herramientas modernas como `xUnit`, `MSTest` y `NUnit`.
  - Ejemplo en un pipeline:
    ```yaml
    - script: dotnet test --configuration Release
      displayName: 'Run Tests'
    ```

- **ASP.NET Framework**:
  - Utiliza `MSTest` o bibliotecas compatibles con Visual Studio.
  - Ejemplo en un pipeline:
    ```yaml
    - task: VSTest@2
      inputs:
        testSelector: 'TestAssemblies'
        testAssemblyVer2: '**\*test*.dll'
      displayName: 'Run Tests for ASP.NET Framework'
    ```

**Diferencia clave**: ASP.NET Core usa bibliotecas multiplataforma y CLI; ASP.NET Framework está más integrado con Visual Studio.

---

### **6. Generación de Artefactos**

1. **¿Por qué?**
   - Los artefactos son los binarios generados que se despliegan en entornos como servidores o la nube.

2. **ASP.NET Core**:
   - Usa el comando `dotnet publish` para generar artefactos:
     ```yaml
     - script: dotnet publish --configuration Release --output $(Build.ArtifactStagingDirectory)
       displayName: 'Publish ASP.NET Core Application'
     ```

3. **ASP.NET Framework**:
   - MSBuild empaqueta la aplicación en carpetas listas para despliegue:
     ```yaml
     - task: CopyFiles@2
       inputs:
         sourceFolder: '$(Build.SourcesDirectory)\bin\Release'
         targetFolder: '$(Build.ArtifactStagingDirectory)'
       displayName: 'Copy ASP.NET Framework Artifacts'
     ```

---

### **7. Despliegue (Deployment)**

- **ASP.NET Core**:
  - Compatible con:
    - Contenedores Docker.
    - Azure App Service.
    - Kubernetes.
  - Ejemplo:
    ```yaml
    - task: AzureWebApp@1
      inputs:
        azureSubscription: 'MyAzureServiceConnection'
        appName: 'DemoAppService'
        package: '$(Build.ArtifactStagingDirectory)'
    ```

- **ASP.NET Framework**:
  - Usualmente se despliega en:
    - Servidores IIS (on-premises).
    - Azure App Service.
  - Ejemplo:
    ```yaml
    - task: IISWebAppDeploymentOnMachineGroup@0
      inputs:
        WebSiteName: 'Default Web Site/MyApp'
        Package: '$(Build.ArtifactStagingDirectory)\MyApp.zip'
    ```

---

### **8. Monitoreo**

- **ASP.NET Core y Framework**:
  - Usa **Application Insights** para monitorear métricas como:
    - Rendimiento.
    - Solicitudes HTTP.
    - Errores y excepciones.
  - Ejemplo en un pipeline:
    ```yaml
    - task: AzureAppInsights@1
      inputs:
        ApplicationInsightsResourceGroup: 'MyResourceGroup'
        ApplicationInsightsName: 'MyAppInsights'
    ```

---

## **Resumen del Flujo**

| Etapa             | ASP.NET Core                          | ASP.NET Framework                |
|--------------------|---------------------------------------|-----------------------------------|
| Código fuente      | `dotnet CLI`                         | Visual Studio/MSBuild            |
| Construcción       | `dotnet build`                       | MSBuild                          |
| Pruebas            | `dotnet test`                        | VSTest                           |
| Artefactos         | `dotnet publish`                     | MSBuild/CopyFiles                |
| Despliegue         | Docker, Azure, Kubernetes            | IIS, Azure App Service           |
| Configuración      | `appsettings.json`                   | `Web.config`                     |

Ambos flujos comparten principios fundamentales, pero difieren en herramientas y dependencias según el framework utilizado. Esto asegura que cada proyecto aproveche las mejores prácticas para su plataforma.