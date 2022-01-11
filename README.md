# NGINX as loadbalancer in Docker Container
This is a tutorial on how to configure nginx container as load balancer. We will use dockerfile to configure the load balancer. Custom network (bridge) will be used in Docker to have more control over the ip address assignment. The whole scenerio is tested in Ubuntu Ubuntu 20.04.3 LTS

## Step 1
Create custom bridge network using below command:

`docker network create --subnet=10.10.0.0/24 appnet0`

Above command will create a bridge named `appnet0` in docker having ip range `10.10.0.0`
