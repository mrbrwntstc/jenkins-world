Docker for Development
----------------------

### Docker Compose Basics

Best of both worlds
- not too low level to need to use the docker cli
- not too high level to where there's no more customization

Docker Compose lives and dies by YAML

docker-compose up 
docker-compose ps

#### Runtime vs Buildtime
build time: Dockerfiles/"docker pull"
run time: Running contents inside Docker image
  - any options passed in run time overrides anything in build time
  - can make any changes in real time (don't need to re-run the docker-compose.yaml file)

#### External Dependencies
```
services:
  redis:
    image: redis:alpine
    container_name: redis
    ports:
      - "6379"
    networks:
      - back-tier
```

```
def get_redis():
  if not hasattr(g, 'redis'):
    print("redis not available")
```

*Containers are meant to be ephemeral*

#### Data volumes
```
services:
  db:
    image: postgre:9.4
    container_name: db
    volumes:
      - "db-data:/var/lib/posgresql/data"
    networks: 
      - back-tier
```

-- -- -- -- -- -- -- --

### Optimizing Images

*Dockerfiles and container images are the heart of your application*

Dockerfiles
  - list of instructions outlining packages, files, dependencies, and everything required for your application to be running

Example Dockerfile:
```
FROM node:10.9-alpine

RUN mkdir -p /app
WORKDIR /app

RUN npm install -g nodemon
RUN npm config set registry https://registry.npmjs.org
COPY package.json /app/package.json
RUN npm install \
  && npm ls \
  && npm cache clean --force \
  && mv /app/node_modules /node_modules
COPY . /app

ENV PORT 80
EXPORT 80

CMD ["node", "server.js"]
```

#### Caching
If nothing changes in a layer command, Docker will use an existing version (cache hit)

Drastically reduces build time

- Copy over package or dependency manifests and instal those before copying the rest of the code
- Lockfiles change infrequently, so it's not necessary to reinstall everything all the time

Reducing the number of of layers will reduce the overall size of your image, even if the output is the same

Chain
```
RUN npm install \
  && npm ls \
  && npm cache clean --force \
  && mv /app/node_modules /node_modules
```

Sequential
```
RUN npm install
RUN npm ls 
RUN npm cache clean --force 
RUN mv /app/node_modules /node_modules
```

*Chain is 10MB smaller than Sequential*

**DockerHub compresses its images, so the size seen on docker.io is not the actual size on disk**

#### Avoid anti-patterns
- simultaneous innovation
- skipping the fundamentals like giving love to your Dockerfiles
- forgetting over development best practices 
  - pinning dependencies to versions
  - don't use the 'latest' tag

#### Laura's Model for Super Duper Container Success
Get it running on a container. Just create an application template and get it running. Then optimize the Dockerfile. Then worry about everything else

-- -- -- -- -- -- -- --

### Debugging in Containers
- viewing log output
  - logs are visible by default when starting the app with Docker Compose
  - ``` docker-compose logs ```
  - ``` docker-compose logs -t <docker service name> ```

- start container in interactive mode
  - ``` docker run -it ubuntu:10:04 /bin/bash ```
  - ```
    services:
      web:
        image: ubuntu:10:04
        tty: true # allows interactive shell
        stdin_open: true
        entrypoint: /bin/bash
    ```

- mount a volume if you need to with -v

docker top
  - list of all running docker services
docker inspect
  - give **all** details about a given resource 
  - \-\-format to pass in a Go template