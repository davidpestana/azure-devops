# Casos de Uso para Aplicaciones .NET: On-Premises (Windows Server, IIS)

## Introducción

El despliegue de aplicaciones .NET en entornos **on-premises** (en instalaciones locales) es una práctica común en organizaciones que, por razones legales, de seguridad o de infraestructura, prefieren mantener sus aplicaciones dentro de su propia red. **Internet Information Services (IIS)** es el servidor web predilecto en entornos Windows para hospedar aplicaciones .NET Framework o .NET Core en este escenario.

---

## Beneficios de Usar IIS en On-Premises

1. **Control completo**: La organización gestiona directamente el hardware, el sistema operativo y la red.
2. **Compatibilidad con .NET Framework y .NET Core**: IIS ofrece soporte nativo para ambas versiones del framework.
3. **Integración con Active Directory (AD)**: Ideal para aplicaciones empresariales que requieren autenticación de usuarios dentro de la organización.
4. **Optimización para Windows Server**: IIS está diseñado para aprovechar las capacidades de Windows Server, como la administración de recursos y la seguridad.

---

## Requisitos para el Despliegue On-Premises con IIS

1. **Hardware**
   - Servidores físicos o máquinas virtuales con Windows Server.
   - Recomendaciones:
     - Mínimo 4 GB de RAM.
     - Procesador de 2 núcleos para cargas pequeñas.

2. **Software**
   - **Windows Server**: Versiones compatibles con la aplicación (Windows Server 2016 o superior para .NET Core).
   - **IIS (Internet Information Services)**:
     - Debe estar habilitado y configurado en el servidor.
   - **Framework de .NET**:
     - Instalado y configurado según las necesidades de la aplicación.
   - **Base de datos (opcional)**:
     - SQL Server local o remoto para aplicaciones que requieran almacenamiento.

3. **Seguridad**
   - Certificados SSL/TLS para tráfico HTTPS.
   - Configuración de firewalls para permitir únicamente el tráfico necesario.

---

## Flujo de Trabajo para el Despliegue On-Premises en IIS

### **1. Preparar el Servidor Windows**
   - Asegurarse de que Windows Server esté actualizado.
   - Instalar y habilitar IIS:
     1. Abrir el Administrador del Servidor.
     2. Seleccionar "Agregar roles y características".
     3. Habilitar el rol de "Servidor Web (IIS)" y los módulos necesarios (como ASP.NET Core o ASP.NET Framework).

### **2. Configurar IIS**
   - Crear un nuevo **sitio web** en IIS:
     1. Abrir el Administrador de IIS.
     2. Crear un nuevo sitio web y asignarle un nombre.
     3. Especificar la ruta física donde se encuentra la aplicación publicada.
     4. Asignar un puerto y un nombre de host (dominio o IP).

   - Configurar los **bindings**:
     - Agregar un binding HTTPS y asignar un certificado SSL válido.

   - Habilitar las versiones necesarias del framework .NET:
     - Usar el **Administrador de características de Windows** para habilitar las extensiones requeridas (como ASP.NET 4.8 o el Módulo de .NET Core).

### **3. Publicar la Aplicación**
   - **Para aplicaciones .NET Framework**:
     1. Publicar el proyecto desde Visual Studio en formato de archivo (`bin`).
     2. Copiar los archivos al servidor mediante FTP o una herramienta como RoboCopy.
   - **Para aplicaciones .NET Core**:
     - Usar **Web Deploy** o Docker si la aplicación está en un contenedor.

### **4. Configurar la Base de Datos**
   - Si la aplicación depende de SQL Server:
     1. Instalar SQL Server en el mismo servidor o en un servidor dedicado.
     2. Configurar cadenas de conexión en el archivo de configuración de la aplicación (`appsettings.json` o `web.config`).

### **5. Realizar Pruebas**
   - Validar que el sitio esté accesible a través de HTTP/HTTPS.
   - Asegurarse de que la autenticación, la base de datos y otras dependencias funcionen correctamente.

---

## Casos de Uso Típicos

1. **Aplicaciones Empresariales Internas**:
   - Sistemas de gestión de recursos humanos (HRM) o ERP que requieren acceso exclusivo dentro de la red corporativa.

2. **Sistemas Críticos de Negocio**:
   - Aplicaciones que manejan datos sensibles y no pueden exponerse a internet.

3. **Integración con Sistemas Locales**:
   - Aplicaciones que dependen de hardware local (impresoras, sensores, etc.) o sistemas legados.

---

## Estrategias de Despliegue

1. **Manual**:
   - Para entornos pequeños o aplicaciones con baja frecuencia de cambios.

2. **Automatizado con Herramientas**:
   - **Web Deploy**: Permite publicar directamente desde Visual Studio a IIS.
   - **Azure DevOps o GitHub Actions**:
     - Configurar pipelines para construir y desplegar en servidores on-premises.

---

## Desafíos y Soluciones

1. **Desafío: Seguridad del servidor IIS**
   - Solución: Implementar reglas de firewall estrictas y habilitar solo protocolos HTTPS.

2. **Desafío: Escalabilidad limitada**
   - Solución: Configurar balanceadores de carga (como NLB en Windows Server) para distribuir el tráfico.

3. **Desafío: Mantenimiento manual**
   - Solución: Automatizar actualizaciones con scripts o herramientas CI/CD.

---

Este enfoque es ideal para organizaciones que no pueden mover sus aplicaciones a la nube o que tienen dependencias locales críticas. Sin embargo, siempre es importante considerar la transición gradual hacia entornos híbridos o completamente en la nube para aprovechar los beneficios de escalabilidad y flexibilidad.