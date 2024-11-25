# Herramientas de Pruebas Automatizadas: **MSTest** y **xUnit**

Las pruebas automatizadas son un componente esencial en cualquier flujo de desarrollo moderno, ya que ayudan a garantizar la calidad del código y detectar errores rápidamente. En el ecosistema .NET, dos herramientas principales para pruebas son **MSTest** y **xUnit**. Ambas son ampliamente usadas y compatibles con frameworks como **ASP.NET Core** y **ASP.NET Framework**.

---

## **1. ¿Qué es MSTest?**

**MSTest** es el marco de pruebas integrado de Microsoft para proyectos .NET. Se utiliza ampliamente en proyectos que dependen de herramientas de Microsoft, como Visual Studio, y proporciona compatibilidad directa con proyectos de pruebas creados en el IDE.

### **Características de MSTest**
- Integración nativa con Visual Studio.
- Soporte para pruebas unitarias y de integración.
- Amplio conjunto de atributos (`[TestMethod]`, `[TestClass]`, `[TestInitialize]`).
- Compatibilidad con Azure DevOps para ejecutar pruebas en pipelines.

---

### **Ejemplo Básico con MSTest**

#### **1. Crear un Proyecto de Pruebas**
1. En Visual Studio, selecciona **Agregar nuevo proyecto**.
2. Elige **Proyecto de pruebas unitarias (.NET)**.
3. Configura el nombre del proyecto, como `MyApp.Tests`.

#### **2. Estructura Básica de MSTest**
```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace MyApp.Tests
{
    [TestClass]
    public class CalculatorTests
    {
        private Calculator _calculator;

        [TestInitialize]
        public void Setup()
        {
            _calculator = new Calculator();
        }

        [TestMethod]
        public void Add_TwoNumbers_ReturnsSum()
        {
            // Arrange
            int a = 5;
            int b = 3;

            // Act
            int result = _calculator.Add(a, b);

            // Assert
            Assert.AreEqual(8, result);
        }
    }
}
```

### **Explicación del Código**
- **[TestClass]**: Declara que esta clase contiene pruebas.
- **[TestInitialize]**: Método que se ejecuta antes de cada prueba para configurar el estado inicial.
- **[TestMethod]**: Marca un método como prueba.
- **Assert**: Valida que el resultado esperado coincida con el resultado real.

---

### **Ejecución de Pruebas con MSTest**
1. Abre el **Explorador de Pruebas** en Visual Studio.
2. Ejecuta las pruebas manualmente desde el IDE o usa el siguiente comando:
   ```bash
   dotnet test
   ```

---

## **2. ¿Qué es xUnit?**

**xUnit** es un marco de pruebas moderno y ligero diseñado para proyectos .NET Core. Es una evolución de **NUnit**, con un enfoque en la simplicidad y extensibilidad.

### **Características de xUnit**
- Independiente de Visual Studio (compatible con múltiples IDEs y CI/CD pipelines).
- Pruebas paralelas por defecto, lo que mejora el rendimiento.
- Diseño más limpio y enfocado (sin atributos redundantes como `[TestClass]`).
- Extensibilidad mediante **Fixtures** y **Traits**.

---

### **Ejemplo Básico con xUnit**

#### **1. Crear un Proyecto de Pruebas**
1. En Visual Studio o CLI, crea un nuevo proyecto de pruebas:
   ```bash
   dotnet new xunit -n MyApp.Tests
   ```

#### **2. Estructura Básica de xUnit**
```csharp
using Xunit;

namespace MyApp.Tests
{
    public class CalculatorTests
    {
        private readonly Calculator _calculator;

        public CalculatorTests()
        {
            _calculator = new Calculator();
        }

        [Fact]
        public void Add_TwoNumbers_ReturnsSum()
        {
            // Arrange
            int a = 5;
            int b = 3;

            // Act
            int result = _calculator.Add(a, b);

            // Assert
            Assert.Equal(8, result);
        }
    }
}
```

### **Explicación del Código**
- **[Fact]**: Marca un método como una prueba.
- **Assert.Equal**: Valida que el resultado esperado coincida con el resultado real.
- **Constructor**: xUnit no utiliza `[TestInitialize]`; en su lugar, puedes usar el constructor para inicializar los objetos necesarios.

---

### **Pruebas Avanzadas con xUnit**
#### **Pruebas Parametrizadas**
```csharp
[Theory]
[InlineData(1, 2, 3)]
[InlineData(-1, -1, -2)]
public void Add_VariousInputs_ReturnsCorrectSum(int a, int b, int expected)
{
    // Act
    int result = _calculator.Add(a, b);

    // Assert
    Assert.Equal(expected, result);
}
```
- **[Theory]**: Define pruebas parametrizadas.
- **[InlineData]**: Proporciona diferentes conjuntos de datos para la prueba.

#### **Uso de Fixtures para Compartir Recursos**
```csharp
public class CalculatorFixture : IDisposable
{
    public Calculator Calculator { get; private set; }

    public CalculatorFixture()
    {
        Calculator = new Calculator();
    }

    public void Dispose()
    {
        // Liberar recursos
    }
}

public class CalculatorTests : IClassFixture<CalculatorFixture>
{
    private readonly Calculator _calculator;

    public CalculatorTests(CalculatorFixture fixture)
    {
        _calculator = fixture.Calculator;
    }

    [Fact]
    public void Add_TwoNumbers_ReturnsSum()
    {
        int result = _calculator.Add(2, 3);
        Assert.Equal(5, result);
    }
}
```

---

### **Ejecución de Pruebas con xUnit**
Ejecuta las pruebas con el comando:
```bash
dotnet test
```

---

## **Comparativa: MSTest vs. xUnit**

| Característica              | MSTest                       | xUnit                         |
|-----------------------------|------------------------------|-------------------------------|
| **Integración con Visual Studio** | Nativa                        | Complementaria (requiere paquete) |
| **Pruebas paralelas**       | No                           | Sí                            |
| **Extensibilidad**          | Limitada                    | Amplia                        |
| **Simplicidad**             | Atributos redundantes       | Menos atributos               |
| **Pruebas parametrizadas**  | Soporte básico              | Muy avanzado                  |

---

## **Uso en CI/CD Pipelines**

Ambas herramientas se integran fácilmente con Azure Pipelines:

### **Ejemplo en YAML (xUnit o MSTest)**
```yaml
steps:
- script: dotnet restore
  displayName: 'Restore dependencies'

- script: dotnet build --configuration Release
  displayName: 'Build project'

- script: dotnet test --configuration Release
  displayName: 'Run tests'
```

---

## **Conclusión**

- **MSTest** es ideal para proyectos que dependen de herramientas de Microsoft, como Visual Studio y Azure DevOps.
- **xUnit** es preferible para proyectos modernos, especialmente aquellos que requieren extensibilidad y paralelismo.

Ambas son herramientas poderosas que garantizan la calidad del código y son compatibles con flujos de trabajo de CI/CD en Azure DevOps y otras plataformas.