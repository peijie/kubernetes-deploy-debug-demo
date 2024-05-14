# Build Docker Image Using DockerFile
Assume docker has installed and an executable .jar file is available, the following steps below to build docker image using Dockerfile
1. Create the Dockerfile
2. Use docker build command to build the docker image
```
docker build -f src/main/docker/Dockerfile -t payroll .
```
3. List the docker images on local machine
```
docker image ls
```
4. Run the docker image on local machine to confirm it is working
```
docker run -dp 127.0.0.1:8080:8080 payroll
```
5. Test the application is working
```markdown
curl http://localhost:8080/employees
```
6. List running docker containers
```markdown
docker ps
```
7. Login docker container
```
docker exec -it containerID sh
example:
docker exec -it fec39b7e0860 sh
```
8. Stop the running container
```markdown
docker stop containerID
example:
docker stop fec39b7e0860
```


# Push Docker Image to Remote Registry
ITCP project will use Harbor for docker registry. It is deployed on Kubernetes. The URL is harbor.itcpdev.mrr.ste.com
1. Login to docker registry
```markdown
docker login harbor.itcpdev.mrr.ste.com
```
2. Tag docker version for harbor registry
```markdown
docker tag payroll:latest harbor.itcpdev.mrr.ste.com/demo/payroll:0.1
```
3. Push the docker image to harbor
```markdown
docker push harbor.itcpdev.mrr.ste.com/demo/payroll:0.1
```

# Build Docker Image using JIB
1. Add JIB plugin in pom file
2. Build docker imge using JIB
JIB will build docker image and push to docker image repository. The correct plugin configuration is required.
```markdown
 ./mvnw verify
```
Another way is to use docker build if docke is installed
```markdown
./mvnw compile jib:dockerBuild
```

3. Push to harbor if you use docker build
```markdown
docker push harbor.itcpdev.mrr.ste.com/itcp/payroll:0.3
```

# Deploy Application to Kubernetes
1. Create a Kubernetes secret to keep docker registry credentials
```markdown
kubectl create secret docker-registry <secret name> -n <namespace> --docker-username=<username> --docker-password=<password> --docker-email=<email> --docker-server=harbor.itcpdev.mrr.ste.com --dry-run=client -o yaml > my-reg-secret.yaml
```
2. Deploy the secret
```markdown
kubectl apply -f src/main/kubernetes/harbor-secret.yaml
```
3. Deploy the service
```markdown
kubectl apply -f src/main/kubernetes/payroll-deployment.yaml
```
4. Check the pod is running
```markdown
kubectl get pod -n demo
```
5. Login to the pod for verification
```markdown
kubectl exec -n <name space here> <pod-name>  -it -- /bin/sh
```
6. Create and deploy ingress
```markdown
kubectl apply -f src/main/kubernetes/payroll-ingress.yaml -n demo
```
7. Test service
```markdown
curl -v https://payroll.itcpdev.mrr.ste.com/employees
```
# Remote debug the service using port forward
1. Special settings in deployment files
```markdown
    spec:
      containers:
        - name: payroll
          image: harbor.itcpdev.mrr.ste.com/demo/payroll:0.2
          imagePullPolicy: Always
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
            - name: debug-port
              containerPort: 5005
              protocol: TCP
          env:
            - name: JAVA_TOOL_OPTIONS
              value: '-Xdebug -agentlib:jdwp=transport=dt_socket,address=0.0.0.0:5005,server=y,suspend=n'
          resources:
            limits:
              memory: 512Mi
              cpu: "1"
            requests:
              memory: 256Mi
              cpu: "0.2"
      imagePullSecrets:
        - name: harbor-secret

```
2. access service from local computer by using port-forward
```markdown
kubectl port-forward payroll-7654d8969f-2jzxv 5005:5005 -n demo
```
3. Attaching Remote Debugger with Intellij </br>
   To attach a debugger, go to the run section in the right-hand corner and add a “Remote JVM debug” run configuration.

# Remote debug the service running on Kubernetes using Telepresence

1. Download and install Telepresence following the link below
   https://www.getambassador.io/docs/telepresence-oss/latest/install

2. Install traffic manager by following the link below
   https://www.getambassador.io/docs/telepresence-oss/latest/install/manager

```markdown
telepresence helm install
```
To install the traffic manager on specific namespace using following command
```markdown
telepresence helm install traffic-manager --namespace demo datawire/telepresence
```

3. Configure kubeconfi file
```markdown
- cluster:
    server: https://xx.xxx.xx.xxx:6443
	extensions:
    - name: telepresence.io
      extension:
        manager:
          namespace: ambassador
    insecure-skip-tls-verify: true
  name: itcp-dev-aws
```

4. Connect to a namespace
```markdown
telepresence connect --namespace demo
```

5. List services
```markdown
telepresence list
```

6. Intercept your service
```markdown
telepresence intercept payroll --port 8080
```
7. Debug your code locally
   Start the application in local mode. When remote service call is triggered, the request will be route to the local service.




