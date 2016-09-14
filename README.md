# Dockerizing a Node.js web app

The goal of this example is to show you how to get a Node.js application into a Docker container. The guide is intended for development, and not for a production deployment. The guide also assumes you have a working Docker installation and a basic understanding of how a Node.js application is structured.

In the first part of this guide we will create a simple web application in Node.js, then we will build a Docker image for that application, and lastly we will run the image as a container.

Docker allows you to package an application with all of its dependencies into a standardized unit, called a container, for software development. A container is a stripped-to-basics version of a Linux operating system. An image is software you load into a container.

## Create the Node.js app

First, create a new directory where all the files would live. In this directory create a ```package.json``` file that describes your app and its dependencies:

```
{
  "name": "docker_web_app",
  "version": "1.0.0",
  "description": "Node.js on Docker",
  "author": "FX Martin <mail@fxmartin.me>",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.13.3"
  }
}
```

Then, create a ```server.js``` file that defines a web app using the [Express.js](http://expressjs.com/) framework:

```
'use strict';

const express = require('express');

// Constants
const PORT = 8080;

// App
const app = express();
app.get('/', function (req, res) {
  res.send('Hello world\n');
});

app.listen(PORT);
console.log('Running on http://localhost:' + PORT);
```

In the next steps, we'll look at how you can run this app inside a Docker container using the official Docker image. First, you'll need to build a Docker image of your app.

## Creating a Dockerfile

Create an empty file called ```Dockerfile```:

```
FROM node:argon

# Create app directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# Install app dependencies
COPY package.json /usr/src/app/
RUN npm install

# Bundle app source
COPY . /usr/src/app

EXPOSE 8080

CMD [ "npm", "start" ]
```

## Building your image

Go to the directory that has your ```Dockerfile``` and run the following command to build the Docker image. The ```-t```flag lets you tag your image so it's easier to find later using the docker images command:

```docker build -t <your username>/node-web-app .```

## Run the image

Running your image with -d runs the container in detached mode, leaving the container running in the background. The ```-p``` flag redirects a public port to a private port inside the container. Run the image you previously built:

```$ docker run -p 49160:8080 -d <your username>/node-web-app```

Print the output of your app:

```
# Get container ID
$ docker ps

# Print app output
$ docker logs <container id>

# Example
Running on http://localhost:8080
```

If you need to go inside the container you can use the exec command:

```
# Enter the container
$ docker exec -it <container id> /bin/bash
```

## Test

To test your app, get the port of your app that Docker mapped:

```
$ docker ps

# Example
ID            IMAGE                                COMMAND    ...   PORTS
ecce33b30ebf  <your username>/node-web-app:latest  npm start  ...   49160->8080
```

In the example above, Docker mapped the 8080 port inside of the container to the port 49160 on your machine.

Now you can call your app using curl (install if needed via: sudo apt-get install curl):

```
$ curl -i localhost:49160

HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 12
Date: Sun, 02 Jun 2013 03:53:22 GMT
Connection: keep-alive

Hello world
```

We hope this tutorial helped you get up and running a simple Node.js application on Docker.
