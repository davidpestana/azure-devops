### Estrategias de despliegue: Canary, Blue-Green y Rolling Updates

Las estrategias de despliegue son esenciales para garantizar transiciones seguras entre versiones de una aplicación. Este contenido equilibra escenarios **on-premises con IIS**, incluyendo ejemplos con **Application Request Routing (ARR)**, junto con aplicaciones en **Azure App Service**, **Azure Functions**, y **Kubernetes**.

---

#### **1. Canary Deployment**

**Descripción:**
- Consiste en desplegar una nueva versión para un subconjunto limitado de usuarios o tráfico, mientras el resto continúa accediendo a la versión actual.
- Es ideal para validar cambios en producción con riesgos controlados.

---

##### **Escenarios y Ejemplos**

1. **IIS on-premises:**
   - Publicar la nueva versión en un servidor adicional o una carpeta virtual de IIS.
   - Configurar ARR (Application Request Routing) o un balanceador externo para redirigir un porcentaje del tráfico.

   **Configuración en ARR para tráfico canary (por ejemplo, 10%):**
   ```xml
   <rewrite>
     <rules>
       <rule name="Canary Deployment">
         <conditions>
           <add input="{RANDOM}" pattern="^([0-9])$" /> <!-- 10% tráfico -->
         </conditions>
         <action type="Redirect" url="http://canary-server/app" />
       </rule>
     </rules>
   </rewrite>
   ```

2. **Azure App Service:**
   - Utilizar deployment slots (por ejemplo, `Production` y `Staging`).
   - Configurar el tráfico progresivo hacia el slot `Staging` desde el portal de Azure o el pipeline de CD.

3. **Kubernetes:**
   - Desplegar un nuevo `Deployment` con menos réplicas para la versión canary.
   - Configurar un `Service` o un `Ingress` para redirigir un porcentaje del tráfico.

   **Ejemplo en Kubernetes:**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-canary
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: myapp
         version: canary
     template:
       metadata:
         labels:
           app: myapp
           version: canary
       spec:
         containers:
         - name: myapp
           image: myapp:canary
   ```

---

#### **2. Blue-Green Deployment**

**Descripción:**
- Mantiene dos entornos completamente separados:
  - **Blue (Producción):** Versión actualmente en uso.
  - **Green (Nueva versión):** Versión en pruebas.
- Cambia el tráfico al entorno green una vez validado.

---

##### **Escenarios y Ejemplos**

1. **IIS on-premises:**
   - Publicar la nueva versión en un servidor o sitio separado (green).
   - Redirigir el tráfico al sitio green mediante ARR o balanceadores externos.

   **Ejemplo de configuración en ARR para Blue-Green:**
   ```xml
   <rewrite>
     <rules>
       <rule name="Switch to Green">
         <action type="Redirect" url="http://green-server/app" />
       </rule>
     </rules>
   </rewrite>
   ```

2. **Azure App Service:**
   - Desplegar la nueva versión en el slot `Staging`.
   - Realizar un swap entre `Staging` y `Production` una vez validada la nueva versión.

3. **Kubernetes:**
   - Crear dos conjuntos de `Deployments` para las versiones blue y green.
   - Cambiar el selector del `Service` para redirigir el tráfico al entorno green.

   **Ejemplo en Kubernetes:**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-blue
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: myapp
         version: blue
     template:
       metadata:
         labels:
           app: myapp
           version: blue
       spec:
         containers:
         - name: myapp
           image: myapp:blue
   ---
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-green
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: myapp
         version: green
     template:
       metadata:
         labels:
           app: myapp
           version: green
       spec:
         containers:
         - name: myapp
           image: myapp:green
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: myapp-service
   spec:
     selector:
       app: myapp
       version: blue # Cambiar a green para redirigir el tráfico
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   ```

---

#### **3. Rolling Updates**

**Descripción:**
- Reemplaza gradualmente las instancias de la aplicación con la nueva versión.
- Garantiza disponibilidad continua, ya que siempre hay instancias activas para manejar el tráfico.

---

##### **Escenarios y Ejemplos**

1. **IIS on-premises:**
   - Actualizar los servidores de un clúster uno por uno.
   - Retirar temporalmente cada servidor del balanceador de carga, actualizarlo y reintegrarlo.

   **Pasos para ARR con Rolling Updates:**
   - Configurar ARR para manejar tráfico en tiempo real.
   - Deshabilitar temporalmente un servidor desde la consola ARR mientras se actualiza.

2. **Azure App Service:**
   - Usar deployment slots y configurar réplicas en el slot `Staging`.
   - Actualizar las réplicas gradualmente antes de un swap completo.

3. **Kubernetes:**
   - Configurar el `Deployment` para usar la estrategia `RollingUpdate`, que es nativa en Kubernetes.
   - Ajustar los parámetros `maxUnavailable` y `maxSurge` para controlar la progresión.

   **Ejemplo en Kubernetes:**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp
   spec:
     replicas: 3
     strategy:
       type: RollingUpdate
       rollingUpdate:
         maxUnavailable: 1
         maxSurge: 1
     selector:
       matchLabels:
         app: myapp
     template:
       metadata:
         labels:
           app: myapp
       spec:
         containers:
         - name: myapp
           image: myapp:new
   ```

---

#### **Comparación de Estrategias**

| Estrategia       | Downtime | Recursos duplicados | Riesgo inicial | Rollback rapidez | Escenarios ideales                    |
|-------------------|----------|---------------------|----------------|------------------|---------------------------------------|
| **Canary**        | No       | No                  | Bajo           | Moderado         | IIS, App Service, Kubernetes          |
| **Blue-Green**    | No       | Sí                  | Nulo           | Alto             | IIS, App Service, Kubernetes          |
| **Rolling Update**| No       | No                  | Moderado       | Automático       | IIS con balanceo, App Service, K8s    |

---

### **Conclusión**

Para **IIS on-premises**, las estrategias **Canary Deployment** y **Blue-Green Deployment** son particularmente efectivas al usarse con ARR o balanceadores de carga. En escenarios en la nube, como **Azure App Service** y **Kubernetes**, la estrategia **Rolling Updates** destaca por su capacidad de manejar despliegues escalonados sin interrupciones. Proveer ejemplos detallados para cada tecnología asegura que estas estrategias puedan aplicarse con éxito en entornos diversos y robustos.