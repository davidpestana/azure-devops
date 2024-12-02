### Despliegue en Kubernetes

El despliegue de aplicaciones en Kubernetes proporciona alta disponibilidad, escalabilidad y gestión avanzada para aplicaciones modernas. Este enfoque es ideal para microservicios y aplicaciones distribuidas.

---

#### **1. Preparativos previos**

1. **Crear un clúster de Kubernetes:**
   - Utilizar un servicio gestionado como Azure Kubernetes Service (AKS) o configurar un clúster manualmente.
   - Asegurarse de tener acceso al clúster mediante `kubectl`.

2. **Configurar el entorno local:**
   - Instalar `kubectl` para interactuar con el clúster.
   - Instalar `Helm` si se utilizan charts para el despliegue.
   - Crear un archivo `kubeconfig` para autenticar el acceso al clúster.

3. **Preparar la aplicación .NET:**
   - Asegurarse de que la aplicación esté empaquetada como una imagen Docker.
   - Publicar la imagen en un registro de contenedores (por ejemplo, Azure Container Registry o Docker Hub).

---

#### **2. Crear manifiestos de Kubernetes**

1. **Archivo `Deployment`:**
   Define el pod y la estrategia de despliegue.

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-deployment
     labels:
       app: myapp
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: myapp
     template:
       metadata:
         labels:
           app: myapp
       spec:
         containers:
         - name: myapp-container
           image: <docker_registry>/<image_name>:<tag>
           ports:
           - containerPort: 80
   ```

2. **Archivo `Service`:**
   Expone el despliegue a través de un servicio.

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: myapp-service
   spec:
     selector:
       app: myapp
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
     type: LoadBalancer
   ```

3. **Archivo `Ingress` (opcional):**
   Configura reglas para acceder a la aplicación.

   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: myapp-ingress
   spec:
     rules:
     - host: myapp.example.com
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: myapp-service
               port:
                 number: 80
   ```

---

#### **3. Configuración del pipeline en Azure DevOps**

##### YAML para el despliegue:

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: Docker@2
    inputs:
      containerRegistry: '<AzureContainerRegistry>'
      repository: '<image_name>'
      command: 'buildAndPush'
      Dockerfile: '**/Dockerfile'
      tags: '$(Build.BuildId)'

  - task: Kubernetes@1
    inputs:
      connectionType: 'Azure Resource Manager'
      azureSubscription: '<AzureSubscription>'
      azureResourceGroup: '<ResourceGroup>'
      kubernetesCluster: '<AKSClusterName>'
      namespace: '<Namespace>'
      command: 'apply'
      useConfigurationFile: true
      configuration: 'manifests/*.yaml'
```

1. **Detalles clave:**
   - Construcción y publicación de la imagen Docker.
   - Autenticación con el clúster de AKS utilizando Azure Resource Manager.
   - Aplicación de manifiestos de Kubernetes desde la carpeta `manifests`.

---

#### **4. Estrategias de despliegue**

1. **Rolling Updates:**
   - Actualiza los pods gradualmente sin interrupción del servicio.
   - Configurado por defecto en el recurso `Deployment`.

2. **Canary Deployments:**
   - Despliega una nueva versión para una fracción de usuarios antes de realizar el despliegue completo.
   - Utiliza múltiples `Deployment` y un balanceador de carga configurado.

3. **Blue-Green Deployments:**
   - Mantiene una versión "blue" (producción) y despliega una nueva versión "green" para validación antes de realizar el cambio de tráfico.

---

#### **5. Pruebas posteriores al despliegue**

1. **Verificar los recursos:**
   - Listar pods y verificar que están en estado `Running`:
     ```bash
     kubectl get pods
     ```
   - Verificar el servicio:
     ```bash
     kubectl get services
     ```

2. **Acceder a la aplicación:**
   - Si se utiliza un `LoadBalancer`, acceder mediante la IP pública.
   - Si se utiliza `Ingress`, verificar la URL configurada.

3. **Monitorear logs:**
   - Revisar los logs de los pods:
     ```bash
     kubectl logs <pod_name>
     ```

---

#### **6. Ventajas del despliegue en Kubernetes**

- **Escalabilidad automática:** Ajusta los recursos según la demanda.
- **Alta disponibilidad:** Los pods se distribuyen en múltiples nodos del clúster.
- **Gestión avanzada:** Kubernetes permite orquestar despliegues complejos con facilidad.

---

Este enfoque es ideal para aplicaciones modernas que requieren entornos flexibles, escalables y resistentes, ofreciendo un control granular sobre todo el ciclo de vida del despliegue.