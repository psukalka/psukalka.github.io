+++
date = '2025-04-23T01:10:18+05:30'
draft = false
title = 'Caddy'
+++
Just as a golf caddie makes the game easier for golfers by carrying their equipments and offering guidance, the Caddy web server aims to make web hosting easier by handling complicated tasks (like SSL/TLS certificates) automatically. With caddy, you don't need to worry about your certificate expiration. It started back in the days when Let's Encrypt was just starting and enabling HTTPS on your website was a headache and costly. 

With time, caddy has evolved too. When you setup HTTPS, caddy automatically enables HTTP/2. It also has built in support for HTTP/3, which Nginx supports on commercial version. Caddy is written Go and designed with modern protocols in mind from start, while NGINX has adapted to support newer protocols. 

Both are asynchronous and extremely fast in their own way. Both are capable of handling traffic at scale (with Nginx having longer track record). Caddy is preferred for its feature rich behaviour, quick adaptabiity to newer protocols and easy configuration style. 

---

Let's try to use Caddy as a web server. First create a simple index.html file: 
```
sudo mkdir -p /var/www/html
echo "<h1>Hello from Pavan!</h1>" | sudo tee /var/www/html/index.html
```

Now go to `/etc/caddy/Caddyfile` and add following lines:
```
:80 {
    root * /var/www/html
    file_server
}
```

That's it, your webserver is up and running. It's so simple with caddy. 

---
Let's leverage caddy to simulate a simple load balancer. 
Create 3 dummy index files at `/var/www/server1/index.html`, `/var/www/server2/index.html`, `/var/www/server3/index.html` . 
```
sudo mkdir -p /var/www/server1 /var/www/server2 /var/www/server3
echo "<h1>Server 1</h1>" | sudo tee /var/www/server1/index.html
echo "<h1>Server 2</h1>" | sudo tee /var/www/server2/index.html
echo "<h1>Server 3</h1>" | sudo tee /var/www/server3/index.html
```

Now, update `/etc/caddy/Caddyfile` to act as load balancer: 
```
:80 {
    reverse_proxy localhost:8081 localhost:8082 localhost:8083 {
        lb_policy round_robin
    }
}

:8081 {
    root * /var/www/server1
    file_server
}

:8082 {
    root * /var/www/server2
    file_server
}

:8083 {
    root * /var/www/server3
    file_server
}
```

That's it. It's so simple with caddy. We have enabled port 8081, 8082, 8083 to serve index files and load balanced them on port 80. 

Another interesting feature of Caddy is its dynamic configurability through API.
You can access current configuration at `curl http://localhost:2019/config` . 
You can load complete new config from a json file as follows: 
```
curl -X POST http://localhost:2019/load \
  -H "Content-Type: application/json" \
  -d @new-config.json
```

This can be quite helpful for containerized environments where pods can be created or destroyed