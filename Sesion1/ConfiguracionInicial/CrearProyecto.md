# Creación de un Proyecto en Azure DevOps

Crear un proyecto en Azure DevOps es el primer paso para comenzar a utilizar sus herramientas de gestión de código, CI/CD y colaboración. Este proceso incluye configurar un repositorio de código, personalizar las herramientas del proyecto y preparar el entorno para los miembros del equipo.

---

## Pasos para Crear un Proyecto en Azure DevOps

### **1. Iniciar Sesión en Azure DevOps**
   - Accede a [Azure DevOps](https://dev.azure.com) con una cuenta de Microsoft o Azure Active Directory.
   - Si es la primera vez, crea una **organización**:
     - Selecciona un nombre único para la organización.
     - Configura la región geográfica (preferentemente cercana a tu equipo o infraestructura).

---

### **2. Crear un Nuevo Proyecto**
1. En el portal de Azure DevOps, selecciona la **organización** en la que deseas crear el proyecto.
2. Haz clic en **"Nuevo proyecto"**.
3. Completa los siguientes campos:
   - **Nombre del proyecto**: Asigna un nombre único y descriptivo.
   - **Descripción** (opcional): Proporciona información adicional sobre el propósito del proyecto.
4. Selecciona la configuración del proyecto:
   - **Visibilidad**:
     - **Privado**: Solo los miembros del equipo pueden acceder.
     - **Público**: Acceso abierto para cualquier usuario (ideal para proyectos de código abierto).
   - **Control de versiones predeterminado**:
     - **Git**: Repositorios distribuidos.
     - **TFVC (Team Foundation Version Control)**: Repositorios centralizados (menos común).
5. Haz clic en **"Crear"**.

---

### **3. Configurar Herramientas del Proyecto**
Una vez creado el proyecto, Azure DevOps habilita por defecto una serie de herramientas que puedes personalizar según las necesidades del equipo:

1. **Tableros de Trabajo (Azure Boards)**:
   - Configura el tablero Kanban, backlog o sprints si usas metodologías ágiles.
   - Añade tareas, bugs, historias de usuario y épicas.

2. **Repositorio (Azure Repos)**:
   - Clona el repositorio en tu máquina local utilizando Git:
     ```bash
     git clone <URL del repositorio>
     ```
   - Crea ramas para diferentes características o tareas.

3. **Pipelines (Azure Pipelines)**:
   - Crea pipelines para CI/CD.
   - Usa un archivo YAML para definir el pipeline como código.
     Ejemplo básico para construir una aplicación .NET:
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

     - script: dotnet build
       displayName: 'Build Application'
     ```

4. **Pruebas (Azure Test Plans)**:
   - Define casos de prueba manuales o automatizados.
   - Vincula los casos de prueba con elementos de trabajo del tablero.

5. **Artefactos (Azure Artifacts)**:
   - Crea un feed para gestionar paquetes como NuGet, npm o Maven.

---

### **4. Configurar Permisos y Roles**
   - Añade miembros al equipo:
     1. Ve a **"Configuración del proyecto" → Equipos**.
     2. Selecciona **"Añadir miembros"** y especifica sus direcciones de correo.
   - Asigna roles según los permisos necesarios:
     - **Lector**: Acceso de solo lectura.
     - **Colaborador**: Permisos para contribuir al repositorio y modificar elementos.
     - **Administrador**: Control total del proyecto.

---

### **5. Personalizar el Proyecto**
1. **Configuración del Proceso**:
   - Ve a **"Configuración del proyecto" → Configuración de procesos**.
   - Selecciona el proceso que mejor se adapte a tu flujo de trabajo:
     - **Agile**: Ideal para equipos que usan metodologías ágiles.
     - **Scrum**: Diseñado específicamente para Scrum.
     - **CMMI**: Para proyectos con flujos más rígidos y controlados.

2. **Configuración del Entorno**:
   - Configura integraciones con herramientas externas como Slack, Teams o GitHub.

3. **Gestión de Notificaciones**:
   - Personaliza las notificaciones para eventos como commits, builds fallidas o tareas cerradas.

---

### **6. Probar el Proyecto**
1. Realiza un commit inicial al repositorio.
2. Crea un pipeline simple para construir el código y validar que todo funciona correctamente.
3. Configura un flujo básico en Azure Boards y asigna tareas iniciales al equipo.

---

## Ejemplo de Caso Práctico

1. **Proyecto**: Desarrollo de una aplicación web en .NET.
2. **Configuraciones**:
   - Repositorio Git.
   - Pipeline para construir y probar la aplicación automáticamente en cada commit.
   - Tableros Kanban para organizar las tareas del sprint.
3. **Roles**:
   - Administrador: Configura el proyecto y los pipelines.
   - Desarrolladores: Contribuyen al código y actualizan el tablero.

---

## Beneficios de Usar Azure DevOps para Proyectos

1. **Gestión Centralizada**:
   - Herramientas integradas para gestionar código, tareas, pruebas y despliegues.
2. **Escalabilidad**:
   - Adecuado para proyectos pequeños o grandes con múltiples equipos.
3. **Automatización Completa**:
   - Pipelines personalizables para CI/CD.
4. **Colaboración Eficiente**:
   - Mejora la colaboración entre desarrolladores, testers y gerentes de proyecto.

Con este flujo, Azure DevOps permite iniciar rápidamente un proyecto y escalarlo según las necesidades del equipo.