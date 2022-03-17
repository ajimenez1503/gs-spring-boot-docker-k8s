# Getting started with Spring boot docker and kubernetes
Ref:
- https://spring.io/guides/gs/spring-boot-kubernetes/
- https://github.com/kubernetes-sigs/kind

#### Create and run a Spring Boot Application
```shell
curl https://start.spring.io/starter.tgz -d dependencies=webflux,actuator | tar -xzvf -
./mvnw install
java -jar target/*.jar
curl localhost:8080/actuator | json_pp
```

### Containerize the Application
- Create `Dockerfile`
```shell
FROM adoptopenjdk/openjdk11:jdk-11.0.2.9-slim
WORKDIR /opt
ENV PORT 8080
EXPOSE 8080
COPY target/*.jar /opt/app.jar
ENTRYPOINT exec java $JAVA_OPTS -jar app.jar
```
```shell
docker build -t demo-java .
docker run -p 8080:8080 demo-java
curl localhost:8080/actuator | json_pp
```

### Push the conteiner
- Create the repository https://hub.docker.com/repository/docker/ajimenez15/demo-java/general
```shell
docker tag demo-java ajimenez15/demo-java
docker push ajimenez15/demo-java
```

### Create cluster in Kubernetes
```shell
curl -Lo ./kind "https://kind.sigs.k8s.io/dl/v0.11.1/kind-$(uname)-amd64"
chmod +x ./kind
./kind create cluster
```
```shell
kubectl cluster-info
kubectl get all
```

### Deploy the Application to Kubernetes
```shell
kubectl create deployment demo --image=demo-java  --dry-run=client -o=yaml > deployment.yaml
kubectl apply -f deployment.yaml
kubectl get all
kubectl get pods
kubectl port-forward pods/demo-868dfcb84f-cvjnt 8080:8080
curl localhost:8080/actuator | json_pp
```

### Destroy the cluster
```shell
./kind delete cluster
kubectl cluster-info
```

# Deploy in Google
https://www.youtube.com/watch?v=SzbeDqBSRkc
- Log in https://cloud.google.com/
- Create a cluster with the name `k8s-docker-demo-java`
- Create the file `deployment_google.yaml`
- Connect to the cluster using the Google cloud shell:
```shell
gcloud container clusters get-credentials k8s-docker-demo-java --zone europe-west1-b --project analog-hull-344411
```
- upload the file `deployment_google.yaml`
- Deploy the container in the cluster.
```shell
kubectl apply -f deployment_google.yaml
```
- In the workloads tab, select `demo-java-google-deployment`
- Click in expose
- Set `8080` for the port-target
- In the external endpoint you will find the IP in my case `http://34.140.165.145/`
- check the microservice is working fine:
```shell
curl http://34.140.165.145/actuator | json_pp
```

- In the cluster tab, select delete this cluster.