### Despliegue en IIS con Web Deploy

El despliegue de aplicaciones .NET en servidores IIS mediante Web Deploy es una técnica común para entornos on-premises o híbridos. Este proceso automatiza la publicación y configuración de aplicaciones en servidores IIS desde pipelines de CI/CD.

---

#### **1. Configuración del servidor IIS**

1. **Instalación de IIS y Web Deploy:**
   - Instalar el rol de servidor web (IIS) en un servidor Windows.
   - Añadir el componente **Web Deploy 4.0** desde el instalador de la plataforma web de Microsoft.

2. **Configuración de la publicación remota:**
   - Habilitar la opción de publicación web (Web Management Service).
   - Configurar las credenciales de usuario para la publicación (usualmente un usuario local con permisos en el sitio IIS).
   - Permitir las conexiones en el puerto 8172 para el servicio de administración web.

3. **Creación del sitio web:**
   - Configurar un nuevo sitio en IIS con el nombre y la ruta del contenido.
   - Asegurarse de que la aplicación se ejecuta bajo el grupo de aplicaciones adecuado con .NET habilitado.

---

#### **2. Configuración del pipeline en Azure DevOps**

1. **Preparar el proyecto .NET:**
   - En el archivo del proyecto (`.csproj`), incluir el perfil de publicación si es necesario.
   - Verificar que las dependencias del proyecto están actualizadas.

2. **Pipeline YAML básico para Web Deploy:**
   ```yaml
   trigger:
     - main

   pool:
     vmImage: 'windows-latest'

   steps:
     - task: UseDotNet@2
       inputs:
         packageType: 'sdk'
         version: '7.x' # Cambiar según la versión de .NET
     
     - task: DotNetCoreCLI@2
       inputs:
         command: 'publish'
         arguments: '--configuration Release --output $(Build.ArtifactStagingDirectory)'
         projects: '**/*.csproj'

     - task: PublishBuildArtifacts@1
       inputs:
         PathtoPublish: '$(Build.ArtifactStagingDirectory)'
         ArtifactName: 'drop'

     - task: IISWebAppDeploy@1
       inputs:
         WebSiteName: 'NombreDelSitioIIS'
         Package: '$(Pipeline.Workspace)/drop/*.zip'
         TakeAppOfflineFlag: true
         RemoveAdditionalFilesFlag: true
         AdditionalArguments: '-verbose'
         XmlTransformation: true
         WebAppPool: 'DefaultAppPool' # O especificar el grupo de aplicaciones
   ```

3. **Detalles de configuración:**
   - `WebSiteName`: Especificar el nombre del sitio configurado en IIS.
   - `Package`: Ruta del paquete `.zip` generado por la tarea de publicación.
   - `WebAppPool`: Nombre del grupo de aplicaciones en IIS.

---

#### **3. Flujo de despliegue**

1. **Compilación y publicación:**
   - Compilar el proyecto con el comando `dotnet publish`.
   - Generar un paquete `.zip` con la salida de la compilación.

2. **Despliegue remoto:**
   - Utilizar la tarea `IISWebAppDeploy` para publicar el paquete en el servidor IIS mediante Web Deploy.
   - Asegurarse de que el pipeline autentica correctamente al servidor usando las credenciales configuradas.

3. **Pruebas posteriores al despliegue:**
   - Verificar que la aplicación está accesible desde el navegador.
   - Comprobar los logs en IIS para diagnosticar errores.

---

#### **4. Ventajas y casos de uso**

- **Automatización completa:** Integrar Web Deploy en un pipeline elimina errores manuales y permite despliegues consistentes.
- **Flexibilidad:** Adecuado para aplicaciones corporativas y sistemas internos.
- **Compatibilidad:** Soporte nativo para aplicaciones ASP.NET Framework y ASP.NET Core.

Este enfoque permite mantener una solución confiable y repetible para la entrega continua de aplicaciones en entornos on-premises basados en IIS.