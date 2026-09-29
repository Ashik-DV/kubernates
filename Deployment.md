Deployment is kubernates controler for the pods ,here pods are just the wrapper for the container it doest have the ability of autohealing and autoscalling so deployment is come into picute 

deployment
   |
replicatset
   |
  pods
    |
container

replicatset is kubernates cntrler which ensure that and manges the desired number of pods even if we deled the pads manualy whic is agian maintain same number of pods


apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80


commands :

kubectl appy -f deployment.yml   // it create deployment -> replicaset => pods=> container all those things
