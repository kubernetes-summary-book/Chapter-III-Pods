
# Pods

Un Pod es la unidad más pequeña de despliegue en Kubernetes. Pod es un conjunto de uno o más contenedores.

La recomendación general en Kubernetes es seguir el principio de "un proceso por contenedor" en un Pod. 
Un contenedor en Pod, sin embargo, hay situaciones en las que puede tener sentido ejecutar múltiples contenedores en un solo Pod:

    - Acoplamiento estrecho: Si dos aplicaciones necesitan comunicarse de manera muy estrecha y esenciales una para con la otra, puede ser útil ejecutarlas juntas en el mismo Pod.

	- Sidecar: Un patrón común es usar un contenedor adicional (llamado sidecar) para proporcionar funcionalidades adicionales a la aplicación principal, como la recolección de registros, monitoreo, o gestión de configuraciones.

	- Init Containers: `Init Containers` son contenedores que se ejecutan antes que los contenedores principales en un Pod y **se utilizan para tareas de inicialización, como la configuración o preparación del entorno**.

	------
		NOTA: Dos contenedores en un Pod NO pueden escuchar el MISMO puerto, pero pueden comunicarse en Localhost.
	------

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cap03-multicontenedor
spec:
  containers:
  - name: nginx  # Primer contenedor
    image: nginx:stable-alpine
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html  # Monta el volumen (html) en /usr/share/nginx/html
    ports:
    - containerPort: 80
  - name: logger  # Segundo contenedor
    image: alpine:latest
    command: ["/bin/sh", "-c"]  # Ejecutamos en el contenedor lo que hay en "args"
    args:
    - while true; do
        date >> /html/index.html;
        sleep 5;
      done
    volumeMounts:
    - name: html
      mountPath: /html
  volumes:  # Volumen creado
  - name: html
    emptyDir: {}  # Volumen efímero vacío
```

## Ciclo de vida de un Pod

Desde la petición de creación de Un Pod en Kubernetes hasta que que este se crea, o no, dicho Pod puede pasar por diferentes estados:
    - `Pending`: El pod ha sido aceptado por K8s, pero uno o más de los contenedores de dicho Pod no están listos para ser ejecutados.
    - `Running`: El Pod ha sido asignado a un nodo y todos los contenedores se han creado, y al menos un contenedor se está arrancando.
    - `Succeded`: Todos los contenedores del Pod se han arrancado correctamente.
    - `Failed`:  Todos los contenedores del Pod han termnado, y al menos uno de ellos se ha terminado con error, ya sea porque el contenedor ha terminado con un código de error o ha sido detenido por el sistema.
    - `Unknown`: Por algún motivo no se puede obtener el estado del Pod.

Del mismo modo que K8s gestiona los los Pods, también hace lo mismo con los contenedores dentro del Pod.
    - Running
    - Terminated
  
A Kubernetes también se le puede decir que hacer con un Pod en caso de que alguno de los contenedores termine:
    - Always
    - OnFailure
    - Never

```bash
    containers:
    - name: nginx
      image: nginx:stable-alpine
      restartPolicy: Always
```
 
### Contenedores de inicialización

Como hemos mencionado en este capítulo, dentro de un Pod se pueden ejecutar uno o más contenedores. 
Estos servicion son de larga duración, es decir, servicios que esperan peticiones, las procesan y reponden de alguna manera.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container-example 
spec:
  initContainers:
    - name: init
      image: alpine:latest
      command: ["/bin/sh", "-c"]
      args:
        - exit 0            # Indica al sistema que el proceso ha terminado correctamente.

  containers:
    - name: nginx
      image: nginx:stable-alpine
      ports:
        - containerPort: 80

```
Como podemos obervar en el ejemplo proporcionado, se ha definido un contenedor `nginx` en la sección **containers**, pero también se ha creado otro contenedor bajo la sección **initContainers** llamado `init` y usa una imagen apline:lates.

```bash
    kubectl describe po init-container-example
````

```bash
youneskabiri@Youness-MacBook-Pro Chapter-III-Pods % kubectl describe po init-container-example
Name:             init-container-example
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Thu, 11 Jul 2024 23:38:21 +0200
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.0.10
IPs:
  IP:  10.244.0.10
Init Containers:
  init:
    Container ID:  docker://28832ed30f11631e137084f7de555adc57443b87727641afabed7ea66e119add
    Image:         alpine:latest
    Image ID:      docker-pullable://alpine@sha256:b89d9c93e9ed3597455c90a0b88a8bbb5cb7188438f70953fede212a0c4394e0
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c
    Args:
      exit 0
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Thu, 11 Jul 2024 23:38:24 +0200
      Finished:     Thu, 11 Jul 2024 23:38:24 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-b82kl (ro)
Containers:
  nginx:
    Container ID:   docker://6813e8931caf16442a59d2b133cdef2acbbe7a2fd073fd656e577068514d2491
    Image:          nginx:stable-alpine
    Image ID:       docker-pullable://nginx@sha256:208ae3c180b7d26f6a8046fac4c8468b2ab8bd92123ab73f9c5ad0f6f1c5543d
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 11 Jul 2024 23:38:24 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-b82kl (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-b82kl:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  6m38s  default-scheduler  Successfully assigned default/init-container-example to minikube
  Normal  Pulling    6m38s  kubelet            Pulling image "alpine:latest"
  Normal  Pulled     6m36s  kubelet            Successfully pulled image "alpine:latest" in 1.202s (1.202s including waiting)
  Normal  Created    6m36s  kubelet            Created container init
  Normal  Started    6m36s  kubelet            Started container init
  Normal  Pulled     6m36s  kubelet            Container image "nginx:stable-alpine" already present on machine
  Normal  Created    6m36s  kubelet            Created container nginx
  Normal  Started    6m36s  kubelet            Started container nginx
  ```

### Saludo de los contenedores

Kubernetes usa tres tipos de chequeos para saber si un contenedor tiene problemas o está bien:
    
    - `livenessProbe`: Si este chequeo falla, kubectl mala al contendor.
    
    - `readinessProbe`: Este chequeo le dice a kubectl si el contenedor está listo para recibir paticiones.
   
    - `startupProbe`:


###
###


## Authors

- [@Younes Kabiri Farah](https://github.com/younesKabiriFarah)


## Acknowledgements

 - [Kubernetes](https://kubernetes.io/docs/home/)
 - [Book: Kubernetes para profesionales: Desde cero al despliegue de aplicaciones seguras y resilientes](https://0xword.com/es/libros/213-kubernetes-para-profesionales-desde-cero-al-despliegue-de-aplicaciones-seguras-y-resilientes.html)


## Features

- Exponiendo Pods
- Workloads
- Configuración de aplicaciones y secretos
- Selección de nodos
- Volúmenes Persistentes
- Autorización Basada en Roles
- Politicas de Red
- Contexto y Políticas de Seguridad
