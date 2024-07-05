# Statefulset VS Deployment 

## What Is Deployment
Deployment is used for stateless applications where Appllication/Pods are interchangeable with each other and can be freely 
added or removed without maintaing any serial orders ( eg. 1st , 2nd, 3rd ). It provides declarative updates, making it easy 
to manage replicas without needing specific identity or storage constrain.

## Explain Example of Deployment
if we create 3 web app pod with deployment replicaset and 3 Databse pods for read & write operation between app and Databse.
here everything will be fine in initially, and one more thing we have to keep in mind commmunuication between pods and Databse
establish by a service who randomly send traffic from pods to Databse, 

eg: if app's pod no.1 want to write something data in databse Service send app's pod no.1 to Databse's pods no. 2 for wirte his data and he done it.
oky! now when another app's pod no.3 want to read this data from databse, Suppose Service send app's pod no.3 to Databse's pod no. 1 & in there 

app's pod no.3 didn't find anyhthing or any data why? because as we mentioned earlier when ( eg. first line ) Service send app's pod no.1 to Databse's pods no. 2 for wirte his data

so thats why other two databse's pod no. 1 & 3 not updated with data what app's pod no. 1 write in Databse's pods no. 2 
& why thats append beacuse deployment didn't use any syscronaization between all of pods data. 

...

![Screenshot from 2024-07-05 20-23-53](https://github.com/saifuranik/Tutorials-Blog/assets/102476507/801eb6c5-ba35-42b9-9404-a93f0f4d5f9c)




So Basically, Deployment Backlogs is deployment not syscing all of its pods data between others eg. if some one write some data in pod no.1 
that data only keep in here it will not copy and trasfer in other pods no.2 & 3 .



![Screenshot from 2024-07-05 20-38-22](https://github.com/saifuranik/Tutorials-Blog/assets/102476507/eaad4f69-2cf3-4aa7-b556-293ee288ec77)


in the image we can notice deployment only update pod no.2 with new data & other remain same at previous stage they not updated yet
and if some apps pod to other pods they cant not get latest data from them which avilable only in pod no.2 

thats why came satefulset in the picture.

## What is satefulset
Satefulset will normally keep maintain data syncing betweet all of pods in the cluster, so that all of pods keep dated with latest data and information
and whenever some one came to read any data to the Database they whenever server send them they can easily read latest Data from any where

Lets undertsand the work process of sateful set

![Screenshot from 2024-07-05 20-42-48](https://github.com/saifuranik/Tutorials-Blog/assets/102476507/98d707ec-cc4a-4258-b181-22ac693a7aaa)



Statefulset normally maintain Name sequence while creating any pods, and its also set those name as parmanet to the pods
means, if some pod will deleted for any reason and re-create it again it will assign same name what it was have previously

so thats why there is no issue will create in future if some app's pod want to read data from any databse pod they can easily reach
with old name and everything will syncronige

and in future if we increase replicaset pods ammount suppore 3 to 5 setfulset assighn new pods name as sequence like extra two new pod
name will be assign as pod 4 & pod 5

### when pod delete from satefulset 
satefulset when dete any pod it also delete them reverse sequencially, means first delete last number of pod then previous 
##### eg.
- Fisrt delete pod no.5 
- then pod no. 4
- then pod no.3
- then pod no.2 & so on

# How Sycronaigation happen in Stateful set

...

![image](https://github.com/saifuranik/Tutorials-Blog/assets/102476507/89528701-6194-41f2-80b1-21e8381be1ed)

...

when create app's pod and Databse pod we make or declare one of databse pod as "Central pod"
suppose "pod no.1" will be central pod and all of data whatever from app want to write in
database they came into the pod no.1 and write or store there

then if we mention in the replica set as 3 pods statefulset will not create rest of two togather
he first create 2nd pod and mention central pod name in 2nd pod's Hostname - eg. "Hostname = pod 1"

so that 2nd pod copy or sync all of the data from 1st pod and when 2nd pod's everything is copy done 
and its up and running 100%

then 
it will strating create 3rd pod ( before 2nd pod 100% up and ruuning with sync data satetfulset will not create 3nd number pod )
after 3rd pod create, it will first copy all data from pod no.2 and then connected with "central pod" for sysnc brand new data 
which will later come in to central pod. ( because satefulset also set "Hostname = pod 1" as central pod of 3rd pod )

then 
if we want to create another new pod suppose pod no.4 , it will create as same process
first clone all data from 3rd pod and then connected himself with "Central pod" and rest of
latest data will sync when latest data come into central data from app's pod.

....

![image](https://github.com/saifuranik/Tutorials-Blog/assets/102476507/c3c8915e-a1e6-4536-8e21-c4e3c1fcd653)



# example & deffent between Deployment and Satefulset YAML

![image](https://github.com/saifuranik/Tutorials-Blog/assets/102476507/4459ced7-8a11-44de-850f-0b45cf04805d)
























