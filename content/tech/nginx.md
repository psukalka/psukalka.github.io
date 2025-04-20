+++
date = '2025-04-20T21:56:59+05:30'
draft = false
title = 'Nginx'
+++
Nginx can be used for multiple purposes like web serving, load balancing or reverse proxying. It was designed to solve C10k problem so obviously it can handle large number of connections simultaneously. Besides, it also excels at serving static content. But how does it achieve this ? 

Nginx follows an asynchronous event-driven architecture. There is a master process that manages all worker processes. Typically there are as many workers as there are CPU cores. Each worker has an event loop running inside it. When an event comes on socket, OS notifies worker. Worker registers interest in the event and moves ahead. When data is received from client, a read event is triggered which is then processed by worker. Thus, the worker continues with its events without being blocked. 

Similarly when an event is received for static file, nginx directly transfers that file using `sendfile()` (just like kafka). This bypasses the user space. File is directly transferred from kernel space without ever being brought into user space thus improving performance. Also, for larger files nginx implements Asynchronous IO so that its workers aren't blocked. 

Thus, with asynchronous event-driven architecture and efficiently leveraging OS capabilities Nginx is able to deliver terrific performance with little memory footprint.
---
Too much of tech-talk. How do I use nginx ? 

Nginx loads its config from `/etc/nginx/nginx.conf` file. With `include` directive we can import multiple config files. 

For most practical purposes there are few important blocks to note:
- `http` : operates at application (i.e. level 7) and can be used to filter requests
- `stream` : operates at level 4 and can be used to filter TCP / UDP connections or IP addresses
- `events` : for event level settings of workers
- `server` : single server level block

There are few important directives to note: 
- `listen` : to tell which port to listen on
- `location` : the path that should be matched when request comes in. Request is served with the first matched location logic. So, if there are any regex patterns, they should be kept at the end
- `upstream` : to group servers (used for load balancing)
- `proxy_*` : when nginx acts as proxy / load balancer and forwards request to some upstream 
---

With this knowledge, let's try to configure a simple load balancer. Say, there are 3 servers `server1`, `server2`, `server3` . For testing assume that they will be serving `index.html` file. 

Config of each server can look like : 
```
server {
    listen 8081;
    root /usr/share/nginx/html/server1;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
It means `server1` is listening on port 8081 and serving `index.html` file. 

We can create similar config for `server2` and `server3` . 

Now let's group these servers with `upstream` directive.
```
upstream backend {
    server 127.0.0.1:8081;
    server 127.0.0.1:8082;
    server 127.0.0.1:8083;
}
```

Finally, lets use these `backend` servers to serve requests. We haven't provided any algo so servers will be picked in round-robin fashion.

```
server {
    listen 80;
    server_name _;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

We are telling nginx to listen on port 80 (i.e. http) traffic. When a request comes, we are asking it to direct locally to one of the `backend` servers (in round-robin fashion). Thus, requests will be distributed among servers and we have configured a simple load balancer. 