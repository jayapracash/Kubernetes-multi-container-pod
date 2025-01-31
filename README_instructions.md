# Multi-Container Pods in Kubernetes
Simple tutorial to demonstrate the concept of packaging multiple containers into a single pod. 
* Web Pod has a Python Flask container and a Redis container
* DB Pod has a MySQL container
* When data is retrieved through the Python REST API, it first checks within Redis cache before accessing MySQL
* Each time data is fetched from MySQL, it gets cached in the Redis container of the same Pod as the Python Flask container
* When the additional Web Pods are launched manually or through a Replica Set, co-located pairs of Python Flask and Redis containers are scheduled together

![Architecture](https://github.com/janakiramm/Kubernetes-multi-container-pod/blob/master/multi-container-pod.png?raw=true)

Make sure that you have access to a Kubernetes cluster.

## Build a Docker image from existing Python source code and push it to Docker Hub. Replace DOCKER_HUB_USER with your Docker Hub username.
```
cd Build
#do this step only when we want to run in docker compose or custom images instead of janakiramm/py-red
#use docker-compose up to build all these images automatically ( web->redis->mysql, here web is linked with both redis ,mysql)
docker build . -t <DOCKER_HUB_USER>/py-red-sql   #not necessary
docker push <DOCKER_HUB_USER>/py-red-sql    #not necesary
```

## Deploy the app to Kubernetes
```
cd ../Deploy
docker pull janakiramm/py-red  #do this if kubectl unable to create web-pod
kubectl create -f db-pod.yml       
kubectl create -f db-svc.yml
kubectl create -f web-pod-1.yml
kubectl create -f web-svc.yml
```

## Check that the Pods and Services are created
```
kubectl get pods
kubectl get svc
```

## Get the IP address of one of the Nodes and the NodePort for the web Service. Populate the variables with the appropriate values
```
kubectl get nodes
kubectl describe svc web

kubectl get nodes
export NODE_IP=<NODE_IP>
export NODE_PORT=<NODE_PORT>
```

## Initialize the database with sample schema
```
curl http://$NODE_IP:$NODE_PORT/init     #this created users db in mysql through flask(python container->redis->mysql)
```
## Insert some sample data
```
#this curl command only works in linux/ or git bash in windows
curl -i -H "Content-Type: application/json" -X POST -d '{"uid": "1", "user":"John Doe"}' http://$NODE_IP:$NODE_PORT/users/add
curl -i -H "Content-Type: application/json" -X POST -d '{"uid": "2", "user":"Jane Doe"}' http://$NODE_IP:$NODE_PORT/users/add
curl -i -H "Content-Type: application/json" -X POST -d '{"uid": "3", "user":"Bill Collins"}' http://$NODE_IP:$NODE_PORT/users/add
curl -i -H "Content-Type: application/json" -X POST -d '{"uid": "4", "user":"Mike Taylor"}' http://$NODE_IP:$NODE_PORT/users/add
```

## Access the data 
```
curl http://$NODE_IP:$NODE_PORT/users/1
```
## The second time you access the data, it appends '(c)' indicating that it is pulled from the Redis cache
```
curl http://$NODE_IP:$NODE_PORT/users/1

#EX: 100     5  100     5    0     0    161      0 --:--:-- --:--:-- --:--:--   312JP(c) ,here c means from cache redis

```

## Create 10 Replica Sets and check the data
```
kubectl create -f web-rc.yml
curl http://$NODE_IP:$NODE_PORT/users/1
```


