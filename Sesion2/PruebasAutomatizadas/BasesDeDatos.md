### Pruebas con Bases de Datos (On-Premises y Azure SQL)

#### Introducción

Las pruebas de integración con bases de datos validan la interacción de una aplicación con la capa de persistencia de datos, asegurando que las operaciones como consultas y transacciones funcionen correctamente en escenarios reales.

Estas pruebas se aplican tanto para bases de datos locales (on-premises) como para servicios en la nube como **Azure SQL**. La configuración y ejecución varían ligeramente según el entorno.

---

#### Configuración del entorno de pruebas para bases de datos

1. **On-Premises**:
   - Requiere acceso a una base de datos local o un servidor de base de datos configurado dentro de una red privada.
   - Configuración típica:
     - **Base de datos de prueba dedicada**: Minimiza interferencias con datos de producción.
     - **Datos iniciales**: Se cargan mediante scripts SQL o frameworks de migración como Entity Framework.

2. **Azure SQL**:
   - Se utiliza un servidor de base de datos en la nube configurado específicamente para pruebas.
   - Configuración adicional:
     - **Cadenas de conexión seguras**: Usar Azure Key Vault para almacenar credenciales.
     - **Datos simulados**: Población de tablas con datos que imiten casos reales.

---

#### Flujo de trabajo para pruebas con bases de datos

1. **Preparación**:
   - Crear y configurar un entorno de base de datos de prueba.
   - Configurar las cadenas de conexión adecuadas en el código de la aplicación o mediante variables de entorno.

2. **Ejecución de pruebas**:
   - Utilizar frameworks como Entity Framework para realizar operaciones CRUD durante las pruebas.
   - Validar transacciones y consistencia de los datos.

3. **Limpieza posterior**:
   - Restaurar la base de datos al estado inicial después de cada prueba.
   - Usar herramientas como `Database Transactions` para aislar datos generados durante las pruebas.

---

#### Pruebas con bases de datos on-premises

1. **Configuración del servidor local**:
   - Configurar SQL Server o cualquier base de datos relacional local.
   - Proporcionar acceso a la base de datos de prueba desde el entorno de CI.

2. **Cadenas de conexión**:
   - Ejemplo para SQL Server:
     ```json
     {
         "ConnectionStrings": {
             "DefaultConnection": "Server=localhost;Database=TestDB;User Id=sa;Password=YourPassword;"
         }
     }
     ```

3. **Pruebas básicas**:
   - Código de ejemplo para pruebas unitarias en bases de datos on-premises:
     ```csharp
     [TestClass]
     public class LocalDatabaseTests
     {
         private TestDbContext _context;

         [TestInitialize]
         public void Setup()
         {
             _context = new TestDbContext("YourConnectionStringHere");
         }

         [TestMethod]
         public void ShouldAddRecordToDatabase()
         {
             var entity = new MyEntity { Name = "Test" };
             _context.MyEntities.Add(entity);
             _context.SaveChanges();

             var result = _context.MyEntities.FirstOrDefault(e => e.Name == "Test");
             Assert.IsNotNull(result);
         }

         [TestCleanup]
         public void Cleanup()
         {
             _context.Database.ExecuteSqlCommand("DELETE FROM MyEntities");
         }
     }
     ```

---

#### Pruebas con Azure SQL

1. **Configuración del servidor en Azure**:
   - Crear una base de datos Azure SQL desde el portal de Azure.
   - Configurar el firewall de Azure SQL para permitir acceso desde las direcciones IP del entorno de CI.

2. **Conexión segura**:
   - Usar Azure Key Vault para almacenar credenciales y cadenas de conexión.
   - Ejemplo de configuración en Azure DevOps:
     ```yaml
     steps:
       - task: AzureKeyVault@1
         inputs:
           azureSubscription: 'YourAzureSubscription'
           KeyVaultName: 'YourKeyVaultName'
           SecretsFilter: 'SqlConnectionString'
       - script: |
           echo "##vso[task.setvariable variable=SqlConnectionString]$(SqlConnectionString)"
         displayName: 'Configurar cadena de conexión desde Key Vault'
     ```

3. **Ejemplo de prueba**:
   - Código C# para interactuar con Azure SQL:
     ```csharp
     [TestClass]
     public class AzureDatabaseTests
     {
         private string _connectionString = Environment.GetEnvironmentVariable("SqlConnectionString");

         [TestMethod]
         public void ShouldQueryAzureSQLDatabase()
         {
             using (var connection = new SqlConnection(_connectionString))
             {
                 connection.Open();
                 var command = new SqlCommand("SELECT COUNT(*) FROM MyTable", connection);
                 var count = (int)command.ExecuteScalar();

                 Assert.IsTrue(count >= 0, "Table has records or is empty");
             }
         }
     }
     ```

---

#### Integración en un pipeline de CI

1. **On-Premises**:
   - Conectar el agente de build a la base de datos local.
   - YAML de ejemplo:
     ```yaml
     steps:
       - script: dotnet test --filter Category=DatabaseTests
         displayName: 'Ejecutar pruebas con base de datos on-premises'
     ```

2. **Azure SQL**:
   - Usar Azure Key Vault y agentes de build en la nube:
     ```yaml
     steps:
       - task: AzureKeyVault@1
         inputs:
           azureSubscription: 'YourAzureSubscription'
           KeyVaultName: 'YourKeyVaultName'
       - script: dotnet test --filter Category=DatabaseTests
         displayName: 'Ejecutar pruebas con base de datos Azure SQL'
     ```

---

#### Mejores prácticas

1. **Datos aislados por prueba**:
   - Utilizar transacciones o truncar tablas después de cada prueba.

2. **Automatización de migraciones**:
   - Usar herramientas como Entity Framework para garantizar que las bases de datos de prueba estén sincronizadas con el esquema actual.

3. **Simulación de fallos**:
   - Introducir escenarios de error controlados, como pérdida de conexión o tiempos de espera, para probar la resiliencia de la aplicación.