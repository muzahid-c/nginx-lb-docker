# NGINX as loadbalancer in Docker Container
This is a tutorial on how to configure nginx container as load balancer. We will use dockerfile to configure the load balancer. Custom network (bridge) will be used in Docker to have more control over the ip address assignment. The whole scenerio is tested in Ubuntu Ubuntu 20.04.3 LTS.

## Step 1 (Create a custom bridge network in docker)
Create custom bridge network using below command:

`docker network create --subnet=10.10.0.0/24 appnet0`

Above command will create a bridge named `appnet0` in docker having ip range `10.10.0.0/24`.
`appnet0` will have ip address `10.10.0.1`. This can be verified either by `ip addr` or `docker inspect appnet0`.

## Step 2 (Create the dockerfile for load balancer)
Now we will create the docker file for load balancer of Nginx. Dockerfile will be as following:

```
FROM nginx
# Add custom nginx conf for load balancer
COPY nginx.conf /etc/nginx/

```

The name of the dockerfile file in the repo is `lbdockerfile`. Use below command to build the load balancer image named lb with tag v1:

`docker build -t lb:v1 -f lbdockerfile .`

## Step 3 (Create nginx.conf for load balancer)
We need to add below lines in nginx.conf file. We will use `upstream` directive for this. See `nginx.conf` file in the repo to see the whole content.

```
#Load balancing IP
    
    upstream lb0 {
        server 10.10.0.7; #IP of node1
        server 10.10.0.8; #IP of node2
        server 10.10.0.9; #IP of node3
    }
    
    server {
        location / {
            proxy_pass http://lb0;
        }
    }
 
 ```
 This `nginx.conf` will be copied to load balancer Nginx container using dockerfile  
  
