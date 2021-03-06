# REMUX - request demultiplexer
> Remux is an nginx reverse-proxy daemon designed for routing http requests to different docker service backends.

## Why?
When we started changing our technologies into microservices, lots of problems startd:

- Multi services requires multi (sub)domains (and multi SSL certs)
- Harder local development as we have to configure one port for each service and somehow link them together.
- CORS. allowing `*` is unsecure while using specefic domain names requires many config changes in different dev envs.
  or even imagine what if a UI developer whants to simply use staging api server and just focus on developing UI ? :))
- Service discovery problmes with solutins like [nginx-proxy](https://github.com/jwilder/nginx-proxy) for docker SWARM mode ([#520](https://github.com/jwilder/nginx-proxy/issues/520)).
- Typically we don't want to configure or expose our real service IPs.

## The solution
This image is just a simple pre-configured nginx proxy that can be customized using environment variables and routes requests to where they sh
ould go. (It is recommanded using REMUX behind nginx-proxy)

```
+------------+                                   +--------------------------------+
|            |                                   |                                |
|            +-----/gallery/photos-------------->+  public service                |
|            |                                   |                                |
|            |                                   |                       WWW Node |
|            |                                   |                                |
|            |                                   |                                |
| REMUX      +-----/admin/dashboard------------->+  admin service                 |
| Daemon     |                                   |                                |                       
|            |                                   +--------------------------------+
|            |                                   +--------------------------------+
|            |                                   |                                |
|            +-----/api/v1/users---------------->+  api service          API Node |
|            |                                   |                                |
|            |                                   +--------------------------------+
|            |                                   +--------------------------------+
|            |                                   |                                |
|            +-----/cdn/photos/img1.jpg--------->+  cdn service          CDN Node |
|Manager Node|                                   |                                |
+------------+                                   +--------------------------------+
```

## Docker image
Pre built docker image is published at `baninan/remux`.

## docker-compose example
```yaml
version: '3'
services:

 remux:
    image: banian/remux
    deploy:
           placement:
                constraints:
                    - node.hostname==YOUR_HOST_NAME
    environment:
      - VIRTUAL_HOST=mysubdomain.local
      - VIRTUAL_PORT=80
      - API_URL=myservice_api:3000
      - WEB_URL=myservice_web:3001
      - ADMIN_URL=myservice_admin:3002
      - CDN_URL=myservice_cdn:9000

 api:
    image: API_IMAGE_NAME

 web:
    image: WEB_IMAGE_NAME

 admin:
    image: ADMIN_IMAGE_NAME
 
 cdn:
    image: CDN_IMAGE_NAME
```

Make sure to replace the ```API_IMAGE_NAME```, ```WEB_IMAGE_NAME```, ```ADMIN_IMAGE_NAME``` and ```CDN_IMAGE_NAME``` with your own service image names.
## Environment variables & routes
You don't have to configure all of variables, they will be replaced by next one if not set.

Route    |  Variable usage priority
---------|----------------------------------------------
/api     | `API_URL`, `WEB_URL`, `ADMIN_URL`
/admin   | `ADMIN_URL`, `WEB_URL`, `API_URL`
/        | `WEB_URL`, `ADMIN_URL`, `API_URL`
/cdn     | `CDN_URL`, `WEB_URL`, `ADMIN_URL`, `API_URL`

## FAQ
**Can i change hardcoded /api,... to my custom one?**

NO!

## Funny story
When i was commiting initial README file, it was accidentally deleted! Thanks to linux super power i easily recovered it using:
`pv /dev/sda | grep -a -C 500 '/admin/dashboard'` :))

## LICENSE
MIT
