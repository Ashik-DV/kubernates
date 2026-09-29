pods are the wrapper which is responsble to manages the containers
its make easy of devops engineers life

pods are defined by pod.yml
example:


apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80



commands :

Command	                    Purpose

kubectl get nodes	        Get/list Kubernetes nodes
kubectl get pods	        Get/list Pods
kubectl get pods -w	        Watch Pod status changes
kubectl get pods -o wide	Get detailed Pod information
kubectl apply -f pod.yml	Create/update resources from YAML
kubectl create -f pod.yml	Create resources from YAML
kubectl delete pod nginx	Delete a specific Pod
kubectl discribe pod nginx  It shows more information and status of the pod
kubectl logs nginx          It shows the Logs 