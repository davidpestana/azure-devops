# Laboratorio Práctico: Configuración Inicial de un Proyecto .NET en Azure DevOps

Este laboratorio te guiará paso a paso para configurar un proyecto .NET en **Azure DevOps**, desde la creación del proyecto hasta la configuración de un pipeline básico de CI/CD. Nos enfocaremos en **por qué** realizamos cada acción, para que entiendas no solo cómo hacerlo, sino también su propósito.

---

## **Objetivo**
Configurar un entorno funcional en Azure DevOps para el desarrollo y despliegue de una aplicación .NET. Esto incluye:
- Creación de un proyecto en Azure DevOps.
- Configuración de repositorios para gestión de código.
- Creación de un pipeline básico para construcción y pruebas.
- Publicación del proyecto desde Visual Studio.

---

## **Requisitos Previos**
1. Una cuenta de **Azure DevOps**.
2. **Visual Studio** instalado con el SDK de .NET (versión 6.0 o superior).
3. Git configurado en tu máquina local.
4. Acceso a un navegador web para interactuar con el portal de Azure DevOps.

---

## **Paso 1: Crear un Proyecto en Azure DevOps**

### **¿Por qué?**
Un proyecto en Azure DevOps centraliza todas las herramientas necesarias para la gestión, el desarrollo y el despliegue de tu aplicación. Esto permite a los equipos colaborar eficientemente.

### **Instrucciones**
1. Inicia sesión en [Azure DevOps](https://dev.azure.com).
2. Haz clic en **"Nuevo Proyecto"**.
3. Completa los campos:
   - **Nombre del proyecto**: Por ejemplo, `DemoDotNetApp`.
   - **Visibilidad**: Selecciona **Privado** para que solo tu equipo tenga acceso.
   - **Tipo de control de versiones**: Elige **Git**.
4. Haz clic en **Crear**.

---

## **Paso 2: Configurar un Repositorio Git**

### **¿Por qué?**
Un repositorio Git permite gestionar el código fuente de forma eficiente, con historial de cambios y soporte para colaboración en equipo.

### **Instrucciones**
1. Ve a la sección **Repos** en el menú lateral.
2. Copia la URL del repositorio inicial.
3. En tu terminal local, clona el repositorio:
   ```bash
   git clone <URL_DEL_REPOSITORIO>
   ```
4. Crea un archivo `.gitignore` en el repositorio local para excluir archivos innecesarios (por ejemplo, `bin/`, `obj/`, `*.user`).

---

## **Paso 3: Crear una Aplicación .NET en Visual Studio**

### **¿Por qué?**
Visual Studio proporciona herramientas para desarrollar y construir aplicaciones .NET de forma rápida y sencilla.

### **Instrucciones**
1. Abre **Visual Studio**.
2. Selecciona **Crear un nuevo proyecto**.
3. Elige **Aplicación Web ASP.NET Core**.
4. Configura:
   - **Nombre del Proyecto**: `DemoDotNetApp`.
   - **Ubicación**: Dentro de la carpeta del repositorio clonado.
   - **Framework**: Selecciona la versión más reciente de .NET.
5. Guarda el proyecto y ejecuta localmente para asegurarte de que funciona.

### **Añadir al Repositorio**
1. Desde la terminal, añade los archivos del proyecto al repositorio:
   ```bash
   git add .
   git commit -m "Initial commit: Add ASP.NET Core project"
   git push origin main
   ```

---

## **Paso 4: Configurar un Pipeline de CI**

### **¿Por qué?**
Un pipeline de integración continua (CI) garantiza que cada cambio en el código pase por un proceso automatizado de construcción y pruebas, asegurando calidad y detectando errores rápidamente.

### **Instrucciones**
1. Ve a **Pipelines** en Azure DevOps.
2. Haz clic en **Crear Pipeline**.
3. Selecciona **Usar el editor YAML**.
4. Configura el pipeline básico:
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
       version: '6.0.x'

   - script: dotnet restore
     displayName: 'Restore Dependencies'

   - script: dotnet build --configuration Release
     displayName: 'Build Application'

   - script: dotnet test --configuration Release
     displayName: 'Run Tests'
   ```
5. Guarda y ejecuta el pipeline.

---

## **Paso 5: Verificar el Pipeline**

### **¿Por qué?**
Es importante validar que el pipeline funcione correctamente antes de avanzar con configuraciones más avanzadas.

### **Instrucciones**
1. Haz un pequeño cambio en el código, como editar el contenido del archivo `Index.cshtml`.
2. Confirma el cambio:
   ```bash
   git add .
   git commit -m "Update homepage content"
   git push origin main
   ```
3. Observa cómo el pipeline se activa automáticamente.
4. Verifica los logs para asegurarte de que todos los pasos (restaurar, construir y probar) se ejecutaron correctamente.

---

## **Paso 6: Configurar un Pipeline de CD (Entrega Continua)**

### **¿Por qué?**
Un pipeline de entrega continua automatiza el despliegue de la aplicación a entornos como Azure App Service o servidores on-premises.

### **Instrucciones**
1. Modifica el archivo `azure-pipelines.yml` para incluir un paso de despliegue:
   ```yaml
   - task: AzureWebApp@1
     inputs:
       azureSubscription: 'MyAzureServiceConnection'
       appName: 'DemoDotNetAppService'
       package: '$(Pipeline.Workspace)/drop'
   ```
2. Configura una **Service Connection** en Azure DevOps para conectar con tu Azure Subscription.
3. Guarda y ejecuta el pipeline.

---

## **Iteración 1: Probar con Pequeños Cambios**

1. Realiza cambios simples en el código, como modificar la página de inicio.
2. Observa cómo los pipelines aseguran que el cambio se construye, prueba y despliega automáticamente.

---

## **Iteración 2: Introducir Errores Intencionados**

1. Introduce un error en el código para verificar que las pruebas lo detectan.
2. Analiza los resultados del pipeline y corrige el error.
3. Observa cómo el pipeline funciona correctamente tras la corrección.

---

## **Iteración 3: Escalar con Equipos**

1. Invita a otros miembros del equipo al proyecto.
2. Asigna tareas en **Azure Boards**.
3. Colabora en nuevas ramas y crea Pull Requests para revisar los cambios antes de fusionarlos.

---

## **Conclusión**

Este laboratorio cubre los aspectos fundamentales para configurar y validar un proyecto .NET en Azure DevOps. Iterar sobre estas configuraciones permite desarrollar y desplegar aplicaciones de manera eficiente, confiable y colaborativa.