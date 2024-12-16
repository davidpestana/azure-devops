### Uso de plantillas YAML reutilizables

Las plantillas YAML en Azure DevOps permiten simplificar y estandarizar la configuración de pipelines, evitando la repetición de código y facilitando el mantenimiento. Mediante las plantillas, se pueden definir bloques de pasos, trabajos o incluso pipelines completos que se reutilizan en distintos proyectos o etapas.

---

#### 1. **¿Qué son las plantillas YAML?**  

Las plantillas YAML son archivos que contienen una sección o un conjunto de configuraciones reutilizables de un pipeline. Estas plantillas pueden ser llamadas desde otros archivos YAML y permiten **parametrización** para adaptarlas a diferentes escenarios.

---

#### 2. **Estructura de una plantilla YAML**  

Las plantillas deben colocarse en archivos `.yml` o `.yaml` separados y contener al menos un bloque reutilizable (`steps`, `jobs`, `stages`).  

- **Ejemplo de plantilla básica**:  
   ```yaml
   # build-template.yml
   parameters:
     buildConfiguration: 'Release'

   steps:
     - task: UseDotNet@2
       inputs:
         version: '6.x'
       displayName: "Configurar .NET"

     - script: |
         dotnet build --configuration ${{ parameters.buildConfiguration }}
       displayName: "Compilación del proyecto"
   ```

---

#### 3. **Uso de la plantilla en el pipeline principal**  

Una vez definida la plantilla, se puede incluir en el pipeline principal usando la sintaxis `template`.  

- **Ejemplo de llamada a la plantilla**:  
   ```yaml
   trigger:
     - main

   jobs:
     - job: BuildJob
       displayName: "Job de Compilación"
       pool:
         vmImage: 'ubuntu-latest'
       steps:
         - template: build-template.yml
           parameters:
             buildConfiguration: 'Debug'
   ```

   **Explicación**:  
   - `template`: referencia al archivo de la plantilla.  
   - `parameters`: permite personalizar los valores definidos en la plantilla.

---

#### 4. **Plantillas para múltiples etapas (stages)**  

Las plantillas también permiten definir **stages** reutilizables para implementaciones complejas:  

- **Ejemplo de plantilla de stages**:  
   ```yaml
   # deploy-template.yml
   parameters:
     environment: 'dev'

   stages:
     - stage: Deploy
       displayName: "Despliegue en ${{ parameters.environment }}"
       jobs:
         - job: DeployJob
           pool:
             vmImage: 'windows-latest'
           steps:
             - script: echo "Desplegando en ${{ parameters.environment }}"
               displayName: "Ejecución del despliegue"
   ```

- **Uso en el pipeline principal**:  
   ```yaml
   stages:
     - template: deploy-template.yml
       parameters:
         environment: 'production'
   ```

---

#### 5. **Organización de plantillas**  

Para una mejor organización y reutilización:  
- Guarda las plantillas en una carpeta específica, por ejemplo: `templates/`.  
- Usa nombres descriptivos para los archivos: `build-template.yml`, `test-template.yml`, `deploy-template.yml`.  
- Ejemplo de estructura de carpetas:  
   ```
   .
   ├── azure-pipelines.yml
   ├── templates/
   │   ├── build-template.yml
   │   ├── test-template.yml
   │   └── deploy-template.yml
   ```

---

#### 6. **Plantillas parametrizables y flexibles**  

Se pueden agregar parámetros en las plantillas para hacerlas más genéricas y adaptables:  

- **Ejemplo con múltiples parámetros**:  
   ```yaml
   # test-template.yml
   parameters:
     testFramework: 'NUnit'
     testCommand: 'dotnet test'

   steps:
     - script: ${{ parameters.testCommand }} --framework ${{ parameters.testFramework }}
       displayName: "Ejecución de pruebas"
   ```

- **Uso de la plantilla**:  
   ```yaml
   steps:
     - template: templates/test-template.yml
       parameters:
         testFramework: 'xUnit'
         testCommand: 'dotnet test --filter TestCategory=SmokeTests'
   ```

---

#### 7. **Buenas prácticas**  

1. **Divide por función**: crea plantillas específicas para **compilación**, **pruebas**, **despliegue**, etc.  
2. **Mantén parámetros mínimos**: limita los parámetros a lo necesario para evitar complejidad.  
3. **Documenta**: agrega comentarios en las plantillas para explicar su propósito y parámetros.  
4. **Versiona las plantillas**: almacénalas en un repositorio Git y versiona las plantillas para evitar impactos en pipelines existentes.  

---

#### 8. **Ventajas de usar plantillas YAML**  

- **Reutilización**: evita duplicar código entre pipelines.  
- **Mantenimiento**: cambios en un solo archivo afectan a todos los pipelines que la utilizan.  
- **Consistencia**: asegura que todos los pipelines sigan las mismas prácticas y configuraciones.  
- **Escalabilidad**: permite gestionar pipelines más complejos de forma estructurada.

---

Con el uso de plantillas YAML reutilizables, es posible optimizar los pipelines de Azure DevOps, mejorar la organización del código y facilitar la implementación de prácticas de CI/CD de manera eficiente.