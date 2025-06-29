+++
date = '2025-06-29T18:10:56+05:30'
draft = false
title = 'Gunicorn'
+++
While development, we run Django application using `python manage.py runserver`. But that is not recommended for production use. Why so ? 

Firstly, in development mode, lot of details are returned which expose internals of system. This is a security risk in production. Secondly, application doesn't auto-restart if it crashes with some error. Memory leaks in application accumulate over time eventually leading to crashing of server. Load can't be balanced across CPU cores. Then static files are served synchronously this would result in queuing of requests. In dev mode server runs as a single thread so it can't handle multiple concurrent requests thus increasing overall response time. There isn't connection pooling for database connections. Overall it is not a reliable setup that should be used in production. 

Django is a python web framework. It focuses on increasing speed of development. Application serving is not its expertise. For production deployment, we should use WSGI servers like Gunicorn / uWSGI. 

Gunicorn is sufficient for simple production needs. If you need advanced routing capabilities / static file serving capabilities uWSGI might be better option. 

Using gunicorn is quite simple. Instead of `python manage.py runserver` , you will run your application as `gunicorn --bind 0.0.0.0:8000 --workers 4 myproject.wsgi:application` . And you are up with gunicorn. But in production you might want more control over your server. You can configure gunicorn using cocnfig file `gunicorn.conf.py` and pass it to gunicorn as `gunicorn --config gunicorn.conf.py myproject.wsgi:application` 

You can control number of workers using `workers` param. In general, it should be 4 but it can change as per your CPU cores. We can provide `max_requests` param which specifies after how many requests the worker should restart. This might feel counter intuitive at first. Why do we need to restart workers continuously but it is needed to control memory leaks in application. `max_requests_jitter` specifies a random gap so that not all workers restart at same time bringing whole production down. `backlog` is the maximum number of connections that can be held before returning 5xx error to user. While we can handle 10-50 req/sec using development server, we can handle 1000-2000 req/sec using gunicorn with 4 workers.

There are numerous other parameters that you can tune to extract best performance out of your server.