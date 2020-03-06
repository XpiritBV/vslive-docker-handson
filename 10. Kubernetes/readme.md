# Hands-on labs Introduction to Kubernetes

## Exercise 10: Hosting your containers in Kubernetes
In the previous exercise, you create the retro gaming app in a set of containers. We used Docker compose in the end to get all the containers running locally on your machine. Now we are going to move these containers into the Kubernetes cluster that is available for you to use.

### Validate you can access the cluster
In order to be able to work with the cluster, you need to get a command-line tool on your machine that interacts with the Kubernetes cluster.
You can get this tool downloaded to your machine by using the following azure command-line:

`az aks install-cli`

After the command is done, a folder is created with the name `.azure-kubectl` In this folder, there is the executable `kubectl.exe` which is the command-line tool. When you want to run this from any location on your machine you need to set the path in the environment of your system. In the taskbar, use the search option and search for `environment` this will show the option to open the control panel where you can set the environment variables on your system. Pick the `system variables` option and add to the path specification the folder that contains `kubectl.exe`

Now we have kubectl available, but we need to tool to authenticate against our cluster that is ready for you to use. For this go to the git repo and copy the folder with the name `.kube` and copy it to the local computer user directory `c:\user\student` You should now have a folder `c:\user\student\.kube` that contains the configuration to connect to the cluster.

To validate that all is working you can run the following command-line: `kubectl get nodes` and this should return something similar like the following:
```
NAME                                STATUS   ROLES   AGE   VERSION
aks-nodepool1-77988509-vmss000000   Ready    agent   10d   v1.15.7
aks-nodepool1-77988509-vmss000001   Ready    agent   10d   v1.15.7
aksnpwin000000                      Ready    agent   10d   v1.15.7
aksnpwin000001                      Ready    agent   10d   v1.15.7
```
When you see this result, you know you can connect to the available cluster. 

## Create your first deployment
Now we want to deploy our set of containers to the cluster. This means we need to take the following steps:

1) We create a namespace where we want all the containers to reside. You create a namespace that is unique amongst all students, so please use the number postfix that you also used for your virtual machine.
2) You create a deployment for your WebAPI container
3) You expose the WebAPI container(s) via a service and give it a name that can be found through DNS in the cluster
4) you create a deployment for your web frontend and pass it in the name of the service that exposes your API in the cluster
5) You create a service that exposes your web frontend outside the cluster via the load balancer

### Creating the namespace
We can create a namespace using the command line by typing the following command:
`kubectl create namespace student<your unique number>`

You can also create a yaml file, with the name `mynamespace.yaml` with the following contents:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: student<your unique number>
```
Next you issue the following commandline:
`kubectl apply -f mynamespace.yaml`

Both will have the same result. We prefer to use the file, since it makes things more reproducible

### Deploy the WebAPI
We already pre-cooked a container image for you and pushed this to the Docker hub. We can now define in a yaml specification we want to deploy our WebAPI service. for this create a yaml file with the name `webapi.yaml` and paste in the following contents:
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  namespace: student<your unique number>
  name: dep-leaderboardwebapi
spec:
  replicas: 1
  revisionHistoryLimit: 0
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: leaderboardwebapi
    spec:
      containers:
      - name: leaderboardwebapi
        terminationMessagePath: "/tmp/leaderboardwebapi-log"
        image: xpiritbv/leaderboard.webapi
        imagePullPolicy: Always
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 20
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 20
          periodSeconds: 10
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: Production
        - name: ASPNETCORE_URLS
          value: http://0.0.0.0:80
        - name: ApplicationInsights__InstrumentationKey
          value: 59cf101a-97fa-481d-b1aa-f13e6dc14767
        - name: KeyVaultName
          value: https://retrogamevault.vault.azure.net/
        - name: KeyVaultClientSecret
          value: 2Qfo[?L3_e?oKu3ZEzyfcdZ31-LMip3L
        - name: KeyVaultClientID
          value: dc16d1f8-655f-4e1b-ae6e-362504f4521b
        ports:
        - containerPort: 80
        - containerPort: 443
        resources:
          limits:
            cpu: "0.10"
      nodeSelector:
        beta.kubernetes.io/os: linux
```
Please don't forget to replace the namespace with the correct number of the namespace you created.

><b>note:</b> In the specification you see some environment variables being set. One of them is KeyValtName and the other KeyVaultSecret. these values help the WebAPI get the connection string to the database in a secure way, using azure keyvault. It goes beyond this workshop, but it is enough to know we pre-provisioned the keyvault in azure and with the data you provide here on the command-line you get the ability to securely retrieve the credentials form the keyvault. 

Now run the following command:
`kubectl apply -f webapi.yaml`
you should get back the following response:
```
deployment.extensions/dep-leaderboardwebapi created
```
You can now inspect how the deployment is doing by asking the cluster to show the pods it scheduled in your namespace. You can do this by using the following command:
`kubectl get pods --namespace student<your unique number>`
this should give you something similar back as shown here:
```
NAME                                     READY   STATUS    RESTARTS   AGE
dep-leaderboardwebapi-6f69f85774-pl5vv   1/1     Running   0          43s
```
Next, we want to expose this pod to the cluster via a service. This way we have a fixed name we can refer to in the cluster, while we might get new pods spinning up or down. 

To create a service we again create a yaml file, this time with the name webapi-service.yaml and you provide it with the following contents:
```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: student<your unique number>
  name: svc-leaderboardwebapi
  labels:
    version: dev
    product: RetroGaming
spec:
  selector:
    app: leaderboardwebapi
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
```
And we push this configuration to the cluser with the command-line:
`kubectl apply -f webapi-service.yaml`

Now we have a service that we can access in the cluster with the DNS name `svc-leaderboardwebapi` so if we want to call the API from another service in the cluster (your web app) then you can use the uri: http://svc-leaderboardwebapi

### Deploy the web application
Now that we have the WebAPI running we can now create the web application container. For this we are going to create a yaml file with the name webapplication.yaml that contains the following:

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  namespace: student<your unique number>
  name: dep-gamingwebapp
spec:
  replicas: 1
  revisionHistoryLimit: 0
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: gamingwebapp
    spec:
      containers:
      - name: gamingwebapp
        terminationMessagePath: "/tmp/gamingwebapp-log"
        image: xpiritbv/gamingwebapp
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: Production
        - name: ASPNETCORE_URLS
          value: http://0.0.0.0:80
        - name: LeaderboardApiOptions__BaseUrl
          value: http://svc-leaderboardwebapi
        - name: ApplicationInsights__InstrumentationKey
          value: 59cf101a-97fa-481d-b1aa-f13e6dc14767
        ports:
        - containerPort: 80
        - containerPort: 443
        resources:
          limits:
            cpu: "0.10"
      nodeSelector:
        beta.kubernetes.io/os: linux
```

><b>Note:</b> In the environment variables you see the value for `LeaderboardApiOptions_BaseUrl` and that points to the service we just created. This enables the web application to call into one of many pods that provide the API in the cluster depending on the number of replicas we specified. The service is a stable name, while pod names will vary all the time.

We also want to expose this web application to the outside world. If we want to access the application from outside the cluster we can define the service to be of type `LoadBalancer` which will then initiate the external Azure load balancer to get a public IP address instead of a cluster IP address, so we can access it from outside the cluster.

Create the following yaml file with the name webapp-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: student089
  name: svc-gamingwebapp
  labels:
    version: dev
    product: RetroGaming
spec:
  selector:
    app: gamingwebapp
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  type:
    LoadBalancer
```

Now send this to the cluster to create the service

To validate your service is created and you can access the web application, issue the following command:
`kubectl get services --namespace student<your unique number>`
this should show a similar result as the following:
```
C:\Users\student>kubectl get services --namespace student089
NAME                    TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE
svc-gamingwebapp        LoadBalancer   10.0.253.192   20.191.49.177   80:31551/TCP,443:31943/TCP   22s
svc-leaderboardwebapi   ClusterIP      10.0.108.174   <none>          80/TCP,443/TCP               15m
```
Here you can see both the internal service that exposes the WebAPI and the external service that has an external IP address assigned. 
If the results for the webapp service states status: pending, then it is still waiting for the external load balancer to reconfigure. retry the command until you see an external IP address that you can browse to validate the website is up and running. 

If all is running according to expectations, then we should see the following end result in the browser when you browse to the external IP address:

<img src="images\screenshot.PNG">

## Scaling your frontend
Now we have one instance of the web application running and one instance of our WebAPI. It is now rather easy to scale our deployment to have more containers.

To scale our web application to e.g. 5 instances, we can issue the following command:
`kubectl scale --replicas 5 deployments/dep-gamingwebapp --namespace student<your unique number>`

Now when you get the information from the cluster you will find you get still 1 service but now 5 pods servicing this one service.

Get the pods schedule:
`kubectl get pods --namespace student<your unique number>`
this should result in something similar like below:

```
NAME                                     READY   STATUS    RESTARTS   AGE
dep-gamingwebapp-5cdff9656f-fmrzc        1/1     Running   0          5m32s
dep-gamingwebapp-5cdff9656f-g5mlw        1/1     Running   0          5m32s
dep-gamingwebapp-5cdff9656f-lsd2s        0/1     Pending   0          5m32s
dep-gamingwebapp-5cdff9656f-rl6w5        1/1     Running   0          21m
dep-gamingwebapp-5cdff9656f-vq87d        1/1     Running   0          5m32s
dep-leaderboardwebapi-6f69f85774-pl5vv   1/1     Running   0          39m
```