# liberando-productos-Carlos-Julio

## Requisitos
- Tener instalado Docker
- Tener instalado Kubectl
- Tener instalado Minikube 
- Tener instalado Helm
- Tener instalado argocd CLI

## Repositorios
Repo de GIT: https://github.com/cjcuesta/liberando-productos-Carlos-Julio.git

Repo de CircleCI: https://app.circleci.com/pipelines/github/cjcuesta/liberando-productos-Carlos-Julio



## Instalación del escenario
Descargar el repositorio de Git. 
```
git clone https://github.com/cjcuesta/liberando-productos-Carlos-Julio.git 
```
Se ingresa a la carpeta del repositorio 
```
cd https://github.com/cjcuesta/liberando-productos-Carlos-Julio 
```
**lo anterior es necesario para encontrar los archivos necesario en las siguientes instalaciones**

### Creación del cluster en Minikube
Se crea el perfil. 
```
minikube start --kubernetes-version='v1.28.3' \
    --cpus=6 \
    --memory=5120 \
    --addons="metrics-server,default-storageclass,storage-provisioner,ingress,ingress-dns" \
    -p liberando-productos-cj
```
Para ver los perfiles creados
```
minikube profile list 
```
Para borrar los perfiles o clusters que no se usan o que se desean borrar
```
minikube -p nombreperfil delete 
```
```
minikube -p liberando-productos-cj delete  
```

Para ver los contextos
``` 
kctx
```
### Instalación ArgoCD

Baja el repo de helm
```
helm repo add argo https://argoproj.github.io/argo-helm
```
Actualiza el repo de helm
```
helm repo update
```
Instala  ArgoCD
```
helm -n argocd upgrade --install argocd argo/argo-cd --create-namespace --wait --version 3.26.3
```
Se espera a que los PODs de ArgoCD esten desplegados
```
k -n argocd get pods -w
```

Se hace un se hace un port forward para que se  pueda ver el administrativo de ArgoCD
```
kubectl -n argocd port-forward service/argocd-server 8085:443
```

Se abre el administrativo de ArgoCD  [http://localhost:8085](http://localhost:8085)

Obtenemos el pasword del admin de ArgoCD
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d 
> kIxfe07MZ6o585Jy
```
Si se tiene ArgoCD CLI
```
argocd login localhost:8085 --insecure
 Username: admin
 Password: 
```

### Instalación Prometheus
Añadir el repositorio de helm prometheus-community para poder desplegar el chart kube-prometheus-stack:
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
```
helm repo update
```
Desplegar el chart kube-prometheus-stack del repositorio de helm añadido en el paso anterior con los valores 
configurados en el archivo kube-prometheus-stack/values.yaml en el namespace monitoring:

```
helm -n monitoring upgrade \
    --install prometheus \
    prometheus-community/kube-prometheus-stack \
    -f kube-prometheus-stack/values.yaml \
    --create-namespace \
    --wait --version 55.4.0
```	
Realizar split de la terminal o crear una nueva pestaña y ver como se están creando pod en el namespace monitoring utilizado para desplegar el stack de prometheus:

```
kubectl -n monitoring get po -w
```

### Se crea la aplicacion en ArgoCD

```
k apply -f definition_app.yaml 
```

# para sincronizar desde la linea de comandos 
argocd app sync argocd/fast-api

# para si la aplicacion fue creada
argocd app list

# para borrar la aplicacion 
argocd app delete argocd/fast-api

Hacer split de la terminal o crear una nueva pestaña en la misma y observar como se crean los pods en el namespace fast-api donde se ha desplegado el web server:

kubectl -n fast-api get po -w


Obtener los logs del contenedor fast-api-webapp del deployment my-app-fast-api-webapp en el namespace fast-api, observar como está arrancando el servidor fast-api:

kubectl -n fast-api logs -f deployment/my-app-fast-api-webapp -c fast-api-webapp

Debería obtenerse un resultado similar al siguiente:
[2022-11-09 11:28:12 +0000] [1] [INFO] Running on http://0.0.0.0:8081 (CTRL + C to quit)


Abrir una nueva pestaña en la terminal y realizar un port-forward al Service creado para nuestro servidor:

kubectl -n fast-api port-forward svc/fast-api-fast-api-webapp 8081:8081

Abrir una nueva pestaña en la terminal y realizar un port-forward al Service creado para que nuestro servidor envie las metricas

kubectl -n fast-api port-forward svc/fast-api-fast-api-webapp 8000:8000

Empezar a realizar diferentes peticiones al servidor de fastapi, es posible ver los endpoints disponibles y realizar peticiones 
a los mismos a través de la URL http://localhost:8081/docs utilizando swagger

Abrir una nueva pestaña en la terminal y realizar un port-forward del puerto http-web del servicio de Grafana al puerto 3000 de la máquina:

kubectl -n monitoring port-forward svc/prometheus-grafana 3000:http-web

Abrir otra pestaña en la terminal y realizar un port-forward del servicio de Prometheus al puerto 9090 de la máquina:

kubectl -n monitoring port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090

Acceder a la dirección http://localhost:3000 en el navegador para acceder a Grafana, las credenciales por defecto son admin para el usuario 
y prom-operator para la contraseña.

Acceder a la dirección http://localhost:9090 para acceder al Prometheus, por defecto no se necesita autenticación.

En grafana se carga el archivo que se encuentra en la carpeta custom_dashboard.json 

Si se quiere disparar la integracion se hace un cambio en el codigo del repo y luego 

circleci config validate .circleci/config.yml && \
git pull && \
git add . && \
git commit -m "Prueba tags 2" && \
git push && \
git tag v0.0.2 && \
git push --tag



