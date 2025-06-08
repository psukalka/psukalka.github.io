+++
date = '2025-06-08T20:57:38+05:30'
draft = false
title = 'Apache Bench'
+++
I wanted to replace `python manage.py runserver` command with `uwsgi` for production. But before taking any decision I want it to be assessed with some data. Even if some blog says uwsgi can handle better concurrent load, memory management and caching why should I believe it ? How can I test to be clear. 

For starter, I can write a python script that can send n number of requests and I can make them concurrent calls using async / threads. But is there any other readily available option ? When I explored, there is an option: `Apache Bench (ab)`

`ab` is a simple utility available with any linux distribution. If not available, you can install it with: 
```
sudo apt install apache2-utils
```

Once, installed you can run it simply with: 
```
ab -n 100 -c 10 <url>
```

This will run test by sending 100 urls with concurrency level of 10. You can tweak the numbers as per your needs. When I ran it for one of our production urls, this is the output I got: 
```
This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking kukufm.com (be patient).....done


Server Software:        nginx/1.18.0
Server Hostname:        kukufm.com
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES128-GCM-SHA256,2048,128
Server Temp Key:        ECDH P-256 256 bits
TLS Server Name:        kukufm.com

Document Path:          <REMOVED ACTUAL URL>
Document Length:        37810 bytes

Concurrency Level:      10
Time taken for tests:   1.841 seconds
Complete requests:      100
Failed requests:        0
Total transferred:      3854600 bytes
HTML transferred:       3781000 bytes
Requests per second:    54.33 [#/sec] (mean)
Time per request:       184.056 [ms] (mean)
Time per request:       18.406 [ms] (mean, across all concurrent requests)
Transfer rate:          2045.17 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        4    7   2.0      7      18
Processing:   126  156  27.3    146     233
Waiting:      125  155  27.2    146     232
Total:        132  163  27.4    153     239

Percentage of the requests served within a certain time (ms)
  50%    153
  66%    171
  75%    179
  80%    186
  90%    210
  95%    223
  98%    238
  99%    239
 100%    239 (longest request)
 ```

 As you can see, it gives P50, .. P95, P99, all in one go. You can even run timed tests with `-t` option. Post requests can also be sent using `-p` option. Example: 
 ```
 # Create a JSON file first
echo '{"username":"test","password":"test123"}' > post_data.json

ab -n 100 -c 5 -p post_data.json -T application/json http://localhost:8000/api/login/
```

With this utility, you can easily run performance test on your feature before sending it to production. 