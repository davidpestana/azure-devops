### Ejecución de Pruebas Unitarias y de Integración

#### Conceptos clave

1. **Pruebas Unitarias**:
   - Validan el comportamiento de componentes individuales de una aplicación (métodos, clases, funciones).
   - Se enfocan en verificar que una unidad de código funcione como se espera.
   - Herramientas comunes para .NET:
     - **MSTest**: Framework nativo de pruebas unitarias en .NET.
     - **xUnit**: Framework popular y ligero para pruebas unitarias.
     - **NUnit**: Framework flexible y ampliamente utilizado para pruebas unitarias.

2. **Pruebas de Integración**:
   - Validan que diferentes módulos o componentes funcionen juntos correctamente.
   - Simulan escenarios reales para verificar que las interacciones entre componentes sean coherentes.
   - En .NET, se pueden realizar pruebas de integración con bases de datos, APIs y servicios externos.

#### Flujo de trabajo para pruebas en un pipeline de CI

1. **Configuración del entorno de pruebas**:
   - Definir dependencias necesarias, como bases de datos o configuraciones de archivos.
   - Crear datos simulados (fixtures o mocks) para pruebas de integración.

2. **Ejecución de pruebas unitarias**:
   - Ejecutar las pruebas unitarias automáticamente durante el pipeline CI.
   - Generar informes de resultados para identificar fallos rápidamente.

3. **Ejecución de pruebas de integración**:
   - Asegurarse de que los servicios dependientes estén disponibles (pueden ser simulados mediante mocks o contenedores).
   - Probar escenarios clave, como conexión a bases de datos, respuestas de APIs y flujos completos de la aplicación.

#### Configuración en Azure DevOps Pipelines

1. **Archivo YAML para pruebas**:
   - Configuración básica para ejecutar pruebas:
     ```yaml
     trigger:
       - main
     pool:
       vmImage: 'windows-latest'

     steps:
       - task: UseDotNet@2
         inputs:
           packageType: 'sdk'
           version: '6.x' # Versión de .NET

       - script: dotnet restore
         displayName: 'Restaurar dependencias'

       - script: dotnet build --configuration Release
         displayName: 'Compilar solución'

       - script: dotnet test --configuration Release --logger:trx
         displayName: 'Ejecutar pruebas unitarias'
     ```

2. **Agregar pruebas de integración**:
   - Simular servicios con Docker para pruebas de integración.
   - Configuración YAML extendida:
     ```yaml
     steps:
       - script: docker-compose -f docker-compose.test.yml up -d
         displayName: 'Iniciar servicios simulados'

       - script: dotnet test --configuration Release --filter "TestCategory=Integration"
         displayName: 'Ejecutar pruebas de integración'

       - script: docker-compose -f docker-compose.test.yml down
         displayName: 'Detener servicios simulados'
     ```

3. **Resultados de las pruebas**:
   - Configurar Azure DevOps para mostrar resultados de pruebas en la interfaz:
     ```yaml
     steps:
       - task: PublishTestResults@2
         inputs:
           testResultsFormat: 'VSTest'
           testResultsFiles: '**/*.trx'
         displayName: 'Publicar resultados de pruebas'
     ```

#### Pruebas con bases de datos

1. **Configuración local para pruebas**:
   - Usar contenedores para bases de datos como SQL Server o Azure SQL.
   - Inicializar la base de datos con un script o migraciones antes de las pruebas.

2. **Validación de flujos de datos**:
   - Probar operaciones CRUD (Crear, Leer, Actualizar, Eliminar).
   - Verificar integridad y consistencia de los datos.

3. **Ejemplo de prueba de integración con base de datos**:
   - Código en C#:
     ```csharp
     [TestClass]
     public class DatabaseTests
     {
         [TestMethod]
         public void ShouldInsertRecord()
         {
             using (var context = new TestDbContext())
             {
                 var entity = new MyEntity { Name = "Test" };
                 context.MyEntities.Add(entity);
                 context.SaveChanges();

                 Assert.AreEqual(1, context.MyEntities.Count());
             }
         }
     }
     ```