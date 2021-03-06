# Kubernetes.

## kubernetes cmd.

Lista todos los nodos que tiene nuestro cluster.

```bash 
kubectl get nodes
```

Puedes pasarle el archivo de configuración en caso de estar usando uno diferente.

```bash 
kubectl --config
```

Especificas la configuración sin necesidad de darle un archivo.

```bash 
kubectl --server --user
```
Muestra más datos de los nodos.
```bash 
kubectl get nodes -a wide: 
```

Da mucha información de ese nodo en especifico.
```bash 
kubectl describe nodes node1
```

Permite ver la definición de todo lo relacionado a ese nodo.

```bash 
kubectl explain node
```

## Manos en la masa.

Namespace, aisla la visibilidad de los pods, los secrets.

```bash 
kubectl get secrets -n kube-public
```

Crear un pod.

```bash
kubectl create pingpong --image alpine ping 1.1.1.1
```

Ver todos los recursos de nuestro cluster.

```bash
kubectl get all 
```

Ler los logs del pod poingpong.

```bash
kubectl logs pingpong 
```

Ver los últimas 20 líneas.

```bash 
kubectl logs pingpong --tail=20
```

Ver los últimas 20 líneas del log, lo hace en forma continua.

```bash 
kubectl logs pingpong --tail=20 -f
```

Todos los pods que esten corriendo.

```bash 
kubectl logs -l run=pingpong
```

Obtener la descripción de un pod en particular. 

```bash 
kubectl describe pod pingpong
```

Ver todos los pods y capas.

```bash 
kubectl get all
```

Aumentar replicas a 8.

```bash
kubectl scale deploy/pingpong --replicas 8
```

Ver la ip y status interno. 

```bash
kubectl get pods -o wide
```

Ver en línea el estado y los cambios de los pods.

```bash 
kubectl get pods -w
```

Ver la descripción de pod específico.

```bash
kubectl describe pods pingpong-98f6d5899-4tkfs
```

Ver la especificación en yaml.

```bash 
kubectl get pod -o yaml pingpong-98f6d5899-4tkfs
```

Expone el yaml y todos los recursos para la creación del cluster. 

```bash
kubectl run --dry-run -o yaml pingpong --image alpine ping 1.1.1.1
```

## Balanceador.

Create un deployment desde la imagen **jpetazzo/httpenc**.
```bash
kubectl create deployment httpenv --image jpetazzo/httpenc
```

Revisar el pod (error en la imagen).
```bash 
kubectl describe pod httpenv-748fd4bd77-l9h7t
```

Eliminar el deployment.

```bash 
kubectl delete deploy/httpenv
```

Creamos replicas 10 del pod.
```bash 
kubectl scale deployment httpenv --replicas=10
```

Exponer el deployment en el puerto 8888, crea un servicio con **ip interna** y **puerto interno**.
```bash
kubectl expose deployment httpenv --port=8888 --target-port=8888
```

Ver los servicios.
```bash
kubectl get svc
```

Recurso para conocer los endpoint.
```bash 
kubectl describe endpoints httpenv 
```

Obtener la información de los endpoint en formato yaml.
```bash
kubectl get endpoints httpenv -o yaml
```

## Deployar nuestra app.

Deployar nuestra app dockercoins.
```bash
kubectl create deployment redis --image=redis
kubectl create deployment hasher --image=dockercoins/hasher:v0.1
kubectl create deployment rng --image=dockercoins/rng:v0.1
kubectl create deployment webui --image=dockercoins/webui:v0.1
kubectl create deployment worker --image=dockercoins/worker:v0.1
```

Exponer los servicios. 
```bash
kubectl expose deployment redis --port 6379
kubectl expose deployment rng --port 80
kubectl expose deployment hasher --port 80
```

Revisamos el log del worker. 
```bash 
kubectl logs deploy/worker
```

Exponer puerto 80 del deploy webui (se puede usar --target-port=8888).
```bash
kubectl expose deploy/webui --type=NodePort --port=80
```

## Accediendo a servicio internos y externos.

Para levantar el cliente en modo local. 
```bash
kubectl proxy
```

Para exponer un pod en un puerto especifico. 
```bash
kubectl port-forward svc/redis 10000:6379
```

## Dashboard.

Instalar el dashboard de kubernetes.
```bash 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml

```

Editar la configuración del service (cambiar de ClusterIP a NodePort).
```bash 
kubectl edit service kubernetes-dashboard -n kubernetes-dashboard
```

Para obtener permisos, seguir la siguiente [guía para crear el token](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md). 

## Balanceo y Daemon sets.

Revisar el estado de los deployments en vivo.
```bash
kubectl get deployment -w
```

Exportar una definición de deployment como yaml.
```bash 
kubectl get deploy/rng -o yaml > rng.yml
```

Aplica cambios omitiendo la validación.
```bash
kubectl apply -f .\rng.yml --validate=false
```

Buscar con selectores, por ejemplo con el label app=rng
```bash
kubectl get pods --selector=app=rng
```

Eliminar **label app** de un pod.
```bash
kubectl label pod rng-5d8b6c4cff-zgf7q app-
```

## Despliegues controlados.

Output en JSON de los deploys.
```bash
kubectl get deploy -o json
```

Obtener del deploy los nombres, con los valores de maxUnavailable y maxSurge.
```bash
kubectl get deploy -o json | jq ".items[] | {name:.metadata.name} + .spec.strategy.rollingUpdate"
```

Muestra el número de réplicas indicando cuántos Pods debería gestionar.
```bash
kubectl get replicasets -w
```

Cambia la imagen de worker original a la versión v0.2.
```bash
kubectl set image deploy worker worker=dockercoins/worker:v0.2
```

Rollback de un deploy.
```bsh
kubectl rollout undo deploy worker
```

Cambiar estrategia de deployment, cambiar 1 pod y no dejar indisponible el ambiente. Es decir maxSurge: 1 y maxUnavailable: 0 
```bash
kubectl edit deploy worker
```

## Healthchecks.

Editar el deploy de redis.
```bash
kubectl edit deploy/redis
```

Agregamos el livenessProbe. 
```yaml
livenessProbe:
  exec: 
    command: ["redis-cli", "ping"]
```

Revisamos la descripción del pod.
```bash
kubectl describe pod redis-57dd56d6b9-bkxxp
```

Ingresa a un pod específico.
```bash
kubectl exec redis-57dd56d6b9-q2lc7 -it bash
```

## Gestionar stack con helm.

Entregar permisos a helm.
```bash
kubectl create clusterrolebinding add-on-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:default
```

Buscar un paquete. 
```bash
helm search repo prometheus
```

Agregar el repositorio.
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

Instalar prometheus.
```bash
helm install --generate-name stable/prometheus --set server.service.type=NodePort --set
```

Obtener todos los recursos en formato yaml. 
```bash
while read kind name; do kubectl get -o yaml $kind $name > dockercoins/templates/$name-$kind.yaml
done <<EOF
deployment worker
deployment hasher
daemonset rng
deployment webui
deployment redis
service hasher
service rng
service webui
service redis
EOF
```

## Gestionar la configuración

Crear una configuración por medio del configmap, crear el configmap por medio de un archivo **cfg**.
```bash
kubectl create configmap haproxy --from-file=haproxy.cfg
```

Deployar un pod con el configmap, con los volumenes.
```bash
kubectl apply -f .\haproxy.yaml
```

Crear un configmap, desde la línea de comandos. 
```bash
kubectl create configmap haproxy --from-literal=http.addr=0.0.0.0:80
```

Crear un pod con el registry, idem a haproxy. 
```bash
kubectl apply -f registry.yaml
```

## Namespace

Ver los namespace en kubernetes.
```bash
kubectl get namespace
```

Ver configuración de usuario y contexto.
```bash
kubectl config get-contexts
```

Crear **namespace**.
```bash
kubectl create namespace blue
```

Cambiar de **namespace**.
```bash
kubectl config set-context --current --namespace=blue
```