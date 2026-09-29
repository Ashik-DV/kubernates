Deployment and Service are separate resources. Deployment manages Pods; Service provides stable network access to those Pods.


loadbalancing,
discovery (label and selector) and
expose the ip to extenal world

label and selectors are used to label the pods once it gone down it should have the same the lable withouch depending on the changable ip 
load balancing it main load balnce by hnadling the request
it expose the ip to external word

#cluster ip :- we can access only within the kubernates env
#nodeport :- we can access within the orgainzation (like in our laptop and outside the kubernates env)
#loadbalancer:- we can access everywehere (its wrk in cloud provider ex aws ,gcp,azure)


this is Deployment mainfest file

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: =====> ngnix
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ======> nginx
  template:
    metadata:
      labels:
        app:  ======> nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80



this is the service mainfest file

apiVersion: v1
kind: Service

metadata:
  name: nginx-service

spec:
  selector:
    app: ===> nginx   //here and deploy.yml name should be same

  ports:
    - port: 80
      targetPort: 80

  type: NodePort / Loadbalancer/ ClusterIP  =====> this are based on users requirement





  minikube ssh   ==> login in local cluster