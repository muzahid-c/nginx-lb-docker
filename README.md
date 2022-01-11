# NGINX as loadbalancer in Docker Container
This is a tutorial on how to configure nginx container as load balancer. We will use dockerfile to configure the load balancer. Custom network (bridge) will be used in Docker to have more control over the ip address assignment. The whole scenerio is tested in Ubuntu 20.04.3 LTS. The load balancer configured here is HTTP Load Balancing.

## Step 1 (Create a custom bridge network in docker)
Create custom bridge network using below command:

`docker network create --subnet=10.10.0.0/24 appnet0`

Above command will create a bridge named `appnet0` in docker having ip range `10.10.0.0/24`.
`appnet0` will have ip address `10.10.0.1`. This can be verified either by `ip addr` or `docker inspect appnet0`.

## Step 2 (Create the dockerfile for load balancer)
Now we will create the docker file for load balancer of Nginx. The name of the dockerfile file in the repo is `lbdockerfile` and it contains below lines only:

```
FROM nginx
# Add custom nginx conf for load balancer
COPY nginx.conf /etc/nginx/

```

Use below command to build the load balancer image named `lb` with tag `v1`:

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
 This `nginx.conf` will be copied to load balancer Nginx container using dockerfile. 
 
## Step 4 (Running the load balancer image with custom IP)
We will now run the load balancer with IP 10.10.0.5 using bridge `appnet0`. Run below command:

`docker run --net appnet0 --ip 10.10.0.5 -p 80:80 --name nginx_lb lb:v1`

Here we use bridge `appnet0` and ip `10.10.0.5` for load balancer. We gave custom name `nginx_lb`. Also we are binding port 80 of container to port 80 of host machine. Now load balancer is accessible from outside.

If everything goes well then load balancer will run perfectly. If we launch browser (Chrome recommended) and put ip 10.10.0.5 nothing will happen as nodes are not running right now. We can see from console that load balancer is trying to connect IP 7,8,9 one by one.

```
2022/01/11 05:58:49 [error] 31#31: *1 connect() failed (113: No route to host) while connecting to upstream, client: 10.10.0.1, server: , request: "GET / HTTP/1.1", upstream: "http://10.10.0.7:80/", host: "10.10.0.5"
2022/01/11 05:58:49 [warn] 31#31: *1 upstream server temporarily disabled while connecting to upstream, client: 10.10.0.1, server: , request: "GET / HTTP/1.1", upstream: "http://10.10.0.7:80/", host: "10.10.0.5"
2022/01/11 05:58:52 [error] 31#31: *1 connect() failed (113: No route to host) while connecting to upstream, client: 10.10.0.1, server: , request: "GET / HTTP/1.1", upstream: "http://10.10.0.8:80/", host: "10.10.0.5"
2022/01/11 05:58:52 [warn] 31#31: *1 upstream server temporarily disabled while connecting to upstream, client: 10.10.0.1, server: , request: "GET / HTTP/1.1", upstream: "http://10.10.0.8:80/", host: "10.10.0.5"
2022/01/11 05:58:55 [error] 31#31: *1 connect() failed (113: No route to host) while connecting to upstream, client: 10.10.0.1, server: , request: "GET / HTTP/1.1", upstream: "http://10.10.0.9:80/", host: "10.10.0.5"
2022/01/11 05:58:55 [warn] 31#31: *1 upstream server temporarily disabled while connecting to upstream, client: 10.10.0.1, server: , request: "GET / HTTP/1.1", upstream: "http://10.10.0.9:80/", host: "10.10.0.5"
10.10.0.1 - - [11/Jan/2022:05:58:55 +0000] "GET / HTTP/1.1" 502 559 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36" "-"
10.10.0.1 - - [11/Jan/2022:05:58:55 +0000] "GET /favicon.ico HTTP/1.1" 502 559 "http://10.10.0.5/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36" "-"
2022/01/11 05:58:55 [error] 31#31: *1 no live upstreams while connecting to upstream, client: 10.10.0.1, server: , request: "GET /favicon.ico HTTP/1.1", upstream: "http://lb0/favicon.ico", host: "10.10.0.5", referrer: "http://10.10.0.5/
```

Eventually the browser will show `504 Gateway Time-out`.
![bad_gw](https://user-images.githubusercontent.com/36810834/148908786-45e83bda-a7a5-4954-9d22-78e6cffac6a8.png)

## Step 5 (Running node with custom IP)
For this we will use another docker file to upload custom index.html as we want to see which node is connected each time a browser send the requrest. See node1, node2 and node3 folder for docker file and custom index.html. Run below commands to build the image and launch the container. Remember to run below command in each folder.

```
docker build -t node1:v1 .

docker run --net appnet0 --ip 10.10.0.7 --name node1 node1:v1
``` 
```
docker build -t node2:v1 .

docker run --net appnet0 --ip 10.10.0.8 --name node2 node2:v1
``` 
```
docker build -t node3:v1 .

docker run --net appnet0 --ip 10.10.0.9 --name node3 node3:v1
``` 
   
Now you can see browser is getting response and connecting each node once hit refresh!

![node1](https://user-images.githubusercontent.com/36810834/148900320-708fb240-3d4f-49c0-b98f-84403da9821b.png)
![node2](https://user-images.githubusercontent.com/36810834/148900329-198d7ad7-ebaa-445f-997c-79dcee7e0862.png)
![node3](https://user-images.githubusercontent.com/36810834/148900330-e6c6c5a5-0530-4263-a436-8fa8ff96a7e2.png)

### Some load balancing methods
By default Nginx use Round Robin method. We can use other methods too depending the situations. Please see refenece below:

https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/
