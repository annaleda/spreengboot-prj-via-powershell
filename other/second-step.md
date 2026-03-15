# 📝 Wiki Completa -- Second Step: Deploy su Kubernetes (Kind)

## Indice

1.  Introduzione
2.  Prerequisiti Kubernetes
3.  Creazione cluster Kubernetes con Kind
4.  Build delle immagini Docker
5.  Caricamento immagini nel cluster Kind
6.  Creazione Namespace
7.  Deployment MySQL
8.  Deployment Backend
9.  Deployment Frontend
10. Creazione dei Service
11. Verifica dei Pod
12. Debug della comunicazione tra Pod
13. Gestione Spring Security
14. Test finale frontend → backend

------------------------------------------------------------------------

## Introduzione

In questo step estendiamo il progetto creato nel **primo-step.md** per
eseguire l'applicazione su **Kubernetes usando Kind**.

Obiettivi:

-   Deploy di MySQL
-   Deploy di Backend Spring Boot
-   Deploy di Frontend
-   Comunicazione interna tra servizi
-   Debug networking
-   Test del flusso frontend → backend

------------------------------------------------------------------------

## Prerequisiti Kubernetes

Installare:

-   Docker
-   kubectl
-   Kind

Installazione Kind (Windows con Chocolatey):

``` bash
choco install kind
```

Verifica installazione:

``` bash
kind version
kubectl version --client
```

------------------------------------------------------------------------

## Creazione cluster Kubernetes con Kind

Creare il cluster:

``` bash
kind create cluster
```

Verificare il nodo:

``` bash
kubectl get nodes
```

Output atteso:

    NAME                 STATUS   ROLES           AGE
    kind-control-plane   Ready    control-plane   1m

------------------------------------------------------------------------

## Build delle immagini Docker

Costruire backend:

``` powershell
docker build -t backend-service-1:latest C:\Users\annaleda\Desktop\ckad\project\backend\backend-service-1
```

Costruire frontend:

``` powershell
docker build -t front-end-app:latest C:\Users\annaleda\Desktop\ckad\project\frontend\front-end-app
```

Verifica immagini:

``` powershell
docker images
```

------------------------------------------------------------------------

## Caricamento immagini nel cluster Kind

Kind non usa direttamente le immagini Docker locali.

Caricare le immagini nel cluster:

``` powershell
kind load docker-image backend-service-1:latest
kind load docker-image front-end-app:latest
```

Se non viene fatto questo passaggio si ottiene:

    ErrImageNeverPull
    ImagePullBackOff

------------------------------------------------------------------------

## Creazione Namespace

Creiamo un namespace dedicato:

``` powershell
kubectl create namespace backend-app
```

Verifica:

``` powershell
kubectl get namespaces
```

------------------------------------------------------------------------

## Deployment MySQL

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: backend-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: secret
        - name: MYSQL_DATABASE
          value: appdb
        ports:
        - containerPort: 3306
```

Service MySQL:

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  namespace: backend-app
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
```

------------------------------------------------------------------------

## Deployment Backend

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-service
  namespace: backend-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend-service
  template:
    metadata:
      labels:
        app: backend-service
    spec:
      containers:
      - name: backend-service
        image: backend-service-1:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_DATASOURCE_URL
          value: jdbc:mysql://mysql-service:3306/appdb
        - name: SPRING_DATASOURCE_USERNAME
          value: root
        - name: SPRING_DATASOURCE_PASSWORD
          value: secret
```

------------------------------------------------------------------------

## Deployment Frontend

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-service
  namespace: backend-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend-service
  template:
    metadata:
      labels:
        app: frontend-service
    spec:
      containers:
      - name: frontend-service
        image: front-end-app:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 80
        env:
        - name: REACT_APP_BACKEND_URL
          value: http://backend-service:8080
```

------------------------------------------------------------------------

## Creazione Service

Service backend:

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: backend-app
spec:
  type: NodePort
  selector:
    app: backend-service
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30080
```

Service frontend:

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: backend-app
spec:
  type: NodePort
  selector:
    app: frontend-service
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30001
```

------------------------------------------------------------------------

## Verifica Pod

``` powershell
kubectl get pods -n backend-app
```

Output atteso:

    backend-service   Running
    frontend-service  Running
    mysql             Running

------------------------------------------------------------------------

## Debug comunicazione tra Pod

Entrare nel pod frontend:

``` powershell
kubectl exec -it <frontend-pod> -n backend-app -- sh
```

Test backend:

``` bash
curl http://backend-service.backend-app.svc.cluster.local:8080
```

Se compare:

    HTTP 401 Unauthorized

significa che:

-   il networking Kubernetes funziona
-   il backend è raggiungibile

------------------------------------------------------------------------

## Gestione Spring Security

Per test veloci possiamo disabilitare temporaneamente la sicurezza.

Creare:

    src/main/java/.../config/SecurityConfig.java

``` java
@Configuration
public class SecurityConfig {

@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {

http
.csrf(csrf -> csrf.disable())
.authorizeHttpRequests(auth -> auth.anyRequest().permitAll());

return http.build();

}
}
```

Ricostruire l'immagine:

``` powershell
mvn clean package
docker build -t backend-service-1:latest .
kind load docker-image backend-service-1:latest
kubectl rollout restart deployment backend-service -n backend-app
```

------------------------------------------------------------------------

## Test finale

Test dal pod frontend:

``` bash
curl http://backend-service:8080
```

Oppure dal tuo PC:

    http://localhost:30080

Frontend:

    http://localhost:30001

------------------------------------------------------------------------

## Risultato finale

Architettura funzionante:

    Frontend (NodePort 30001)
            │
            ▼
    Backend Spring Boot
            │
            ▼
    MySQL
