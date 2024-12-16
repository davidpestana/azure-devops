### Laboratorio: Añadir monitorización con Application Insights

En este laboratorio, aprenderás a integrar **Application Insights** en una aplicación desplegada y configurada a través de un pipeline de Azure DevOps. Esto permitirá realizar un **seguimiento en tiempo real** del rendimiento, errores y telemetría de la aplicación.

---

#### **Objetivo del laboratorio**

- Configurar **Application Insights** en una aplicación existente.
- Automatizar la integración en el pipeline de CI/CD.
- Visualizar métricas y trazas en el portal de Azure.

---

#### **Requisitos previos**

1. Cuenta en **Azure** con permisos para crear recursos.
2. Clave de **Application Insights** (se obtendrá durante el laboratorio).
3. Aplicación .NET Core (o cualquier otra aplicación) preparada para el despliegue.
4. Pipeline en Azure DevOps configurado previamente.

---

### **Pasos del laboratorio**

---

#### 1. **Crear un recurso de Application Insights en Azure**

1. Accede al portal de Azure: [https://portal.azure.com](https://portal.azure.com).  
2. En el menú, selecciona **Create a resource** → **Application Insights**.  
3. Completa la configuración:  
   - **Name**: `MyAppInsights`.  
   - **Resource Group**: Usa un grupo existente o crea uno nuevo.  
   - **Application Type**: Selecciona **ASP.NET Core** (o según corresponda).  
   - **Location**: Selecciona la región más cercana.  
4. Haz clic en **Create**.  
5. Una vez creado, copia la **Clave de Instrumentación** (Instrumentation Key).

---

#### 2. **Configurar la aplicación para usar Application Insights**

Si trabajas con una aplicación **.NET Core**, sigue los siguientes pasos:

1. Instala el paquete **Microsoft.ApplicationInsights.AspNetCore**:  
   ```bash
   dotnet add package Microsoft.ApplicationInsights.AspNetCore
   ```

2. Añade la configuración de Application Insights en `Program.cs` o `Startup.cs`:  

   ```csharp
   using Microsoft.ApplicationInsights.Extensibility;

   var builder = WebApplication.CreateBuilder(args);

   // Configurar Application Insights
   builder.Services.AddApplicationInsightsTelemetry("INSTRUMENTATION_KEY");

   var app = builder.Build();

   app.MapGet("/", () => "Hello, world!");

   app.Run();
   ```

   **Nota**: Reemplaza `INSTRUMENTATION_KEY` con la clave obtenida en el paso anterior.

3. Ejecuta la aplicación localmente para validar que no existan errores.

---

#### 3. **Integrar Application Insights en el Pipeline de CI/CD**

Modifica tu pipeline YAML existente para automatizar la configuración de Application Insights.

- **Pipeline YAML actualizado**:  

   ```yaml
   trigger:
     - main

   variables:
     azureSubscription: 'MyAzureSubscription'
     appInsightsName: 'MyAppInsights'
     resourceGroup: 'MyResourceGroup'
     webAppName: 'MyWebApp'

   stages:
     - stage: Deploy
       displayName: "Despliegue en Desarrollo"
       jobs:
         - deployment: DeployToDev
           environment: Development
           pool:
             vmImage: 'ubuntu-latest'
           strategy:
             runOnce:
               deploy:
                 steps:
                   # Despliegue en Azure Web App
                   - task: AzureWebApp@1
                     inputs:
                       azureSubscription: $(azureSubscription)
                       appType: 'webApp'
                       appName: $(webAppName)
                       package: '$(Pipeline.Workspace)/drop/**/*.zip'
                     displayName: "Desplegar aplicación en Azure"

                   # Configurar Application Insights
                   - task: AzureCLI@2
                     inputs:
                       azureSubscription: $(azureSubscription)
                       scriptType: 'bash'
                       scriptLocation: 'inlineScript'
                       inlineScript: |
                         echo "Configurando Application Insights..."
                         az monitor app-insights component update \
                           --app $(appInsightsName) \
                           --resource-group $(resourceGroup) \
                           --application-type web
   ```

   **Explicación**:
   - `AzureWebApp@1`: Despliega la aplicación en Azure Web Apps.
   - `AzureCLI@2`: Configura Application Insights automáticamente mediante la CLI de Azure.

---

#### 4. **Verificar la integración**

1. Ejecuta el pipeline desde Azure DevOps.  
2. Una vez completado el despliegue:  
   - Accede al portal de Azure.  
   - Abre el recurso **Application Insights**.  
   - Ve a la sección **Live Metrics** o **Performance** para ver las métricas en tiempo real.  
3. Genera tráfico en la aplicación (por ejemplo, abre la URL desplegada en un navegador).  
4. Comprueba que las siguientes métricas estén disponibles:  
   - **Solicitudes** (número de peticiones).  
   - **Tiempo de respuesta**.  
   - **Errores**.  
   - **Trazas**.

---

#### 5. **Crear alertas en Application Insights**

Configura alertas para recibir notificaciones cuando ocurran problemas:

1. En el portal de Azure, abre **Application Insights**.  
2. Selecciona **Alerts** → **+ New Alert Rule**.  
3. Configura una condición, por ejemplo:  
   - **Tipo**: Disponibilidad.  
   - **Regla**: "Tiempo de respuesta > 2 segundos".  
4. Selecciona una **acción** para recibir notificaciones (correo electrónico, webhook, etc.).

---

### **Resultado final**

Al finalizar este laboratorio:  
- Tu aplicación estará conectada a **Application Insights**.  
- El pipeline automatizará la configuración de la monitorización en cada despliegue.  
- Podrás visualizar métricas, errores y trazas en el portal de Azure en tiempo real.

---

**Próximos pasos**:  
- Configurar **Dashboards** en Azure para visualizar las métricas clave.  
- Integrar alertas avanzadas y análisis continuo del rendimiento.