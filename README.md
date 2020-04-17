![](./images/vmware-logo.png)

This is an experimental lab for use with the 
[VMware TKG-i Workshop](https://github.com/rm511130/Tanzu-Workshop-TKG-i). It uses capabilities found in [Spring Boot 
2.3.0.M4](https://spring.io/blog/2020/04/03/spring-boot-2-3-0-m4-available-now).

### LAB-X1: Spring for Kubernetes

Spring Boot has new and compelling capabilities for Kubernetes including 
a new Spring Boot Maven Plugin goal for building Docker images without a dockerfile. It also includes the addition of
[Kubernetes Readiness and Liveness Probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
endpoints in Spring Boot Actuator.

#### Kubernetes Probes
- The readiness state tells whether the app is able to accept requests. If an app is not ready, Kubernetes should not 
route traffic to it.

- The liveness state tells if an app has failed. Restarting a container in such a state can help to make the 
application more available despite bugs.

#### Clone the repo and explore the Spring Boot project

![](./images/java-spring-tiny.png)

Using your Ubuntu VM you are going to package a Spring Boot app and run it locally and on Kubernetes.
```
cd ~ 
git clone https://github.com/jimbasler-pivotal/springkube  
```

#### Create and run a Docker container locally

![](./images/docker-tiny.png)

```
cd ~/springkube
./mvnw spring-boot:build-image
```

You just created a Docker image using Spring Boot Maven Plug-in's spring-boot:build-image goal! View the image in your 
local repository.
```
docker images
```

Run a container locally.
```
docker run -d -p 8080:8080 -t springkube:0.0.1-SNAPSHOT 
```

Check to see if the application is ready to handle requests.
```
curl http://localhost:8080/actuator/health/readiness; echo
```

Check the REST service endpoint.
```
curl http://localhost:8080/greeting; echo
```

#### Deploy to Kubernetes

![](./images/k8s.png)

A Docker image identical to the one you created has been tagged and uploaded into the Public Docker Hub as 
jbasler/springkube. Let's use the jbasler/springkube image to run the REST service app on your Kubernetes cluster.

Inspect the deployment that will be used. Notice the readiness and liveness probe definitions.
```
cat springkube-deployment.yaml
```

It should look like this:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springkube
spec:
  replicas: 1
  selector:
    matchLabels:
      app: springkube
  template:
    metadata:
      labels:
        app: springkube
    spec:
      containers:
        - image: jbasler/springkube
          ports:
            - containerPort: 8080
              protocol: TCP
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 20
          imagePullPolicy: Always
          name: springkube
```

Inspect the service that will be used.
```
cat springkube-service.yaml
```

It should look like this:
```
apiVersion: v1
kind: Service
metadata:
  name: springkube
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 8080
  selector:
    app: springkube
  type: LoadBalancer
```

Create the deployment and expose it through a service.
```
kubectl create -f springkube-deployment.yaml
kubectl create -f springkube-service.yaml
```

Run the command below and note the EXTERNAL-IP for the springkube service you just created. You may see that it is in a 
state of PENDING, so wait a moment and run the command again if necessary.
```
kubectl get service springkube
```

Check your REST service to see if it is working.
```
curl http://<EXTERNAL-IP found above>/greeting; echo
```
 
**Let's recap:** 

- You learned the functions of readiness and liveness probes in Kubernetes and how to expose actuator health endpoints 
in Spring Boot that probes can check.
- You built and executed a Docker image on your Ubuntu VM using Spring Boot Maven Plugin's build-image goal.
- You created a deployment on a Kubernetes cluster with the REST service app and configured the readiness and liveness 
probes.

Congratulations, you have completed LAB-X!

If you would like to dig a bit deeper into interesting Spring Kubernetes tools and technology, check out Ryan Baxter's 
[excellent workshop](https://hackmd.io/@ryanjbaxter/spring-on-k8s-workshop).