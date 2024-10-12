# Project-2: 3 Tier App Deploy in k8s Ingress with HPA

![337433078-7380d2df-5554-427e-8fd4-d46418616496](https://github.com/user-attachments/assets/b92299e4-7d24-4a06-9e1e-376902fa67a7)



**GitHub Repo:** https://github.com/saifuranik/4-tier-Application-Deploy.git


## Step 1- Follow the repo link and clone 
 * Create Dockerfiles
    * Backend
    * Frondend
    * Nginx
 ### if your MongoDB connection string NOT WORKING might be expired 
  ####  Create new one from MongoDB atlas or you can use on promise machine by Downlod MongoDB

## Follow [This Link](https://saifuranik.medium.com/deploy-4-tier-react-node-mongodb-application-with-nginx-67add197577c) for Docker Compose setup & initial Setup of Project

# Step 00: set up K8s Cluster with Kubeadm
 * Follow [This Link](https://saifuranik.medium.com/create-k8-cluster-with-kubeadm-7b7f4cee47cd) for SetUp Cluster

## Step 1: set up K8s Cluster with Kubeadm
 * Link : [Kubeadm setup Guide](https://saifuranik.medium.com/create-k8-cluster-with-kubeadm-7b7f4cee47cd)

## Step 2: Push Frontend & Backend Docker image in to the DockerHub
 * for using ingress we have to push ***Docker file's image*** in to the ***Docker-hub***

### Follow the Steps

```sh 
docker login
```
```sh
docker push <dockerhub-username>/<Docker-image-name>:<tag>
```

# Step 3: Download Nginx Ingress Controller for LoadBalnce
* After Setup Kubernets Cluster ,download ingress controller for Load balancing.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```
you can check by

```bash
kubectl get svc
kubectl get all
kubectl get ing
```

### if you follow Step 00 whole project is already ready 85%

* Make a folder in project Named Ingress and we create YAML fiels there

***1. frontend.yaml***
   
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client-deployment
spec:
  selector:
    matchLabels:
      app: client
  template:
    metadata:
      labels:
        app: client
    spec:
      containers:
      - name: client
        image: saifuranik/goals-frontend  
        ports:
        - containerPort: 3000
        resources:                    
          requests:
            cpu: "100m"           
          limits:
            cpu: "200m"          

---

apiVersion: v1
kind: Service
metadata:
  name: client-clusterip
spec:
  type: ClusterIP
  selector:
    app: client
  ports:
  - port: 3000
    targetPort: 3000

```
***2. backend.yaml***

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: server-deployment
spec:
  selector:
    matchLabels:
      app: server
  template:
    metadata:
      labels:
        app: server
    spec:
      containers:
      - name: server
        image: saifuranik/goals-backend  
        ports:
        - containerPort: 5000
        resources:                     
          requests:
            cpu: "100m"                 
          limits:
            cpu: "200m"                 

---

apiVersion: v1
kind: Service
metadata:
  name: server-clusterip
spec:
  selector:
    app: server
  ports:
  - port: 5000
    targetPort: 5000
```

**3. Ingress.yaml**

   
   ```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-service
  annotations:
    kubernetes.io/ingress.class: 'nginx'
    nginx.ingress.kubernetes.io/use-regex: 'true'
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: saifuranik.com  #Add your Custom Domain
      http:
        paths:
          - path: /?(.*)      # Use ImplementationSpecific here
            pathType: ImplementationSpecific
            backend:
              service:
                name: client-clusterip
                port:
                  number: 3000
          - path: /api/?(.*)  # Use ImplementationSpecific here as well
            pathType: ImplementationSpecific
            backend:
              service:
                name: server-clusterip
                port:
                  number: 5000
```               




        
* apply all of those file by



 
```bash
 kubectl apply -f frontend.yaml
 kubectl apply -f backend.yaml
 kubectl apply -f ingress.yaml
```

# Step 4: Maping Custom Domain Name with LoadBalncer IP in Base Machine
  ### when we setup out Cluster with MetalLb LoadBalancer we will find a public Loadbalncer IP for access our app in Browser

  * we have to mapp it with our Custom Domian Name in Base machine’s following path
    
```bash
vim /etc/hosts
```
![Custom Doaim Mapping](https://github.com/user-attachments/assets/ab0bad55-0114-4604-aa03-34034cb5cbb8)


here you can mapp your custom domain with your cluster machine’s IP as shown in Image

so that we hit in Domain name in Browser it will redirect us to the LoadBalancer Public ip and we can Access the App

# Step 5: HPA — Horizontal Pod Auto-scaling
 * First we have to download matrics Server for HPA its very important

   ### STEP 1 — Download Mactrics Server

```bash
wget -O metricserver.yml kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

You can check by finding file metricserver.yml

```bash
ls metricserver.yml
```
you will find the yaml fiel



# Verify the Installation
 * After applying the manifest, you can check if the Metrics Server is running with the following command:

```bash
kubectl get pods -n kube-system
```
Look for a pod named metrics-server-*. It should be in the Running state.

![Screenshot from 2024-10-12 16-28-16](https://github.com/user-attachments/assets/221682bd-4483-40e6-b152-52b90314921d)


#### check logs if show this message that’s mean everything fine

![Screenshot from 2024-10-12 16-30-07](https://github.com/user-attachments/assets/d6b43c7c-c510-4156-b07e-a4451791272c)


   ### STEP 2— Make yaml files for frotnend & backend — for monitor them

 ***Frontend — client-hpa.yaml***

```bash
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: client-hpa
  namespace: default  # Change if your deployment is in a different namespace
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: client-deployment
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50  # Adjust the target utilization as needed

  ```

***Backend — server-hpa.yaml***


```bash
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: server-hpa
  namespace: default  # Change if your deployment is in a different namespace
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: server-deployment
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50  # Adjust the target utilization as needed
```
  * Apply and run those file by Command

```bash
kubectl apply -f client-hpa.yaml
kubectl apply -f server-hpa.yaml
```
You can check everying fine or not by
```bash
kubectl get hpa
```
![Screenshot from 2024-10-13 01-16-02](https://github.com/user-attachments/assets/1273c6c7-5848-4740-a3f5-45e573e35aec)



After those step hope you can access the app in browser by searching

http://saifuranik.com

![Custom Domain access](https://github.com/user-attachments/assets/9d658a84-b5cd-4bbc-a0d9-e88cb4af2d1e)


# Step 6: make Load & stress in app for test HPA
make ssh connection with new tab of terminal in same machine

find the front-end pod

```sh
kubectl get pod
```
then Go inside of the Frontend pod

```sh
kubectl exec <frontend pod name > -it -- /bin/bash
```
after enter inside pod now you can make stess here by follwing command

```sh
apt update && apt-get install -y stress

kubectl run cpu-load-pod --image=busybox --restart=Never -- /bin/sh -c "while true; do :; done"

stress --cpu 4 --timeout 900
```
you will show pods are automatically creting loads are higher follow image

![Screenshot from 2024-10-12 19-47-53](https://github.com/user-attachments/assets/6b5a5039-07ed-4c5f-9737-f3df5eb4ca61)



after stop stress and load on CPU they automatically delete and get back normal stage scale down

![Screenshot from 2024-10-12 20-12-44](https://github.com/user-attachments/assets/e099cee6-52c4-43d7-8aff-1112dac2087f)



### Next going to impliment it in jenkins CI/CD






























