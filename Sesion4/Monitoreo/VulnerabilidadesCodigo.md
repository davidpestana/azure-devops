### Análisis de vulnerabilidades en código y dependencias

El análisis de vulnerabilidades en código y dependencias es una práctica esencial para garantizar la seguridad del software en el ciclo de CI/CD. En Azure DevOps, se pueden integrar herramientas de análisis estático y escaneo de dependencias para detectar problemas de seguridad antes de que lleguen a producción.

---

#### 1. **¿Por qué realizar análisis de vulnerabilidades?**

- Detecta dependencias desactualizadas o inseguras.
- Identifica errores de seguridad en el código fuente.
- Cumple con estándares de seguridad y compliance.
- Evita la introducción de vulnerabilidades críticas en el software.

---

#### 2. **Integración de herramientas de análisis de código en Azure DevOps**  

Se pueden integrar herramientas tanto **open source** como comerciales en el pipeline de Azure DevOps.

---

##### a) **Análisis estático con SonarCloud o SonarQube**  

SonarCloud (SaaS) y SonarQube (on-premise) son herramientas que permiten escanear el código fuente en busca de vulnerabilidades, errores y código duplicado.

- **Integración de SonarCloud en un pipeline**:  

   1. Instala la extensión **SonarCloud** en Azure DevOps.  
   2. Agrega las tareas de análisis al pipeline:  

   ```yaml
   variables:
     sonarProjectKey: 'MyProjectKey'
     sonarOrganization: 'MyOrganization'
     sonarToken: '$(SONAR_TOKEN)'

   steps:
     - task: SonarCloudPrepare@1
       inputs:
         SonarCloud: 'SonarCloud Service Connection'
         organization: $(sonarOrganization)
         projectKey: $(sonarProjectKey)
         projectName: 'MyProject'

     - task: DotNetCoreCLI@2
       inputs:
         command: 'build'
         projects: '**/*.csproj'

     - task: SonarCloudAnalyze@1

     - task: SonarCloudPublish@1
       inputs:
         pollingTimeoutSec: '300'
   ```

   **Explicación**:  
   - `SonarCloudPrepare@1` configura el escaneo.  
   - `DotNetCoreCLI@2` compila el código fuente.  
   - `SonarCloudAnalyze@1` ejecuta el análisis estático.  
   - `SonarCloudPublish@1` publica los resultados en SonarCloud.  

---

##### b) **Escaneo de dependencias con OWASP Dependency-Check**  

OWASP Dependency-Check es una herramienta que detecta vulnerabilidades conocidas en las dependencias.

- **Configuración en YAML**:  
   ```yaml
   steps:
     - task: CmdLine@2
       displayName: "Escaneo de dependencias con OWASP Dependency-Check"
       inputs:
         script: |
           curl -L https://github.com/jeremylong/DependencyCheck/releases/download/v7.0.0/dependency-check-7.0.0-release.zip -o dc.zip
           unzip dc.zip -d dependency-check
           ./dependency-check/bin/dependency-check.sh --project "MyProject" --scan "$(Build.SourcesDirectory)" --format "HTML" --out "dependency-report"
     - publish: 'dependency-report'
       artifact: 'DependencyCheckReport'
   ```

   **Explicación**:  
   - Descarga y ejecuta OWASP Dependency-Check.  
   - Genera un informe HTML con los resultados.  
   - Publica el reporte como un artefacto del pipeline.  

---

##### c) **Análisis de vulnerabilidades con WhiteSource Bolt**  

WhiteSource Bolt es una herramienta gratuita para Azure DevOps que realiza escaneo de vulnerabilidades en dependencias.

1. Instala la extensión **WhiteSource Bolt** desde el marketplace de Azure DevOps.  
2. Agrega la tarea en el pipeline:  

   ```yaml
   steps:
     - task: WhiteSource Bolt@19
       inputs:
         cwd: '$(Build.SourcesDirectory)'
         apiToken: '$(WHITE_SOURCE_TOKEN)'
   ```

---

#### 3. **Escaneo de contenedores (Docker) en CI/CD**  

Para aplicaciones containerizadas, es importante analizar las imágenes Docker en busca de vulnerabilidades. Herramientas como **Trivy** o **Aqua Security** pueden integrarse fácilmente.

- **Ejemplo usando Trivy**:  

   ```yaml
   steps:
     - script: |
         docker pull myapp:latest
         docker run --rm -v $(pwd):/data aquasec/trivy:latest image --exit-code 1 --severity HIGH,CRITICAL myapp:latest
       displayName: "Escaneo de vulnerabilidades en imagen Docker"
   ```

   **Explicación**:  
   - `docker pull` descarga la imagen.  
   - `trivy image` realiza el escaneo y falla el pipeline si encuentra vulnerabilidades **HIGH** o **CRITICAL**.

---

#### 4. **Publicación de reportes y resultados**  

Es importante compartir los resultados de los análisis de vulnerabilidades dentro del equipo:

- Publica los reportes como **artefactos** en Azure DevOps.  
- Configura alertas o notificaciones si se encuentran vulnerabilidades críticas.  

**Ejemplo de publicación de reporte**:  
```yaml
steps:
  - publish: 'reports/security-report.html'
    artifact: 'SecurityReport'
    displayName: "Publicar reporte de vulnerabilidades"
```

---

#### 5. **Buenas prácticas**  

1. **Automatiza el análisis en cada build**: integra herramientas en el pipeline para realizar análisis en cada commit.  
2. **Define umbrales de severidad**: falla el pipeline si se detectan vulnerabilidades críticas.  
3. **Mantén dependencias actualizadas**: usa herramientas como Dependabot o Renovate para automatizar la actualización de dependencias.  
4. **Revisa periódicamente**: realiza auditorías periódicas para asegurar que no queden vulnerabilidades sin resolver.  
5. **Monitoreo continuo**: configura alertas para nuevas vulnerabilidades en dependencias.

---

Con estas prácticas y herramientas, el análisis de vulnerabilidades en código y dependencias se convierte en un paso clave dentro del pipeline de Azure DevOps, asegurando aplicaciones seguras y de alta calidad.