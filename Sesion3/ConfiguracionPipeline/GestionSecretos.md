### Gestión de secretos con Azure Key Vault

Azure Key Vault es un servicio para almacenar y gestionar secretos, claves de cifrado y certificados de forma segura. Su integración con pipelines de CI/CD garantiza que las aplicaciones y procesos puedan acceder a configuraciones sensibles sin exponerlas en el código.

---

#### **1. Configuración inicial en Azure Key Vault**

1. **Crear un Key Vault:**
   - En Azure Portal, selecciona **Create a resource** y busca **Key Vault**.
   - Configura los detalles:
     - **Nombre:** Nombre único del Key Vault.
     - **Región:** Escoge la región más cercana.
     - **Pricing tier:** Estándar o Premium (para claves HSM).

2. **Añadir secretos al Key Vault:**
   - En el Key Vault, navega a **Secrets** y selecciona **+ Generate/Import**.
   - Introduce un nombre para el secreto (por ejemplo, `ConnectionString`) y su valor.

3. **Configurar acceso a Key Vault:**
   - En **Access policies**, añade una política para permitir acceso al servicio o identidad que ejecutará el pipeline.
   - Permitir permisos como **Get** y **List** para secretos.

---

#### **2. Configuración en Azure DevOps**

1. **Crear una conexión de servicio con Azure Key Vault:**
   - En Azure DevOps, ve a **Project Settings > Service Connections**.
   - Crea una conexión de tipo **Azure Resource Manager** con acceso al Key Vault.

2. **Incorporar variables del Key Vault al pipeline:**
   - Configura una tarea de Key Vault en el pipeline para importar secretos como variables:
     ```yaml
     - task: AzureKeyVault@2
       inputs:
         azureSubscription: '<NombreDeConexión>'
         keyVaultName: '<NombreDelKeyVault>'
         secretsFilter: '*'
         runAsPreJob: true
     ```

3. **Uso de secretos en el pipeline:**
   - Una vez importados, los secretos pueden ser referenciados como variables:
     ```yaml
     - script: |
         echo "Using secret value"
         echo $(ConnectionString)
       displayName: 'Print ConnectionString'
     ```

---

#### **3. Integración con aplicaciones .NET**

1. **Configurar acceso desde la aplicación:**
   - Añadir el paquete `Azure.Extensions.AspNetCore.Configuration.Secrets`:
     ```bash
     dotnet add package Azure.Extensions.AspNetCore.Configuration.Secrets
     ```

2. **Configurar Key Vault en el `Program.cs` o `Startup.cs`:**
   ```csharp
   var builder = WebApplication.CreateBuilder(args);

   // Add Azure Key Vault to configuration
   var keyVaultEndpoint = new Uri(Environment.GetEnvironmentVariable("KEYVAULT_URI")!);
   builder.Configuration.AddAzureKeyVault(keyVaultEndpoint, new DefaultAzureCredential());

   var app = builder.Build();
   ```

3. **Autenticación con Managed Identity:**
   - Habilitar la Identidad Administrada en el App Service.
   - Configurar permisos en Key Vault para la identidad administrada.

---

#### **4. Prácticas recomendadas**

1. **Uso de Managed Identity:**
   - Prefiere usar identidades administradas en lugar de credenciales estáticas para mayor seguridad.

2. **Segmentación de secretos:**
   - Organiza los secretos en múltiples Key Vaults según el entorno (Desarrollo, Testing, Producción).

3. **Rotación automática de secretos:**
   - Habilita la rotación automática para claves y certificados gestionados.

4. **Restricción de acceso:**
   - Configura políticas de acceso basadas en roles (RBAC) para limitar quién puede ver o modificar los secretos.

---

#### **5. Ejemplo de flujo completo en YAML**

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: 'PipelineVariables' # Variables generales

steps:
  # Obtener secretos del Key Vault
  - task: AzureKeyVault@2
    inputs:
      azureSubscription: '<AzureSubscription>'
      keyVaultName: '<KeyVaultName>'
      secretsFilter: '*'
      runAsPreJob: true

  # Usar los secretos en un script
  - script: |
      echo "Connection string: $(ConnectionString)"
      echo "API Key: $(ApiKey)"
    displayName: 'Print secrets from Key Vault'

  # Implementación en App Service usando secretos
  - task: AzureWebApp@1
    inputs:
      azureSubscription: '<AzureSubscription>'
      appName: '<AppName>'
      package: '$(Pipeline.Workspace)/drop/*.zip'
      appSettings: '-ConnectionString $(ConnectionString) -ApiKey $(ApiKey)'
```

---

#### **6. Beneficios de usar Azure Key Vault**

1. **Seguridad:** Centraliza secretos y evita exponer configuraciones sensibles en el código o en variables de entorno.
2. **Escalabilidad:** Simplifica la gestión de secretos a medida que aumenta la complejidad de la infraestructura.
3. **Integración nativa:** Compatible con servicios de Azure y herramientas como Azure DevOps y Terraform.

---

Azure Key Vault es una herramienta fundamental para implementar una gestión segura de secretos, mejorando la seguridad y eficiencia en pipelines de CI/CD y aplicaciones modernas.