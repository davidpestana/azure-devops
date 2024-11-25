# Documentación: Curso CI/CD con .NET y Azure

## Índice
1. [Sesión 1: Introducción a CI/CD y Herramientas en Azure](#sesión-1-introducción-a-cicd-y-herramientas-en-azure)
2. [Sesión 2: Pipeline de Integración Continua (CI)](#sesión-2-pipeline-de-integración-continua-ci)
3. [Sesión 3: Pipeline de Entrega Continua (CD)](#sesión-3-pipeline-de-entrega-continua-cd)
4. [Sesión 4: Gestión Avanzada y Optimización de CI/CD](#sesión-4-gestión-avanzada-y-optimización-de-cicd)

---

## Sesión 1: Introducción a CI/CD y Herramientas en Azure

### Objetivos
- Comprender los conceptos clave de CI/CD.
- Familiarizarse con las herramientas de Azure para CI/CD.
- Configurar el entorno inicial para despliegues en distintos escenarios.

### Contenidos
1. Conceptos básicos de CI/CD:
   - [Definición y flujo de trabajo](./Sesion1/ConceptosBasicos/DefinicionYFlujo.md)
2. Casos de uso para aplicaciones .NET:
   - [On-premises (Windows Server, IIS)](./Sesion1/CasosDeUso/OnPremises.md)
   - [En la nube (Azure App Service, Kubernetes)](./Sesion1/CasosDeUso/Nube.md)
   - [Serverless (.NET en Azure Functions o AWS Lambda)](./Sesion1/CasosDeUso/Serverless.md)
3. Introducción a Azure DevOps:
   - [Servicios principales y flujo de trabajo en Azure DevOps](./Sesion1/IntroduccionAzureDevOps/Servicios.md)
   - [Alternativa con GitHub Actions](./Sesion1/IntroduccionAzureDevOps/GitHubActions.md)
4. Configuración inicial del entorno:
   - [Creación de un proyecto en Azure DevOps](./Sesion1/ConfiguracionInicial/CrearProyecto.md)
   - [Configuración de repositorios para .NET](./Sesion1/ConfiguracionInicial/ConfigRepositorios.md)
   - [Conexión entre credenciales para despliegues on-premises y en la nube](./Sesion1/ConfiguracionInicial/ConexionCredenciales.md)

### Laboratorio práctico
- [Configuración inicial de un proyecto .NET en Azure DevOps](./Sesion1/Laboratorio/ConfiguracionInicial.md)

---

## Sesión 2: Pipeline de Integración Continua (CI)

### Objetivos
- Configurar un pipeline de CI para automatizar builds y pruebas.
- Soportar aplicaciones .NET en entornos mixtos.

### Contenidos
1. Conceptos de CI en .NET:
   - [Flujo para ASP.NET Core y Framework](./Sesion2/ConceptosCI/Flujo.md)
   - [Herramientas de pruebas automatizadas (MSTest, xUnit)](./Sesion2/ConceptosCI/PruebasAutomatizadas.md)
2. Configuración de un pipeline de CI:
   - [Creación de pipelines en YAML](./Sesion2/ConfiguracionPipeline/CreacionYAML.md)
   - [Uso de agentes de build para Windows y Azure](./Sesion2/ConfiguracionPipeline/AgentesBuild.md)
3. Pruebas automatizadas en CI:
   - [Ejecución de pruebas unitarias y de integración](./Sesion2/PruebasAutomatizadas/EjecucionPruebas.md)
   - [Pruebas con bases de datos (on-premises y Azure SQL)](./Sesion2/PruebasAutomatizadas/BasesDeDatos.md)

### Laboratorio práctico
- [Configurar un pipeline de CI para ASP.NET Core](./Sesion2/Laboratorio/PipelineCI.md)

---

## Sesión 3: Pipeline de Entrega Continua (CD)

### Objetivos
- Diseñar pipelines de CD para despliegues on-premises, cloud y serverless.
- Integrar herramientas de infraestructura como código (IaC).

### Contenidos
1. Casos de uso de CD:
   - [Despliegue en IIS con Web Deploy](./Sesion3/CasosDeUso/IISWebDeploy.md)
   - [Implementación en Azure App Service con slots](./Sesion3/CasosDeUso/AppServiceSlots.md)
   - [Despliegue en Kubernetes](./Sesion3/CasosDeUso/Kubernetes.md)
   - [Configuración en Azure Functions](./Sesion3/CasosDeUso/AzureFunctions.md)
2. Configuración del pipeline de CD:
   - [Tareas de despliegue en distintos entornos](./Sesion3/ConfiguracionPipeline/TareasDespliegue.md)
   - [Gestión de secretos con Azure Key Vault](./Sesion3/ConfiguracionPipeline/GestionSecretos.md)
   - [Estrategias de despliegue: Canary, Blue-Green y Rolling Updates](./Sesion3/ConfiguracionPipeline/EstrategiasDespliegue.md)

### Laboratorio práctico
- [Crear un pipeline de CD para ASP.NET Core](./Sesion3/Laboratorio/PipelineCD.md)

---

## Sesión 4: Gestión Avanzada y Optimización de CI/CD

### Objetivos
- Incorporar prácticas avanzadas y optimizar pipelines.
- Mejorar seguridad y monitoreo de pipelines.

### Contenidos
1. Optimización de pipelines:
   - [Reducción de tiempos de ejecución](./Sesion4/Optimizacion/ReduccionTiempos.md)
   - [Uso de plantillas YAML reutilizables](./Sesion4/Optimizacion/PlantillasYAML.md)
   - [Automatización de despliegues en múltiples entornos](./Sesion4/Optimizacion/DesplieguesMultientornos.md)
2. Monitoreo y seguridad:
   - [Seguimiento con Application Insights](./Sesion4/Monitoreo/SeguimientoInsights.md)
   - [Análisis de vulnerabilidades en código y dependencias](./Sesion4/Monitoreo/VulnerabilidadesCodigo.md)
   - [Gestión de secretos con Key Vault](./Sesion4/Monitoreo/GestionSecretos.md)
3. Caso de uso integral:
   - [Proyecto final: Pipeline completo de CI/CD](./Sesion4/CasoIntegral/ProyectoFinal.md)

### Laboratorio práctico
- [Añadir monitorización con Application Insights](./Sesion4/Laboratorio/ApplicationInsights.md)
