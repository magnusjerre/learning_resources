# Kubernetes tips and tricks
This learning resource contains my notes and experiences with playing around with kubernetes on Windows using the bundled Kubernetes stuff from `Docker Desktop on Windows`.

# Using the terminal to create deployments and services
## Creating a deployemnt
In order to run a docker image in Kubernetes as a deployment we first need to create the deployment:

```bash
> kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1


> kubectl get deployements
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp      1/1     1            1           4h46m
```

The deployment automatically creates a pod which contains the running application. We can get information on the running pod as well:
```bash
> kubectl get pods
NAME                                      READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-69fbc6f4cf-5gbxm      1/1     Running   0          4h47m


> kubectl describe pod/kubernetes-bootcamp-69fbc6f4cf-5gbxm
Name:         kubernetes-bootcamp-69fbc6f4cf-5gbxm
Namespace:    default
Priority:     0
Node:         docker-desktop/192.168.65.3
Start Time:   Fri, 12 Jun 2020 17:01:23 +0200
Labels:       app=kubernetes-bootcamp
              pod-template-hash=69fbc6f4cf
Annotations:  <none>
Status:       Running
IP:           10.1.0.14
IPs:
  IP:           10.1.0.14
Controlled By:  ReplicaSet/kubernetes-bootcamp-69fbc6f4cf
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://2a420f7fd1e4c1b306396ad1b88b1fa31819a3ef52542fc56d9498ff015cb831
    Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
    Image ID:       docker-pullable://gcr.io/google-samples/kubernetes-bootcamp@sha256:0d6b8ee63bb57c5f5b6156f446b3bc3b3c143d233037f3a2f00e279c8fcc64af
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 12 Jun 2020 17:01:32 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-x2vbf (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-x2vbf:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-x2vbf
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>
```
## Exposing the pod to the outside world
By default the pod is not accessible to anyone outside the Kubernetes cluster. In order to make it accessible we create a service that exposes an endpoint for the pod(s). A random port is exposed (i.e 31689)

```bash
> kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port=8080
service/kubernetes-bootcamp exposed


> kubectl get service
NAME                  TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
kubernetes            ClusterIP   10.96.0.1     <none>        443/TCP          7d2h
kubernetes-bootcamp   NodePort    10.99.32.16   <none>        8080:31689/TCP   46s


> curl localhost:31689
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    84    0    84    0     0  10500      0 --:--:-- --:--:-- --:--:-- 10500Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-69fbc6f4cf-5gbxm | v=1

```

## Formatting the __get__ output
There are several was of formatting the output of get `get` expression. The following examples shows how to custom-format a table, and how to return a single field of the service-spec-file using `jsonpath` which could be useful for scripting.

```bash
> kubectl get services -o custom-columns=NAME:metadata.name,EXPOSEPORT:spec.ports[*].nodePort
NAME                  EXPOSEPORT
kubernetes            <none>
kubernetes-bootcamp   31689


> kubectl get services/kubernetes-bootcamp -o jsonpath="{.spec.ports[*].nodePort}"
31689
```
# Using the terminal to delete resources
In order to delete resources we must be sure that we delete the correct resources, this can be done by referencing the labels relevant for the given resource(s). Use `kubectl describe` to find the labels for each resource to delete.
```bash
> kubectl describe services
# Ignoring the standard kubernetes deployment in the output here
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   app=kubernetes-bootcamp # This is the label we're looking for
...

> kubectl describe pods
Name:                   kubernetes-bootcamp
Namespace:              default
CreationTimestamp:      Fri, 12 Jun 2020 17:01:23 +0200
Labels:                 app=kubernetes-bootcamp # This is the label we're looking for
...
```
When we know the labels to delete we can do the following:
```bash
> kubectl delete services -l app=kubernetes-bootcamp
service "kubernetes-bootcamp" deleted
> kubectl delete deployments -l app=kubernetes-bootcamp-script
deployment.apps "kubernetes-bootcamp-script" deleted
```
There are several other ways of deleting resources, just use `kubectl delete --help` for more tips.

# YAML-fileconfig
In order to `POST` to the kubernetes API we utilize yaml-files (or json) to spec our object configuration file, see the [docs](https://kubernetes.io/docs/reference/kubernetes-api/) for specifics, Deployment spec example [here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#deployment-v1-apps) (wait for it to finish loading).
Note that the we have `apiVersion: apps/v1` for __Deployement__, for __Pod__ and __Service__ we use only `apiVersion: v1`.
## Single configuration in per YAML file
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-bootcamp-script
  labels:
    app: kubernetes-bootcamp-script
spec:
  selector:
    matchLabels:
      app: kubernetes-bootcamp-script
  replicas: 1
  template:
    metadata:
      labels:
        app: kubernetes-bootcamp-script
    spec:
      containers:
        - name: master
          image: gcr.io/google-samples/kubernetes-bootcamp:v1
          ports:
          - containerPort: 8080
```

We can deploy our object configuration file (deployment spec) using either `apply` or `create`, `apply` being ["Imperative Management"](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-config/) and `create` being ["Declarative management"](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/). In short, using `apply` tells Kubernetes what we want the end "state" to be without caring about how Kubernetes achieves it (i.e creating new deployemnts or updating existing ones), while using `create` we tell Kubernetes how to do something.  
```sh
> kubectl apply -f deployment-spec.yml
> kubectl create -f deployment-spec.yml
```

## Multiple configurations in a single YAML-file
We can use multiple configuration files if we want, or we can use one configuration file containing all the configuration. When using only one configuration file the `services` should be configured before the corresponding `deployments` (dont't remeber why at the moment), I guess this also applies for mulitple configuration files?
__NOTE TO SELF! The comments can't be included in the actual YAML-file.__
```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-bootcamp-service
  labels:
    app: kubernetes-bootcamp
spec:
  ports:
  - port: 8080
  type: NodePort
  selector:
    app: kubernetes-bootcamp-script # must match the label(s) for the deployemnt it should expose
--- # the three dashes indicate that a new configuration is coming
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: kubernetes-bootcamp-script
  name: kubernetes-bootcamp-script
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kubernetes-bootcamp-script
  template:
    metadata:
      labels:
        app: kubernetes-bootcamp-script
    spec:
      containers:
      - image: gcr.io/google-samples/kubernetes-bootcamp:v1
        name: master
        ports:
        - containerPort: 8080
```

Add the same guide as here: https://levelup.gitconnected.com/kubernetes-merge-multiple-yaml-into-one-e8844479a73a