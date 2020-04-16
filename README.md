
### LAB-X: Building a Docker Image

![](./images/lab.png)

Spring Boot has new and compelling capabilities for Kubernetes including 
a new Spring Boot Maven Plugin goal for building Docker images without a dockerfile and the addition of
[Kubernetes Readiness and Liveness Probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
endpoints in Spring Boot Actuator.

#### Kubernetes Probes
- The Readiness state tells whether the application is ready to accept client requests. If the Readiness state is 
unready, Kubernetes should not route traffic to this instance. If an application is too busy processing a task queue, 
then it could declare itself as busy until its load is manageable again.

- The Liveness state of an application tells whether the internal state is valid. If Liveness is broken, this means 
that the application itself is in a failed state and cannot recover from it. In this case, the best course of action is 
to restart the application instance. For example, an application relying on a local cache should fail its Liveness 
state if the local cache is corrupted and cannot be repaired.

#### Clone the repo and explore the project
- Using your Ubuntu VM you are going to package Spring Boot app and run it locally and on Kubernetes.

- Execute the following commands:

```
# Get the source code from the repo
cd ~ 
git clone https://github.com/jimbasler-pivotal/springkube  
cd ~/springkube
```




#### Create and run a Docker container locally
```
./mvnw spring-boot:build-image
```

You just created a Docker container. View it in your local repository
```
docker images
```

Optionally run it locally
```
docker run -p 8080:8080 -t springkube:0.0.1-SNAPSHOT
```

Open another terminal and check to see if the application is ready to handle requests.
```
curl http://localhost:8080/actuator/health/readiness; echo
```

Check the REST service endpoint
```
curl http://localhost:8080/greeting; echo
```

#### Deploy to Kubernetes

A Docker image identical to the one you created has been tagged and uploaded into the Public Docker Hub as 
jbasler/springkube. Let's use the jbasler/springkube image to run the REST service app on your Kubernetes cluster.

Execute the following commands:
```
kubectl create deployment springkube --image=jbasler/springkube
kubectl get all
kubectl expose deployment springkube --type=LoadBalancer --port=80 --target-port=8080
```

Note the EXTERNAL-IP for the springkube service you just created.
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

Congratulations, you have completed LAB-X.