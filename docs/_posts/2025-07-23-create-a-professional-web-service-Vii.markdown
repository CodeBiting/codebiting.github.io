---
layout: post
title:  "Create a professional web service - Part VII - Dockerization"
author: Jordi Dalmau Heras
date:   2025-07-23 16:25:00 +0200
categories: development
---
Learn how to package an application and its dependencies into a standardized container called a “Docker container.”

## Containerizing the Service

To deploy using Docker Compose, Swarm, Kubernetes or Cloud Run, we first need to package the application inside a container.

When containerizing, it's recommended to follow [Docker's official best practices](https://docs.docker.com/build/building/best-practices/):

* Ensure the final image is small and fast to load:
  * Use **multi-stage builds**
  * Choose a **trusted base image**
  * Select the **smallest image possible** for portability, fast startup, and minimal attack surface
  * Avoid installing unnecessary packages
* Rebuild images regularly to **update dependencies** and reduce vulnerabilities
* Exclude irrelevant files from the build context
* Create **ephemeral containers** that can be stopped, removed, and recreated with minimal configuration
* **Decouple applications** into multiple containers — each with a single responsibility
* Leverage **build cache** to speed up builds and reuse common layers
* Pin image versions using **immutable digests (hashes)** to avoid breaking changes
* **Test images** as part of the CI (Continuous Integration) pipeline

---

## Creating the `Dockerfile`

```dockerfile
FROM        # Specify the base image
LABEL       # Metadata for organization and licensing
RUN         # Install dependencies into the image
CMD         # Run the main software inside the image
EXPOSE      # Define the exposed ports
ENV         # Set environment variables
ADD / COPY  # Copy folders and files into the container
ENTRYPOINT  # Main command of the image
VOLUME      # Declare mountable persistent storage
USER        # Specify non-root user for running the service
WORKDIR     # Set working directory inside the container
ONBUILD     # Define instructions for child builds
```

## Create the container locally and see if it works correctly

```bash
# Check which images we have locally
$ docker images

# Build a local image from the Dockerfile
# docker build --tag <new image name> <path to Dockerfile>
# Use a new version tag each time
$ docker build --tag codebiting/onion-cargo-loading:v1 .

# Verify that the image was created successfully by checking if it exists
$ docker images

# Run the image in interactive mode
$ docker run -it -p 8080:8080 -d codebiting/onion-cargo-loading:v1

# Check that the container is running with the correct ports, and review its logs:
$ docker ps
$ docker logs <container id>
$ docker inspect <container id>

# See all containers (running and exited)
$ docker ps -a

# Confirm that the application is working by opening the following URL in a browser:
[http://<my.servlet.host>:8080/api-documentation](http://localhost:8080/api-documentation/)

# Enter the running container (for debugging or inspection)
$ docker exec -it <container id> /bin/bash

# If the container is stopped, you can start and attach to it with:
$ docker start -ai <container id>

# Stop the container — two options:
$ docker stop <container id>
$ docker kill <container id>

# Run the container using Docker Compose
# Reference:
# https://docs.docker.com/compose/
# https://docs.docker.com/compose/production/
# https://www.educative.io/blog/docker-compose-tutorial
```
