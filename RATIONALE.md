# Rationale

## Base image choice

The final image needs to serve the built website files to a browser. It does not need Node.js or the tools used to build the files after the build is finished. The multi-stage Dockerfile uses `node:24` for the build stage and `nginx:alpine` for the final stage. Nginx is a web server that can serve the finished static files, and the Alpine-based image is smaller than using the full Node image to run the website. This makes `nginx:alpine` a suitable choice for the final image. [1][3]

A multi-stage build separates the work of building the application from the work of running it. In the first stage, Node.js and npm are used to create the `build` folder. In the final stage, Docker copies only that folder into the Nginx image. The build tools and other files from the first stage are not copied into the final image. This is why the multi-stage image is much smaller than the single-stage image. [1]

| Image | Disk usage | Content size |
|---|---|---|
| Multi-stage (`nginx:alpine`) | 93.6 MB | 26.3 MB |
| Single-stage (`node:24`) | 1.65 GB | 413 MB |

These results show that the single-stage image is about 17.6 times larger than the multi-stage image by disk usage. The disk usage includes the image layers pulled to the computer, while content size shows the amount contributed by the image itself. The exact numbers can change with Docker versions and updated image layers, but the comparison shows the same result: the single-stage image is much larger. [6]

A smaller image is useful because it takes less space to store and less data to transfer when it is downloaded or deployed. It can also be faster to pull and gives an attacker fewer installed tools and packages to target. Keeping only what is needed to serve the website makes the final image simpler. [5]

## Service split

The `web` and `cache` services have different jobs, so it makes sense to keep them separate. The `web` service builds from the Dockerfile and serves the website through Nginx. The `cache` service uses the public `redis:7-alpine` image to provide a Redis cache. [4] Keeping Redis separate means the project can use a ready-made cache service instead of installing and managing it inside the web container. It also makes the services easier to configure, inspect, restart, or replace separately.

This service split connects to the Week 3 idea that containers sit between Infrastructure as a Service (IaaS) and Platform as a Service (PaaS). [7] Containers provide control over how the application and its services are packaged and run, while pre-built images such as Redis save the work of creating and maintaining every component ourselves. In this project, the web image is built from our Dockerfile, while Redis is accepted as a pre-built dependency. This is a practical balance between control and convenience.

In `compose.yaml`, `depends_on` is set so that `web` waits for `cache` to become healthy before starting. The Redis healthcheck runs `redis-cli ping`; a successful response shows that Redis is responding. The condition `service_healthy` is important because simply starting a container does not prove that the service inside it is ready. Docker Compose can use the healthcheck result to control the startup order. [2]

In T05, stopping the `cache` service did not stop the `web` service, and the website still returned HTTP 200. This happened because the current web page is served as static content by Nginx and does not make a runtime request to Redis. The `depends_on` setting controls startup order; it does not automatically stop `web` or check the cache continuously after startup. In a real application that uses Redis for requests, the application would also need to handle cache connection failures. [2]

## References

1. Docker Docs, "Multi-stage builds." https://docs.docker.com/build/building/multi-stage/
2. Docker Docs, "Control startup and shutdown order in Compose." https://docs.docker.com/compose/how-tos/startup-order/
3. Docker Hub, "nginx" official image. https://hub.docker.com/_/nginx
4. Docker Hub, "redis" official image. https://hub.docker.com/_/redis
5. Docker Docs, "Building best practices." https://docs.docker.com/build/building/best-practices/
6. Course handout, "T02: Write a Dockerfile (Week 3)."
7. Course lecture, Week 3: service models (IaaS, PaaS) and containers.
