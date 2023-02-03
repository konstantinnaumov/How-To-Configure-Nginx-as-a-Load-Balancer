![Иллюстрация к проекту](https://github.com/konstantinnaumov/naumov_project/blob/main/Task%201/Nginx-Load-Balancing.webp)
# How To Configure Nginx as a Load Balancer 

## What is load balancing?
Load balancing involves effectively distributing incoming network traffic across a group of backend servers. A load balancer is tasked with distributing the load among the multiple backend servers that have been set up. There are multiple types of load balancers

Application Load balancer
Network Load Balancer
Gateway load balancer
Classic Load balancer
There are multiple load balancer examples and all have different use cases

Haproxy

Nginx

mod_athena

Varnish

Balance

Linux Virtual Server(LVS)

In this article, I am going to cover Nginx as a load balancer.

## What Is Nginx Load Balancing?
[Nginx](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/ "Nginx Guide")
 is a high-performance web server which can also be used as a load balancer. Nginx load balancing refers to the process of distributing web traffic across multiple servers using Nginx.

This ensures that no single server is overloaded and that all requests are handled in a timely manner. Nginx uses a variety of algorithms to determine how to best distribute traffic, and it can also be configured to provide failover in case one of the servers goes down.

You can use either Nginx open source or Nginx Plus to load balance HTTP traffic to a group of servers.

Personally, I use Nginx open source to set up my load balancers and that is what I am going to show you in this article.

## Advantages of load balancing
Load balancing helps in scaling an application by handling traffic spikes without increasing cloud costs. It also helps remove the problem of a single point of failure. Because the load is distributed, if one server crashes, the service would still be online.
## Installing Nginx
The first step is to install Nginx. Nginx can be installed in Debian, Ubuntu or CentOS. I am going to use Ubuntu.

`sudo apt-get update`

`sudo apt-get install nginx`
## Configure Nginx as a load balancer
The next step is to configure Nginx. We can edit an existing file configuration for the load balancer.

`cd /etc/nginx/`

`sudo nano nginx.conf`

    http {
     upstream app{
        server 10.2.0.100; 
        server 10.2.0.101;
        server 10.2.0.102;
     }
  
    # This server accepts all traffic to port 80 and passes it to the upstream. 
    # Notice that the upstream name and the proxy_pass need to match.
  
    server {
       listen 80; 
       
       server_name mydomain.com;
  
        location / {
            include proxy_params;
           
            proxy_pass http://app;
            
  
            proxy_redirect off;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
     }
    }

We will need to define the upstream directive and server directive in the file. Upstream defines where Nginx will pass the requests upon receiving them. It contains upstream server group (backend) IP addresses to which requests can be sent to based on the load balancing method chosen. By default, Nginx uses a round-robin load balancing method to distribute the load across the servers.

The server segment defines the port 80 through which Nginx will receive requests. It also contains a proxy_pass variable

The proxy_pass variable is used to tell NGINX where to send traffic that it receives. In this case, the proxy_pass variable is set to point to 3 servers. This tells NGINX to forward traffic that it receives to any of the upstream servers’ IPs provided. Nginx acts as both a reverse proxy and a load balancer.

A reverse proxy is a server that sits in between backend servers and intercepts requests from clients.

## Selecting a load-balancing method
The next step is to determine the load balancing method to use. There are multiple load balancing methods we can use. They include:

## Round Robin
Round Robin is a load balancing method where each server in a cluster is given an equal opportunity to process requests. This method is often used in web servers, where each server request is distributed evenly among the servers.

The load is distributed in rotation meaning that each server will have its time to execute a request. For example, if you have three upstream servers, A, B and C, then the load balancer will first distribute the load to A then to B and finally to C before redistributing the load to A. It is fairly simple and does have its fair share of imitations.

One of the limitations is that you will have some servers idle simply because they will be waiting for their turn. In this example, if A is given a task and executes it in a second, it would then mean it would be idle until it is next assigned a task. By default, Nginx uses round robin to distribute load among servers.

## Weighted Round Robin
To solve the issue of server idleness, we can use server weights to instruct Nginx on which servers should be given the most priority. It is one of the most popular load balancing methods used today.

This method involves assigning a weight to each server and then distributing traffic among the servers based on these weights. This ensures that servers with more capacity receive more traffic, and helps to prevent overloading of any one server.

This method is often used in conjunction with other methods, such as session Persistence, to provide an even distribution of load across all servers. The application server with the highest weight parameter will be given priority(more traffic) as compared to the server with the least number(weight).

We can update the Nginx configuration to include the server weights

    http {
     upstream app{
        server 10.2.0.100 weight=5; 
        server 10.2.0.101 weight=3;
        server 10.2.0.102 weight=1;
     }
  
    # This server accepts all traffic to port 80 and passes it to the upstream. 
    # Notice that the upstream name and the proxy_pass need to match.
  
    server {
       listen 80; 
       
       server_name mydomain.com;
  
        location / {
            include proxy_params;
           
            proxy_pass http://app;
            
  
            proxy_redirect off;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }
     }
    } 

This is just one example of configuring the Nginx service as a load balancer. See the [article](https://www.iankumu.com/blog/nginx-load-balancing/ )  for more settings.

Check the correctness of our configuration file:

`sudo nginx -t`

If there are no errors, restart the service and check its status:

`systemctl restart nginx`

`sudo systemctl status nginx`

P.S. in case of unexpected errors, it is better to study the logs:

`sudo nano /var/log/syslog`

`sudo nano /var/log/nginx/error.log`

and check if the ports are busy

`sudo netstat -plant | grep 80`
