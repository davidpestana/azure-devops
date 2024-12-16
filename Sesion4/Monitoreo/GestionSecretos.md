### Gestión de secretos con Key Vault

Azure **Key Vault** es un servicio de Azure que permite **almacenar y gestionar de forma segura secretos**, claves y certificados. Integrar Key Vault en los pipelines de **Azure DevOps** ayuda a proteger información confidencial como contraseñas, claves API o cadenas de conexión, evitando que estos valores se expongan en el código.

---

#### 1. **¿Por qué usar Key Vault en Azure DevOps?**

- Centraliza la gestión y protección de secretos.
- Reduce riesgos de seguridad evitando almacenar credenciales en archivos o variables de entorno.
- Facilita la rotación de claves sin necesidad de modificar los pipelines.
- Permite controlar y auditar el acceso a los secretos mediante políticas de seguridad y registros.

---

#### 2. **Configuración de Azure Key Vault**

Antes de integrarlo en Azure DevOps, debes configurar un **Key Vault** en tu cuenta de Azure.

1. **Crear Key Vault**:  
   Ejecuta el siguiente comando con Azure CLI:  
   ```bash
   az keyvault create --name MyKeyVault --resource-group MyResourceGroup --location eastus
   ```

2. **Agregar un secreto al Key Vault**:  
   ```bash
   az keyvault secret set --vault-name MyKeyVault --name MySecretName --value MySecretValue
   ```

   Ejemplo:  
   ```bash
   az keyvault secret set --vault-name MyKeyVault --name SQLConnectionString --value "Server=myserver;Database=mydb;User Id=admin;Password=pass123;"
   ```

---

#### 3. **Integración de Key Vault en Azure Pipelines**

Para acceder a los secretos desde un pipeline, se utiliza la tarea **Azure Key Vault**.

1. **Configuración del pipeline con Key Vault**:  

   ```yaml
   variables:
     azureSubscription: 'MyAzureSubscription'
     keyVaultName: 'MyKeyVault'

   steps:
     - task: AzureKeyVault@2
       inputs:
         azureSubscription: $(azureSubscription)
         KeyVaultName: $(keyVaultName)
         SecretsFilter: '*'
       displayName: "Recuperar secretos de Key Vault"

     - script: |
         echo "Usando el secreto de Key Vault"
         echo "SQL Connection String: $(SQLConnectionString)"
       displayName: "Usar secreto en pipeline"
   ```

   **Explicación**:  
   - `AzureKeyVault@2`: Recupera los secretos del Key Vault.  
   - `KeyVaultName`: Especifica el nombre del Key Vault.  
   - `SecretsFilter`: Filtra qué secretos recuperar (`*` para todos o nombres específicos).  
   - Los secretos recuperados se convierten en **variables de entorno** disponibles para las siguientes tareas.

2. **Usar secretos en scripts o tareas**:  
   Una vez recuperados, los secretos se utilizan en el pipeline como variables.  
   Ejemplo en un script:  
   ```yaml
   steps:
     - script: |
         echo "Valor secreto: $(MySecretName)"
       displayName: "Acceder al secreto"
   ```

---

#### 4. **Seguridad y buenas prácticas**

1. **Asignar permisos mínimos**:  
   Utiliza **Managed Identity** o **Service Principals** para acceder al Key Vault.  
   Configura el acceso con **RBAC** (Role-Based Access Control):  
   ```bash
   az keyvault set-policy --name MyKeyVault --spn <ServicePrincipalID> --secret-permissions get list
   ```

2. **No exponer secretos en logs**:  
   Asegúrate de que las tareas no impriman secretos en los logs del pipeline:  
   ```yaml
   steps:
     - script: |
         echo "Este secreto no se mostrará"
       env:
         SECRET: $(MySecret)
   ```

3. **Rotación de secretos**:  
   Actualiza los valores en Key Vault sin modificar los pipelines. Los cambios se reflejan automáticamente.

4. **Auditoría y monitoreo**:  
   - Habilita **Azure Monitor** para rastrear el acceso al Key Vault.  
   - Configura alertas si se detectan accesos no autorizados.

---

#### 5. **Ejemplo práctico: Conexión a una base de datos**

Supongamos que necesitas una cadena de conexión SQL almacenada en Key Vault.

- **Pipeline YAML**:  
   ```yaml
   variables:
     azureSubscription: 'MyAzureSubscription'
     keyVaultName: 'MyKeyVault'

   steps:
     - task: AzureKeyVault@2
       inputs:
         azureSubscription: $(azureSubscription)
         KeyVaultName: $(keyVaultName)
         SecretsFilter: 'SQLConnectionString'

     - task: SqlAzureDacpacDeployment@1
       inputs:
         azureSubscription: $(azureSubscription)
         ServerName: 'myserver.database.windows.net'
         DatabaseName: 'mydb'
         SqlUsername: 'admin'
         SqlPassword: $(SQLConnectionString)
         DacpacFile: '$(Build.ArtifactStagingDirectory)/mydb.dacpac'
   ```

   **Explicación**:  
   - La cadena de conexión SQL se recupera desde Key Vault.  
   - Se utiliza como variable en la tarea `SqlAzureDacpacDeployment` para desplegar la base de datos.

---

#### 6. **Ventajas de usar Key Vault en CI/CD**

- **Seguridad**: Evita exponer secretos en código o archivos de configuración.  
- **Centralización**: Gestiona todos los secretos en un único lugar.  
- **Rotación de claves**: Actualiza los secretos sin interrumpir los pipelines.  
- **Escalabilidad**: Facilita la integración con múltiples entornos y aplicaciones.  

---

La integración de Azure Key Vault en los pipelines de Azure DevOps garantiza una gestión segura y eficiente de secretos, fortaleciendo la seguridad del proceso de CI/CD.