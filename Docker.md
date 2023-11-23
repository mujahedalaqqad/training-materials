# Introduction

Docker is a platform for building and running applications. It allows you to package your application along with all of its dependencies together as a single executable, called container image, with this you don't need to rely on what's installed on the host.

a Container is a running instance of an image, You can manage your containers using the docker CLI or API

# Docker Architecture

Docker follows a client-server architecture. The docker client makes REST API calls to the docker daemon (backend), the docker daemon is responsible for building and running your applications.

![1700746228518](image/Docker-Intro/1700746228518.png)


# Docker Registry

A docker registry is a place to store docker images, for example, the docker hub is a public docker registry service that allows you to store your images and pull public images.

When you use the **docker pull** or **docker run** commands, Docker pulls the required images from your configured registry. When you use the **docker push** command, Docker pushes your image to your configured registry.


# Containerizing an application

We will be going through the steps required to containerize an application, but make sure you get the [Docker Desktop](https://docs.docker.com/get-docker/) installed:

* Let's assume that our application structure looks like the following:

```

   ├── my-app/
   │ ├── pom.xml
   │ ├── src/

```

* To containerize the application and build an image, we will need to create a Dockerfile, so create a file named Dockerfile in the root of your project:

```

   ├── my-app/
   │ ├── .mvn/
   │ ├── pom.xml
   │ ├── Dockerfile
   │ ├── src/
   │ ├── mvnw

```

* Next you will need to add a line to your Dockerfile to tell it what image you want to use for your application, this image will be used as the base for the app image that we will be building, in our case we will use the Eclipse JDK distribution image

```

FROM eclipse-temurin:17-jdk

```

* Set the current directory to the app directory, this will make things easier when the rest of the commands inside the Dockerfile gets executed, it will instruct docker to use this directory as the default location for the rest of the commands:

```

WORKDIR /app

```

* We need to copy the mvn wrapper (Java-specific) so we can install the dependencies inside the image:

```

COPY .mvn/ .mvn
COPY . /app

```

* Now we need to install the application dependencies:

```

RUN ./mvnw clean install

```

* All we have to do now is to tell docker which command it can use to run our application:

```

CMD ["java","-jar","app/target/*.jar"]

```

The Dockerfile will look like this:

```
FROM eclipse-temurin:17-jdk

WORKDIR /app

COPY .mvn/ .mvn
COPY . /app
RUN ./mvnw clean install

CMD ["java","-jar","app/target/*.jar"]

```

* Now, let's build our image:

```

$ docker build --tag my-app:v1.0.0 .

Step 1/7 : FROM eclipse-temurin:17-jdk
Step 2/7 : WORKDIR /app
...
Successfully built a0bb458aabd0
Successfully tagged my-app:v1.0.0

```

* Now, let's build our image:

```

$ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED          SIZE
my-app            v1.0.0              a2d3f58s60l1        10 minutes ago   267MB

```

* Check for the list of local images on your machine:

```

$ docker images

REPOSITORY          TAG                 IMAGE ID            CREATED          SIZE
my-app            v1.0.0              a2d3f58s60l1        10 minutes ago   267MB

```

* Now, let's run the image, we use the **publish** flag here to expose the port 8000 from inside the container to the host, and the --detach flag to run the container in the background, the --restart flag is used to make sure that the container is started automatically on a machine startup or if the container exits with a failure:

```

$ docker run my-app:v1.0.0 --detach --restart on-failure --publish 8000:8000

```


# Docker Compose

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application's services. Then, with a single command, you create and start all the services from your configuration.

* Let's create a compose.yaml file for our project:

```

   ├── my-app/
   │ ├── .mvn/
   │ ├── pom.xml
   │ ├── Dockerfile
   │ ├── compose.yaml
   │ ├── src/
   │ ├── mvnw

```

* Now, let's define two services in our compose file, here we define two service, myapp and database:

*Note: the database will be accessible from within the myapp container using the hostname **database***

```

services:
  myapp:
    build: .
    ports:
      - "8000:8000"
    environment:
      - POSTGRES_NAME=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    depends_on:
      - database
  database:
    image: postgres
    volumes:
      - ./data/db:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres


```

* Now, let's build and run the services:

```

$ docker compose up

```
