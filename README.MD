Experimenting with Docker Build
====

builds multiple docker images
```shell
docker compose up -d
```

Spring webapp can be started from the source, demo_webapp_service must be stopped
```shell
docker compose stop demo_webapp_service
```

Spring webapp project can be run outside the Docker (using Mongo localhost:27017 running in docker)
```shell
cd webapp
./mvnw spring-boot:run
```

Spring webapp project can be conteinarized to run in docker and use internal network
```shell
cd webapp
docker build -t standalone-webapp .
docker run --network demo-infrastructure_demo-network --publish 8080:8080 -e MONGO_CACHE_URL='mongodb://demo_mongo_cache_service:27017' --name my-standalone-webapp standalone-webapp
```

Spring webapp can be started from IntelliJ runner in docker-compose mode:
* create new run configuration
* select from docker-compose
* select parent docker compose file (select web service)
* environment values will be picked from compose file

Spring webapp can be started from IntelliJ runner in docker mode:
* create new run configuration
* select from docker
* pull openjdk:24-jdk
* run options
* ```--network demo-infrastructure_demo-network --publish 8080:8080 -e MONGO_CACHE_URL=mongodb://demo_mongo_cache_service:27017 --name my-standalone-webapp --rm```

REPACKAGING
-----
with jlink a smaller java runtime can be created - either by build plugin 
(unpackage after packaging) or by other means) jre module dependencies should be 
identified before creating a new linked jre
```dockerfile
graal_optimised/DockerfileRepackedJava
```

GRAAL
-----
Application can run on a GRAAL VM either directly, or, with the help of graal vm, conveted to a native binary

* this gives more options to create a custom docker image (plugin needed - to do a compilation -Pnative native:compile)
Alpine does not work with glibc, so, if we wanted a smaller image, glib compatability must be installed
```dockerfile
graal_optimised/Dockerfile.native
```

* easier to use - plugin that uses Docker and creates a cloud native image
```shell
cd graal_optimised
./mvnw -Pnative spring-boot:build-image  
```

* see https://paketo.io/ for other options (pack tool)

* code reload does not work out of the box.... graal_optimised dockerfiles creates a built app (even in native format)
solution - run configuration either based on Graal or generic java, dockerised and attached to `demo-infrastructure_demo-network`