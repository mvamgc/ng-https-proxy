HTTPS proxy
===========

Suppose the host IP is 10.0.2.15.

App Server container
--------------------

```
docker run -tdi -p 3000:3000 my-nodejs-app
docker ps
```
Note, 3000 exposed to host: 10.0.2.15:3000
This address and port is available from other docker containers as well.


nginx
-----

Run original image
```
mva@docker:~/nginx$ docker run -tdi nginx
9ed96ed69673674083278bc088adb12936eaa86e8d3b292b607ac18688f17009
mva@docker:~/nginx$ docker exec -ti 9e bash
root@9ed96ed69673:/# 
```
note: 9e - container id

Server definition is in /etc/nginx/conf.d/default.conf
Edit root location (comment root and index entries):
```
    location / {
        # root   /usr/share/nginx/html;
        # index  index.html index.htm;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://10.0.2.15:3000;
        client_max_body_size 10M;
    }
```

Now:
```
docker build -t ng1 .
docker run -ti -p 80:80 ng1
```

HTTP works.


HTTPS
-----

Generate certificate:
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx.key -out nginx.crt
```

Add to the server section (keep listen 80, if you like):
```
    listen              443 ssl;
    ssl_certificate     /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;
```

now rebuild and restart

```
docker build -t ng1 .
docker run -ti -p 80:80 -p 443:443 ng1
```
