# IBM_Containerized-Website_Project
In this project, I made a docker image of a website and deployed 2 versions of it. The first version was through Kubernetes and the second deployment was through the web UI of Red Hat OpenShift. 

# Objectives:
  Build and deploy a simple guestbook application. 
  
  Use OpenShift image streams to roll out an update.
  
  Deploy a multi-tier version of the guestbook application.
  
  Create a Watson Natural Language Understanding service instance on IBM Cloud.
  
  Bind the Natural Language Understanding service instance to your application.
  
  Autoscale the guestbook app.
  

![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/Project%20Topology.png)

## This project was deployed using IBM cloud and its service, Wastson Natural Language Understanding for text analysis

![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/Create%20an%20instance%20of%20Natural%20Language%20Understanding%20service.png)


# Guestbook application
Guestbook is a simple, multi-tier web application that will be built and deployed with Docker and Kubernetes. The application consists of a web front end, a Redis master for storage, a replicated set of Redis slaves, and a Natural Language Understanding service that will analyze the tone of the comments left in the guestbook.

There are two versions of this application. Version 1 is the simple application itself, while version 2 extends the application by adding additional features that leverage the Watson Natural Language Understanding service.

It will be deployed and managed entirely on OpenShift.

- [DockerFile](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Files/Dockerfile)

  ```
  FROM golang:1.15 as builder
  RUN go get github.com/codegangsta/negroni
  RUN go get github.com/gorilla/mux github.com/xyproto/simpleredis
  COPY main.go .
  RUN go build main.go

  FROM ubuntu:18.04

  COPY --from=builder /go//main /app/guestbook

  ADD public/index.html /app/public/index.html
  ADD public/script.js /app/public/script.js
  ADD public/style.css /app/public/style.css
  ADD public/jquery.min.js /app/public/jquery.min.js

  WORKDIR /app
  CMD ["./guestbook"]
  EXPOSE 3000

* Build the guestbook app
  ```
  docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1
  
![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/build_guestbook_v1.png)

* Push the image to IBM Cloud Container Registery
  ```
  docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
 
![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/push_guestbook_v1.png)

* List images
  ```
  ibmcloud cr images
  
 * Deploy guestbook app from the OpenShift internal registry
 
    Create an image stream that points to your image in IBM Cloud Container Registry.
    ```
   oc tag us.icr.io/$MY_NAMESPACE/guestbook:v1 guestbook:v1 --reference-policy=local --scheduled
  
With the --reference-policy=local option, a copy of the image from IBM Cloud Container Registry is imported into the local cache of the internal registry and made available to the cluster's projects as an image stream. The --schedule option sets up periodic importing of the image from IBM Cloud Container Registry into the internal registry. The default frequency is 15 minutes.


Now deploying the application using OpneShift web UI console:

* First we need to create an image container using the image form the internal registry on openshift

  
![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/first_deploy_app_osr.png)

![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/9%20-%20Guestbook%20App%20v1.png)


* Updating the title on the website by building and pusghing a new docker image after making changes to the index.html v1
    ```
    docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1 && docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
    
 ![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/3%20-%20Alan's%20Guestbook%20-%20v1.png)
 
 And the image stream history by swiching to the administratos role in OpenShift console:
 
 ![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/image_stream_history.png)
 
# Now to deploying the second version of the web app:

* Create a Redis master Deployment
* [redis-master-deployemnt.yaml](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Files/redis-master-deployment.yaml)

   ```
   oc apply -f redis-master-deployment.yaml
 
   oc get pods
   
![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/deploy_redis_5.png)

[redis-master-service.yaml](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Files/redis-master-service.yaml)

* Create the Redis master service
   ```
   oc apply -f redis-master-service.yaml
   
If you click on the redis-master Deployment in the Topology view, you should now see the redis-master Service in the Resources tab.

![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/deploy_redis_7b.png)


* Create the Redis slave Deployment
[redis-slave-deployment.yaml](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Files/redis-slave-deployment.yaml)

       ```
        oc apply -f redis-slave-deployment.yaml
  
to verify:

    ``` 
    oc get deployments
        
 ![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/deploy_redis_10.png)
    
To list pods:

    ```
     oc get pods
    
![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/deploy_redis_11.png)

* Create the Redis slave Service:
[redis-slave-service.yaml](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Files/redis-slave-service.yaml)

      ```
    oc apply -f redis_slave_service.yaml
    
* Set up and configure the analyzer from IBM cloud and then deploy the YAML files.

- [analyzer-deployment.yaml](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Files/analyzer-deployment.yaml)
- [analyzer-service.yaml](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Files/analyzer-service.yaml)
         
The final topology should look like:
![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/deploy_analyzer_7a.png)

![](https://github.com/AlanShami/IBM_Containerized-Website_Project/blob/main/Images/deploy_analyzer_8.png)
