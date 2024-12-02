### Configuración en Azure Functions

Azure Functions es un servicio serverless que permite ejecutar código bajo demanda, con escalabilidad automática y facturación basada en consumo. Es ideal para aplicaciones que necesitan manejar tareas pequeñas, eventos o integrarse con otros servicios.

---

#### **1. Preparativos iniciales**

1. **Crear una Function App:**
   - En Azure Portal, selecciona **Create a resource** y busca **Function App**.
   - Configura:
     - **Nombre:** Un identificador único para la aplicación.
     - **Plan de hosting:** Consumo (serverless) o Premium (si necesitas recursos dedicados).
     - **Runtime stack:** Selecciona `.NET` y la versión deseada.
     - **Región:** Escoge la ubicación más cercana a tus usuarios.

2. **Configurar el entorno local:**
   - Instala las herramientas necesarias:
     - **Azure Functions Core Tools:** Para ejecutar y depurar funciones localmente.
     - **.NET SDK:** Asegúrate de tener instalada la versión compatible con tu proyecto.
   - Crear un nuevo proyecto:
     ```bash
     func init MyFunctionApp --worker-runtime dotnet
     ```

3. **Autenticación con Azure:**
   - Configura una conexión con Azure utilizando la CLI:
     ```bash
     az login
     az account set --subscription <subscription_id>
     ```

---

#### **2. Crear una función**

1. **Añadir una nueva función:**
   - Dentro del proyecto, crea una función HTTP trigger:
     ```bash
     func new
     ```
   - Selecciona `HTTP trigger` y define un nombre para la función.

2. **Estructura del código:**
   Un ejemplo básico de una función HTTP trigger en `.NET`:
   ```csharp
   [FunctionName("MyHttpFunction")]
   public static async Task<IActionResult> Run(
       [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
       ILogger log)
   {
       log.LogInformation("HTTP trigger function processed a request.");
       string name = req.Query["name"];
       return name != null
           ? (ActionResult)new OkObjectResult($"Hello, {name}")
           : new BadRequestObjectResult("Please pass a name on the query string.");
   }
   ```

3. **Probar localmente:**
   - Ejecuta la aplicación:
     ```bash
     func start
     ```
   - Accede a la función desde el navegador en `http://localhost:7071/api/MyHttpFunction`.

---

#### **3. Configurar el pipeline de CI/CD en Azure DevOps**

##### YAML para despliegue:

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

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

  - task: AzureFunctionApp@1
    inputs:
      azureSubscription: '<AzureSubscription>'
      appType: 'functionApp'
      appName: '<FunctionAppName>'
      package: '$(Build.ArtifactStagingDirectory)'
```

1. **Pasos clave:**
   - Compilar la función en modo `Release`.
   - Publicar la salida como un artefacto del pipeline.
   - Desplegar el paquete en la Function App usando la tarea `AzureFunctionApp`.

2. **Variables necesarias:**
   - **azureSubscription:** Nombre de la conexión de servicio en Azure DevOps.
   - **appName:** Nombre de la Function App creada en Azure.

---

#### **4. Configuración avanzada**

1. **Uso de Azure Key Vault:**
   - Almacenar secretos y configuraciones sensibles en Azure Key Vault.
   - Conectar la Function App con Key Vault desde el portal o usando la CLI.

2. **Integración con triggers:**
   - Configurar triggers adicionales:
     - **Timer:** Para ejecutar funciones en intervalos regulares.
     - **Blob Storage:** Para procesar archivos cargados en un contenedor de Azure Blob Storage.
   - Ejemplo de trigger Timer:
     ```csharp
     [FunctionName("MyTimerFunction")]
     public static void Run(
         [TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
     {
         log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
     }
     ```

3. **Monitoreo con Application Insights:**
   - Habilitar Application Insights para obtener métricas y logs detallados.
   - Configurar la integración desde el portal o añadir la dependencia en el archivo `host.json`.

---

#### **5. Beneficios del uso de Azure Functions**

1. **Escalabilidad automática:** Las funciones se escalan en función de la carga, sin necesidad de gestionar infraestructura.
2. **Modelo de pago por uso:** Solo se factura por el tiempo de ejecución.
3. **Integración nativa con Azure:** Compatible con múltiples servicios como Storage, Event Grid y Service Bus.
4. **Despliegues rápidos:** Ideal para pipelines de CI/CD, permitiendo iteraciones frecuentes.

---

Azure Functions es una solución versátil que se adapta a numerosos casos de uso, desde la ejecución de tareas programadas hasta la integración con sistemas distribuidos, optimizando recursos y simplificando la gestión del ciclo de vida de las aplicaciones.