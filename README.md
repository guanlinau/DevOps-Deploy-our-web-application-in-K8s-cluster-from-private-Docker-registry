### Deploy our web application in K8s cluster from private Docker registry

### Technologies used:

Kubernetes, AWS EKS, AWS Fargate

### Project Description:

1-Create Fargate IAM Role

2-Create Fargate Profile

3-Deploy an example application to EKS cluster using Fargate profile

### Usage Instructions:

###### Step 1: Enter inside Minikube via ssh

```
minikube ssh
```

###### Step 3: Get aws ecr login password inside Minikube

```
aws ecr get-login-password
```

###### Step 4: Docker login ECR inside minikube

```
docker login -u AWS -p <password> 118381122830.dkr.ecr.ap-southeast-2.amazonaws.com/node-app
```

###### Step 5: Get the credentials stored inside .docker/config.json

```
cat .docker/config.json
```

###### Step 6: Copy the credentials from minikube to local laptop terminal

```
scp -i $(minikube ssh-key) docker@$(minikube ip):.docker/config.json .docker/config.json
```

###### Step 7: Encrypt the credentials as base64

```
cat .docker/config.json | base64
```

###### Step 9: Create Secret deployment

```
apiVersion: v1
kind: Secret
metadata:
  name: my-registry-key
data:
  .dockerconfigjson: base64-encoded-contents-of-.docker/config.json-file
type: kubernetes.io/dockerconfigjson

```

###### Step 10: Create node deployment from aws ecr image by k8s

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      imagePullSecrets:
      - name: my-registry-key
      containers:
      - name: my-app
        image: 118381122830.dkr.ecr.ap-southeast-2.amazonaws.com/node-app:2.0
        imagePullPolicy: Always
        ports:
          - containerPort: 3000
```
