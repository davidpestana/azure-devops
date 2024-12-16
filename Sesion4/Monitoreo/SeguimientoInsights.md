### Seguimiento con Application Insights

**Application Insights** es un servicio de **Azure Monitor** que permite realizar monitoreo avanzado de aplicaciones en tiempo real. Proporciona métricas, trazas, registros y diagnósticos que ayudan a identificar problemas de rendimiento, errores y comportamientos inesperados en las aplicaciones desplegadas a través de Azure DevOps.

---

#### 1. **¿Por qué usar Application Insights?**  

- Monitoreo continuo del rendimiento y disponibilidad de la aplicación.
- Captura de excepciones y errores críticos.
- Trazas detalladas del flujo de ejecución.
- Análisis del uso y comportamiento del usuario.
- Alertas y diagnósticos proactivos.

---

#### 2. **Configuración de Application Insights en la aplicación**  

Antes de integrarlo con un pipeline, es necesario agregar Application Insights a la aplicación.

**.NET Core / .NET**  
1. Instala el paquete de NuGet:  
   ```bash
   dotnet add package Microsoft.ApplicationInsights.AspNetCore
   ```

2. Configura Application Insights en `Program.cs` o `Startup.cs`:  
   ```csharp
   using Microsoft.ApplicationInsights.Extensibility;

   var builder = WebApplication.CreateBuilder(args);
   builder.Services.AddApplicationInsightsTelemetry("INSTRUMENTATION_KEY");

   var app = builder.Build();
   app.Run();
   ```

   - Reemplaza `INSTRUMENTATION_KEY` con tu clave de Instrumentación de Application Insights.

---

#### 3. **Integración en un pipeline de Azure DevOps**  

Una vez configurada la aplicación, se puede automatizar la configuración de Application Insights como parte del pipeline de CI/CD.

- **Ejemplo de pipeline con despliegue y monitoreo**:  

   ```yaml
   trigger:
     - main

   variables:
     azureSubscription: 'MyAzureSubscription'
     resourceGroup: 'MyResourceGroup'
     appInsightsName: 'MyAppInsights'
     webAppName: 'MyWebApp'

   stages:
     - stage: Build
       jobs:
         - job: BuildJob
           pool:
             vmImage: 'ubuntu-latest'
           steps:
             - script: |
                 echo "Compilando la aplicación..."
                 dotnet build --configuration Release
               displayName: "Compilación"

     - stage: Deploy
       dependsOn: Build
       jobs:
         - deployment: DeployApp
           environment: 'Production'
           pool:
             vmImage: 'windows-latest'
           strategy:
             runOnce:
               deploy:
                 steps:
                   - task: AzureWebApp@1
                     inputs:
                       azureSubscription: $(azureSubscription)
                       appType: 'webApp'
                       appName: $(webAppName)
                       package: '$(System.DefaultWorkingDirectory)/**/*.zip'
                       deploymentMethod: 'auto'

                   - task: AzureCLI@2
                     inputs:
                       azureSubscription: $(azureSubscription)
                       scriptType: 'ps'
                       scriptLocation: 'inlineScript'
                       inlineScript: |
                         echo "Configurando Application Insights..."
                         az monitor app-insights component update `
                         --app $(appInsightsName) `
                         --resource-group $(resourceGroup) `
                         --application-type web
   ```

   **Explicación**:  
   - `AzureWebApp@1`: Despliega la aplicación en Azure Web Apps.  
   - `AzureCLI@2`: Configura Application Insights utilizando la CLI de Azure.  
   - Las variables `azureSubscription`, `resourceGroup`, y `appInsightsName` centralizan la configuración.

---

#### 4. **Monitoreo de métricas y trazas**  

Una vez integrada la aplicación con Application Insights, las siguientes métricas estarán disponibles en el portal de Azure:

- **Rendimiento**:  
   - Tiempo de respuesta.  
   - Número de solicitudes procesadas.  

- **Errores**:  
   - Excepciones no controladas.  
   - Errores HTTP 4xx y 5xx.  

- **Registros**:  
   - Trazas personalizadas generadas desde la aplicación.  

**Ejemplo de traza en .NET**:  
```csharp
var telemetryClient = new TelemetryClient();
telemetryClient.TrackTrace("Inicio del proceso de despliegue");
```

---

#### 5. **Alertas y diagnósticos**  

Application Insights permite configurar alertas automáticas en función de los datos capturados:

1. En el portal de Azure, selecciona **Application Insights**.
2. Ve a **Alerts** → **+ New Alert Rule**.
3. Configura las condiciones (p. ej., tiempo de respuesta mayor a 2s, errores > 10%).  
4. Define las acciones, como notificaciones por correo o integraciones con servicios externos.

---

#### 6. **Visualización y análisis de datos**  

1. **Live Metrics**: Monitorea métricas en tiempo real.  
2. **Performance Overview**: Gráficos de rendimiento de solicitudes.  
3. **Failures**: Panel de control para revisar errores y excepciones.  
4. **Kusto Query Language (KQL)**: Realiza consultas personalizadas para analizar los registros:  
   ```kusto
   requests
   | where success == false
   | summarize count() by operation_Name
   ```

---

#### 7. **Buenas prácticas**  

1. **Centraliza la configuración**: usa **Variables Groups** en Azure DevOps para la clave de instrumentación.  
2. **Configura alertas**: define alertas proactivas para detectar problemas.  
3. **Minimiza el ruido**: filtra trazas y errores no relevantes para evitar saturar los registros.  
4. **Integra con DevOps**: combina Application Insights con Azure Boards para crear automáticamente **work items** a partir de errores críticos.

---

Con **Application Insights**, podrás realizar un monitoreo continuo y obtener visibilidad completa del rendimiento, errores y comportamiento de las aplicaciones desplegadas. Esto facilita la detección y solución rápida de problemas en los pipelines de CI/CD.