### Reducción de tiempos de ejecución

La optimización de los tiempos de ejecución en los pipelines de Azure DevOps es fundamental para acelerar las entregas de software, reducir costos y mejorar la eficiencia general. A continuación, presentamos prácticas, estrategias y configuraciones clave para lograr esta optimización.

---

#### 1. **Uso de trabajos en paralelo (Parallel Jobs)**  

Azure DevOps permite ejecutar trabajos simultáneamente para reducir el tiempo total de ejecución de un pipeline.  

- **Configuración en YAML**:  
   ```yaml
   trigger:
     - main

   jobs:
     - job: TestJob1
       pool:
         vmImage: 'ubuntu-latest'
       steps:
         - script: echo "Ejecución de Job 1"
           displayName: "Job 1 - Pruebas Unitarias"

     - job: TestJob2
       dependsOn: []
       pool:
         vmImage: 'ubuntu-latest'
       steps:
         - script: echo "Ejecución de Job 2"
           displayName: "Job 2 - Pruebas Funcionales"
   ```

   **Explicación**:  
   - Los dos trabajos (`TestJob1` y `TestJob2`) se ejecutan en paralelo porque `dependsOn` está vacío en el segundo trabajo.  
   - La ejecución en paralelo reduce el tiempo total del pipeline, ideal para pruebas unitarias o funcionales independientes.

---

#### 2. **Uso de caché (Pipeline Caching)**  

El almacenamiento en caché permite reutilizar dependencias descargadas o resultados intermedios entre ejecuciones, lo que acelera significativamente las pipelines.

- **Configuración en YAML**:
   ```yaml
   variables:
     CACHE_RESTORED: 'false'

   steps:
     - task: Cache@2
       inputs:
         key: 'npm | "$(Agent.OS)" | package-lock.json'
         restoreKeys: |
           npm | "$(Agent.OS)"
         path: $(Build.SourcesDirectory)/node_modules
       displayName: "Cache Node Modules"

     - script: |
         npm install
       condition: and(succeeded(), eq(variables.CACHE_RESTORED, 'false'))
       displayName: "Instalar dependencias"
   ```

   **Explicación**:
   - `Cache@2` almacena en caché las dependencias (`node_modules`) para que no se instalen de cero en cada ejecución.  
   - La clave de caché se define en función del sistema operativo y del archivo `package-lock.json`.  
   - El tiempo de instalación se reduce drásticamente cuando la caché está disponible.

---

#### 3. **Optimización de agentes de ejecución**  

Utilizar agentes con recursos adecuados es clave para optimizar los pipelines:  

- **Imágenes ligeras**: Usar agentes más específicos y con menos herramientas preinstaladas. Por ejemplo:  
   - `ubuntu-latest` o `windows-latest` en lugar de agentes personalizados.  
- **Aumentar el número de agentes**: Escalar el número de agentes permite ejecutar pipelines concurrentes.  
   - Configurable en el **Azure DevOps Portal** → **Project Settings** → **Parallel Jobs**.

---

#### 4. **Reducción de pasos innecesarios**  

Analiza y elimina pasos redundantes en tus pipelines:  

- Combina comandos siempre que sea posible:  
   ```yaml
   steps:
     - script: |
         echo "Compilación"
         npm run build
         npm test
       displayName: "Build y Test en un paso"
   ```

- Usa `condition` para evitar pasos que no sean necesarios en ciertas condiciones:  
   ```yaml
   steps:
     - script: echo "Este paso solo se ejecuta en main"
       condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
   ```

---

#### 5. **Reducir la descarga de artefactos**  

Descargar solo los artefactos necesarios para el trabajo actual:  
```yaml
- task: DownloadBuildArtifacts@0
  inputs:
    buildType: 'current'
    downloadType: 'single'
    artifactName: 'release_artifacts'
    downloadPath: '$(System.ArtifactsDirectory)'
  displayName: "Descargar solo artefacto necesario"
```

---

#### 6. **Ejecutar solo los pipelines necesarios**  

Usa `path` y `trigger` para ejecutar pipelines solo cuando los cambios afecten a archivos relevantes.  

```yaml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - src/
      - azure-pipelines.yml
```

---

#### 7. **Pruebas incrementales**  

Ejecuta solo las pruebas relacionadas con los cambios recientes usando herramientas de análisis de cobertura y detección de cambios.  

Ejemplo de uso con herramientas como Jest:  
```bash
jest --onlyChanged
```

---

#### 8. **Monitoreo y ajustes continuos**  

- Activa **"Pipeline Analytics"** para identificar cuellos de botella y tiempos innecesarios.  
- Utiliza métricas disponibles en Azure DevOps:  
   - **Tiempo total de ejecución**  
   - **Tiempo de espera de agentes**  
   - **Duración de cada paso del pipeline**

---

Con estas prácticas, la ejecución de tus pipelines será más eficiente, rápida y optimizada, facilitando una entrega continua más ágil.