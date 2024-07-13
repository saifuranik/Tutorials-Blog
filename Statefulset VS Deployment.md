# Statefulset VS Deployment 

## What Is Deployment
Deployment is the easiest and most used resource for deploying your application. It is a Kubernetes controller that matches the current state of your cluster to the desired state mentioned in the Deployment manifest.

e.g. If you create a deployment with 1 replica, it will check that the desired state of ReplicaSet is 1 and current state is 0, so it will create a ReplicaSet, which will further create the pod. 

If you create a deployment with name counter, it will create a ReplicaSet with name counter-<replica-set-id>, which will further create a Pod with name counter-<replica-set->-<pod-id>.

Deployments are usually used for stateless applications. However, you can save the state of deployment by attaching a Persistent Volume to it and make it stateful,
but all the pods of a deployment will be sharing the same Volume and data across all of them will be same.

For deploying the sample counter app using a deployment, we will be using the following manifest, 
you can deploy it by copying the below manifest and saving it in a file e.g. deployment.yaml, and then applying by

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: counter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: counter
  template:
    metadata:
      labels:
        app: counter
    spec:
      containers:
      - name: counter
        image: "kahootali/counter:1.1"
        volumeMounts:
        - name: counter
          mountPath: /app/
      volumes:
      - name: counter
        persistentVolumeClaim:
          claimName: counter
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: counter
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
  storageClassName: efs
```

#### kubectl apply -f deployment.yaml

...



If we deploy the above deployment, and see the logs of the pod, 
you will be able to see the log in order like 1,2,3,4,5,6,7,8,9,10,11,12...


![image](https://github.com/user-attachments/assets/16486445-38b8-4623-9dd1-ae45693b20eb)


Now if you scale the deployment to 2 by running

#### kubectl scale deployment counter --replicas=2

you can see a new pod created, if you check the logs of the new pod, 

its logs will not start from 1 rather it will start from the last number of the 1st pod.

![image](https://github.com/user-attachments/assets/bcf170ed-c02d-4375-867d-b447fd79d931)


If you see the logs, they are starting from 73, meaning that the previous pod had written till 72 in the file and 

they both are sharing the same file and volume and data is shared across all pods of a Deployment. Also if you check 

the Persistent Volume Claims(PVCs), only one PVC will be created that both the pods will be sharing so it can cause Data Inconsistency.




Deployments, as discussed, creates a ReplicaSet which then creates a Pod so whenever you update the deployment using RollingUpdate(default) strategy, 

a new ReplicaSet is created and the Deployment moves the Pods from the old ReplicaSet to the new one at a controlled rate. Rolling Update means that 

the previous ReplicaSet doesn’t scale to 2 unless the new ReplicaSet is up & running ensuring 100% uptime. If an error occurs while updating, the new 

ReplicaSet will never be in Ready state, so old ReplicaSet will not terminate again ensuring 100% uptime in case of a failed update. In Deployments, 

you can also manually roll back to a previous ReplicaSet, if needed in case if your new feature is not working as expected.


## What is satefulset
StatefulSet(stable-GA in k8s v1.9) is a Kubernetes resource used to manage stateful applications. 

It manages the deployment and scaling of a set of Pods, and provides guarantee about the ordering and uniqueness of these Pods.

StatefulSet is also a Controller but unlike Deployments, it doesn’t create ReplicaSet rather itself 

creates the Pod with a unique naming convention. e.g. If you create a StatefulSet with name counter, 

it will create a pod with name counter-0, and for multiple replicas of a statefulset, their names will 

increment like counter-0, counter-1, counter-2, 


Statefulset normally maintain Name sequence while creating any pods, and its also set those name as parmanet to the pods

means, if some pod will deleted for any reason and re-create it again it will assign same name what it was have previously

so thats why there is no issue will create in future if some app's pod want to read data from any databse pod they can easily reach

with old name and everything will syncronige

and in future if we increase replicaset pods ammount suppore 3 to 5 setfulset assighn new pods name as sequence like extra two new pod

name will be assign as pod 4 & pod 5



### when pod delete from satefulset 
satefulset when dete any pod it also delete them reverse sequencially, means first delete last number of pod then previous 
##### eg. it will 
- Fisrt delete pod no.5 
- then pod no. 4
- then pod no.3
- then pod no.2 & so on



# How Sycronaigation happen in Stateful set

...

![image](https://github.com/saifuranik/Tutorials-Blog/assets/102476507/89528701-6194-41f2-80b1-21e8381be1ed)

...

**Every replica of a stateful set will have its own state, and each of the pods will be creating its** 

***own PVC(Persistent Volume Claim).So a statefulset with 3 replicas will create 3 pods, each having**

**its own Volume, so total 3 PVCs**


For deploying the sample counter app using a statefulset, we will be using the following manifest

you can deploy it by copying the below manifest and saving it in a file e.g. statefulset.yaml,

and then applying by

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: counter
spec:
  serviceName: "counter-app"
  selector:
    matchLabels:
      app: counter
  replicas: 1 
  template:
    metadata:
      labels:
        app: counter
    spec:      
      containers:
      - name: counter
        image: "kahootali/counter:1.1"  
        volumeMounts:
        - name: counter
          mountPath: /app/      
  volumeClaimTemplates:
  - metadata:
      name: counter
    spec:
      accessModes: [ "ReadWriteMany" ]
      storageClassName: efs
      resources:
        requests:
          storage: 50Mi
```

**kubectl apply -f statefulset.yaml**

If you deploy the above statefulset, and see the logs of the pod, 

you will be able to see the log in order like 1,2,3,…

![image](https://github.com/user-attachments/assets/d7811974-02fa-4536-acc1-b2b1700bef09)


Here, you can see the logs start from 1. Now if we scale up the statefulset to 3 replicas by run

**kubectl scale statefulsets counter --replicas=3**

it will first create a new pod counter-1, and once that pod is ready, then another pod counter-2. 
then next pod counter -3 and so on seqeuncially and parmanent name

**those name will not change, even pods will delete & re-create**

The new pods will have their own Volume and if you check the logs, the count will again start 

from 1 for the new pods, unlike in Deployments as we saw earlier.

![image](https://github.com/user-attachments/assets/78defd66-981c-46ad-9132-d90e349a0aba)



Here, the logs are again starting from 1, as this pod has its own Volume,

so it doesn’t read the file of 1st pod. And if we see the Persistent Volume Claims,

their will be 3 claims created as we had scaled the replicas to 3.

![image](https://github.com/user-attachments/assets/0354864b-e0bb-4378-aac5-1a19d03039d3)



StatefulSets don’t create ReplicaSet or anything of that sort, so you cant rollback a StatefulSet to a previous version. 

You can only delete or scale up/down the Statefulset. If you update a StatefulSet, it also performs RollingUpdate 

i.e. one replica pod will go down and the updated pod will come up, then the next replica pod will go down in same manner 

e.g. If I change the image of the above StatefulSet, the counter-2 will terminate and once it terminates completely, 

then counter-2 will be recreated and counter-1 will be terminated at the same time, similarly for next replica 

i.e. counter-0. If an error occurs while updating, so only counter-2 will be down, counter-1 & counter-0 will still be up,

running on previous stable version. Unlike Deployments, you cannot roll back to any previous version of a StatefulSet.


![image](https://github.com/user-attachments/assets/87d81df3-224b-4c1e-b43d-08a15ce0dec4)



If the primary goes down, any of the secondary replica will become primary and the StatefulSet controller

will create a new replica in account of the one that went down, which will now become a secondary replica.

## DaemonSet

A DaemonSet is a controller that ensures that the pod runs on all the nodes of the cluster. 

If a node is added/removed from a cluster, DaemonSet automatically adds/deletes the pod.

---
However, Daemonset automatically doesn’t run on nodes which have a taint e.g. Master. Y

ou will have to specify the tolerations for it on the pod.

Taints are a way of telling the nodes to repel the pods i.e. no pods will be schedule on this node unless

the pod tolerates the node with the same toleration. The master node is already tainted 

---
For deploying the sample counter app using a daemonset, we will be using the following manifest. 

you can deploy it by copying the below manifest and saving it in a file e.g. daemonset.yaml, and then applying by

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: counter-app
spec:  
  selector:
    matchLabels:
      app: counter
  template:
    metadata:
      name: counter-app
      labels:
        app: counter
    spec:
      tolerations: 
      - effect: NoSchedule
        operator: Exists
      containers:
      - name: counter
        image: "kahootali/counter:1.1"
        volumeMounts:
        - name: counter
          mountPath: /app/
      volumes:
      - name: counter
        persistentVolumeClaim:
          claimName: counter
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: counter
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
  storageClassName: efs
```

**kubectl apply -f daemonset.yaml**


When you deploy the daemonset, it will create pods equal to the number of nodes. In terms of behavior,

it will behave the same as Deployments i.e. all pods will share the same Persistent Volume.

![image](https://github.com/user-attachments/assets/68990201-173c-4523-aca0-b947d3f28e75)


These are the logs of a pod of DaemonSet, you can see the logs are not in order, ( e.g. 25,26 then 30 )

meaning that all pods are sharing the same Volume. Also only one PVC will be created 

that all pods will be sharing.

If you update a DaemonSet, it also performs RollingUpdate i.e. one pod will go down and the updated pod will come up,

then the next replica pod will go down in same manner e.g. If I change the image of the above DaemonSet, one pod will go down,

and when it comes back up with the updated image, only then the next pod will terminate and so on. If an error occurs while updating, 

so only one pod will be down, all other pods will still be up, running on previous stable version. Unlike Deployments, 

you cannot roll back your DaemonSet to a previous version.



# Thank You for your time











































