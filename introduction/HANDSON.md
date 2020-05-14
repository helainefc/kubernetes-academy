# Connecting to a GKE Cluster

```bash
$ gcloud config set compute/region us-central1
$ gcloud container clusters get-credentials gke-academy-1 --region=us-central1
$ gcloud container clusters list --region=us-central1
```

# Inspecting the Cluster

```bash
$ kubectl get componentstatuses
$ kubectl get nodes
```

# Pods

```bash
# Create a single pod
$ kubectl run --generator=run-pod/v1 nginx-pod --image=nginx:latest

# Delete the pod
$ kubectl delete pod nginx-pod
```

```yaml
# Pod manifest file nginx-pod.yaml
# If using Vim type ":set paste" before pasting this
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
```

```bash
# Create a pod using the manifest file
$ kubectl create -f nginx-pod.yaml
```

```bash
# List pods
$ kubectl get pods

# List pods with detailed information
$ kubectl get pods -o wide

# Describe the Pod
$ kubectl describe pod nginx-pod

# Delete the pod
$ kubectl delete pod nginx-pod
```

# ReplicaSets

```yaml
# ReplicaSet manifest file nginx-rs.yaml
# If using Vim type ":set paste" before pasting this
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    app: nginx
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: app
        image: nginx:1.7
```

```bash
# Create a new ReplicaSet
$ kubectl apply -f nginx-rs.yaml

# List the ReplicaSets
$ kubectl get replicasets

# List the pods from the replica set
$ kubectl get pods -l tier=frontend

# Scale the number of pods
$ kubectl scale rs/nginx --replicas=5

# List the pods from the replica set
$ kubectl get pods -l tier=frontend

# Update the ReplicaSet Image
$ kubectl set image rs/nginx app=nginx:1.7

# Describe the replica set
# And notice how the container image has been updated
$ kubectl describe rs nginx

# List the pods from the replica set
# Notice how the pods weren't restarted
$ kubectl get pods -l tier=frontend

# Describe one of the pods
# Notice how it is using the old image
$ kubectl describe pod nginx-<hash>

# Delete the pod
# A new pod will be created
$ kubectl delete pod nginx-<hash>

# Describe the new pod
# Notice how it is using the new image
$ kubectl describe pod nginx-<hash>

# The old pods will keep using the old image

# Delete the replicaset
$ kubectl delete rs nginx
```

# Deployments

```yaml
# Deployment manifest file nginx-deployment.yaml
# If using Vim type ":set paste" before pasting this
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.7
        name: app
```

```bash
# Create a Deployment
$ kubectl create -f nginx-deployment.yaml --record

# List the deployments
$ kubectl get deployments

# See the pods deployed
$ kubectl get pods -l app=nginx

# List the replica sets
$ kubectl get replicasets

# Scale up the deployment
$ kubectl scale deployment nginx --replicas=5

# See the pods deployed
$ kubectl get pods -l app=nginx

# List the replica sets
$ kubectl get replicasets

# Rollout a new version
$ kubectl set image deployment nginx app=nginx:latest --record

# Inspect the pods
$ kubectl get pods -l app=nginx

# List the replicasets
$ kubectl get replicasets

# Restore the old version
$ kubectl rollout undo deployment nginx --record

# List the replicasets
$ kubectl get replicasets

# See the deployment history
$ kubectl rollout history deployment nginx
```

# Services

```bash
# List services
$ kubectl get services

# Expose the deployment to the internet
$ kubectl expose deployment nginx --port=80 --target-port=80 --type=LoadBalancer

# The above command will create a service
# Wait for its EXTERNAL-IP to be published
$ kubectl get services

# Once ready you can visit http://<EXTERNAL-IP>
# Inspect the Headers

# Delete the service
$ kubectl delete service nginx
```

```yaml
# Service manifest file nginx-service.yaml
# If using Vim type ":set paste" before pasting this
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
```