# Conexión entre Credenciales para Despliegues On-Premises y en la Nube

La gestión adecuada de las credenciales es esencial para automatizar despliegues en entornos híbridos, combinando servidores locales (on-premises) y servicios en la nube. **Azure DevOps** permite conectar y gestionar credenciales de manera segura mediante servicios integrados como **Service Connections**, **Azure Key Vault** y agentes auto-hospedados.

---

## Estrategias para Manejar Credenciales en Despliegues Híbridos

### **1. Usar Service Connections en Azure DevOps**
Un **Service Connection** permite a Azure DevOps conectarse de manera segura con recursos tanto en la nube como locales. Soporta configuraciones para Azure, AWS, SSH, y más.

#### **Configurar una Service Connection**
1. Ve a **"Configuración del Proyecto" → "Service Connections"**.
2. Haz clic en **"Nuevo servicio de conexión"**.
3. Elige el tipo de conexión:
   - **Azure Resource Manager** para Azure.
   - **AWS** para Amazon Web Services.
   - **SSH** para servidores on-premises.
4. Proporciona los datos necesarios:
   - **Para Azure**:
     - Inicia sesión con tu cuenta de Azure.
     - Selecciona la suscripción y el recurso deseado.
   - **Para SSH**:
     - Especifica la IP del servidor, el puerto y la clave SSH privada.
5. Asigna permisos a los pipelines que usarán esta conexión.

#### **Uso en un Pipeline**
Ejemplo para desplegar en Azure App Service:
```yaml
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'MyAzureServiceConnection'
    appName: 'MyAppService'
    package: '$(Pipeline.Workspace)/drop'
```

---

### **2. Agentes Auto-Hospedados para On-Premises**
Un **agente auto-hospedado** permite a Azure DevOps interactuar con recursos locales. Es esencial para ejecutar scripts y comandos en servidores on-premises.

#### **Configurar un Agente Auto-Hospedado**
1. **Crear un Pool de Agentes**:
   - Ve a **"Configuración de la Organización" → "Pools de Agentes"**.
   - Haz clic en **"Nuevo pool de agentes"** y nómbralo (por ejemplo, `OnPremisesPool`).

2. **Registrar el Agente**:
   - Descarga el agente desde Azure DevOps.
   - Instálalo en el servidor local:
     ```bash
     ./config.sh --unattended \
       --url https://dev.azure.com/your-organization \
       --auth pat \
       --token <personal-access-token> \
       --pool OnPremisesPool
     ```

3. **Asignar el Agente al Pipeline**:
   - Configura el pipeline para usar el pool del agente:
     ```yaml
     pool:
       name: OnPremisesPool
     ```

#### **Usar Credenciales Locales**
- Guarda las credenciales locales en variables de pipeline:
  ```yaml
  variables:
    DB_USERNAME: 'admin'
    DB_PASSWORD: 'securepassword'
  ```

---

### **3. Gestión de Secretos con Azure Key Vault**
**Azure Key Vault** es una herramienta para almacenar y recuperar secretos como claves, contraseñas y cadenas de conexión. Esto permite manejar credenciales de forma centralizada y segura.

#### **Configurar Azure Key Vault**
1. **Crear un Key Vault**:
   - En el portal de Azure, crea un nuevo **Azure Key Vault**.
   - Añade secretos necesarios:
     - `DB_PASSWORD`
     - `STORAGE_ACCOUNT_KEY`

2. **Conectar Azure Key Vault a Azure DevOps**:
   - Ve a **"Service Connections"** y configura una conexión al Key Vault.

3. **Referenciar Secretos en un Pipeline**:
   - Usa la tarea `AzureKeyVault@1` para recuperar secretos:
     ```yaml
     - task: AzureKeyVault@1
       inputs:
         azureSubscription: 'MyAzureServiceConnection'
         KeyVaultName: 'MyKeyVault'
         SecretsFilter: '*'
     ```

4. **Acceso a Secretos**:
   - Usa los secretos como variables:
     ```yaml
     - script: echo $(DB_PASSWORD)
     ```

---

### **4. Uso de Variables de Pipeline**
Para manejar credenciales que no se almacenan en Key Vault, Azure DevOps permite definir **variables de pipeline** que pueden ser encriptadas.

#### **Crear Variables de Pipeline**
1. Ve a **"Pipelines" → "Editar Pipeline"**.
2. Añade variables:
   - Nombre: `DB_PASSWORD`.
   - Valor: `mypassword`.
   - Habilita la opción **"Mantener secreto"**.

#### **Uso de Variables en Tareas**
```yaml
variables:
  DB_PASSWORD: $(DB_PASSWORD)

steps:
- script: echo "Using database password: $(DB_PASSWORD)"
```

---

### **5. Conexión entre Servicios On-Premises y en la Nube**
Para establecer comunicación segura entre entornos locales y en la nube:

1. **Configurar un Proxy o VPN**:
   - Usa un túnel seguro para exponer recursos locales.
   - Opciones como **Azure VPN Gateway** pueden facilitar esta conexión.

2. **Habilitar Azure Arc para Servidores**:
   - Registra servidores on-premises en Azure para gestionarlos como recursos de la nube.
   - Esto permite ejecutar tareas remotas mediante Azure DevOps.

---

## Buenas Prácticas en la Gestión de Credenciales

1. **Nunca Guardar Credenciales en Código**:
   - Usa Key Vault o variables encriptadas para almacenar datos sensibles.

2. **Rotación Regular de Credenciales**:
   - Cambia contraseñas y claves periódicamente para reducir riesgos.

3. **Asignar Permisos Mínimos Necesarios**:
   - Configura roles en Azure y on-premises para limitar accesos.

4. **Monitorear el Acceso**:
   - Usa herramientas como **Azure Monitor** o **SIEM locales** para rastrear el uso de credenciales.

---

Con estas configuraciones, Azure DevOps puede gestionar credenciales para despliegues híbridos de manera segura y eficiente, automatizando flujos de trabajo en entornos locales y en la nube.