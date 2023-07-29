# Kubernetes Fundamentals
[Learning Kubernetes With Kim Schlesinger](https://www.linkedin.com/learning/learning-kubernetes-16086900) 
[Github](https://github.com/kimschles/linkedin-learning-kubernetes)

## [Kubernetes Basics](https://kubernetes.io/docs/home/)

#### Config
 YAML is the standard format used in a Kubernetes configuration file.
--- three dash separate document in yaml
extensions .yaml or .yml
yaml check -> https://yamlchecker.com/

#### Pod
A Pod is a Kubernetes abstraction that represents a group of one or more application containers (such as Docker)

#### Node
A Pod always runs on a Node. A Node is a worker machine in Kubernetes and may be either a virtual or a physical machine, depending on the cluster. Each Node is managed by the control plane. A Node can have multiple pods, and the Kubernetes control plane automatically handles scheduling the pods across the Nodes in the cluster. 


## Install minikube on macos

```sh
brew install minikube
minikube start
```

### Usefull Commands
```sh
kubectl cluster-info
kubectl get nodes
kubectl get namespace = kubectl get ns
kubectl get services -A
kubectl get nodes -A
kubectl describe pod <podName>
kubectl delete pod <podName>
```

### Creating namespace
#### namespace.yaml

```sh
---
apiVersion: v1
kind: Namespace
metadata:
 name: development
```

```sh
kubectl apply -f namespace.yaml
kubectl get namespaces
kubectl delete -f namespace.yaml
```

### Deployment
#### deployment.yaml

```sh
--- 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-info-deployment
  namespace: development
  labels:
    app: pod-info
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pod-info
  template:
    metadata:
      labels:
        app: pod-info
    spec:
      containers:
      - name: pod-info-container
        image: guliz91/demoworkerservice:latest
        ports:
        - containerPort: 3000
        env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
```

```sh
kubectl apply -f deployment.yaml
kubectl get deployments -n development
kubectl get pods -n development
```

### service.yaml
```sh
---
apiVersion: v1
kind: Service
metadata:
  name: pod-service
  namespace: staging
spec:
  ports:
    - port: 80
      targetPort: 3000
      protocol: TCP
  type: NodePort
  selector:
    app: pod-info
```

```sh
kubectl apply -f deployment.yaml
kubectl get deployments -n development
kubectl get pods -n development
kubectl describe pod <podName>
kubectl delete pod <podName>
```



## Exercise - 1

- Create development namespace
```sh
kubectl apply -f namespace.yaml
```

- Create deployment
```sh
kubectl apply -f deployment.yaml
```

- Get deployments in development namespace
```sh
kubectl get deployments -n development
```
```
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
pod-info-deployment   3/3     3            3           6m
```

- Get pods in development namespace
```sh
kubectl get pods -n development -o wide
```
```
NAME                                   READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
pod-info-deployment-757cb75bbb-4c8k8   1/1     Running   0          20s   10.244.0.11   minikube   <none>           <none>
pod-info-deployment-757cb75bbb-58nzv   1/1     Running   0          20s   10.244.0.12   minikube   <none>           <none>
pod-info-deployment-757cb75bbb-xpj8b   1/1     Running   0          20s   10.244.0.10   minikube   <none>           <none>
```

- Delete pod in development namespace
```sh
kubectl delete pod pod-info-deployment-757cb75bbb-4c8k8 -n development
```

- Get pods in development namespace
```sh
kubectl get pods -n development -o wide
```
```
NAME                                   READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
pod-info-deployment-757cb75bbb-58nzv   1/1     Running   0          8m45s   10.244.0.12   minikube   <none>           <none>
pod-info-deployment-757cb75bbb-l27nd   1/1     Running   0          6s      10.244.0.13   minikube   <none>           <none>
pod-info-deployment-757cb75bbb-xpj8b   1/1     Running   0          8m45s   10.244.0.10   minikube   <none>           <none>
```

- Check pods healthy in development namespace
```sh
kubectl describe pod pod-info-deployment-757cb75bbb-l27nd -n development
```


## Exercise - 2

- Create busybox pod
```sh
kubectl apply -f busybox.yaml
```

- Get pod name
```sh
kubectl get pods 
```

- Get inside container, access to the pod for executable shell
```sh
kubectl exec -it busybox-6b95744666-z25cg -- /bin/sh
/# wget 10.244.0.12:3000
/# cat index.html
```
```
{"pod_name":"pod-info-deployment-757cb75bbb-58nzv","pod_namespace":"development","pod_ip":"10.244.0.12"}/ #
```
```sh
/# exit
```

- Check log
```sh
kubectl get pods -n development
```
```
NAME                                   READY   STATUS    RESTARTS   AGE
pod-info-deployment-757cb75bbb-58nzv   1/1     Running   0          68m
pod-info-deployment-757cb75bbb-l27nd   1/1     Running   0          59m
pod-info-deployment-757cb75bbb-xpj8b   1/1     Running   0          68m
```

```sh
kubectl logs pod-info-deployment-757cb75bbb-58nzv -n development
```
```
undefined
Example app listening on port 3000
```

## Exercise - 3

- Requirement
  * deployment and app label names are 'quote-service'
  * use development namespace
  * container-name will be 'quote-container'
  * 2 replicas and image is 'datawire/quote:0.5.0'
  * container port 8080

### quote.yaml
```sh
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quote-service
  namespace: development
  labels:
    app: quote-service
spec:
  replicas: 2
  selector:
    matchLabels:
      app: quote-service
  template:
    metadata:
      labels:
        app: quote-service
    spec:
      containers:
      - name: quote-container
        image: datawire/quote:0.5.0
        ports:
        - containerPort: 8080
```

- Create quote
```sh
kubectl apply -f quote.yaml
```

- Check pods
```sh
kubectl get pods -n development
```