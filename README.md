# Node Sample App with Dockerfile and docker-compose

## Description

This app is intended for use with the Sparta Global Devops Stream as a sample app. You can clone the repo and use it as is but no changes will be accepted on this branch.

To use the repo within your course you should fork it.

The app is a node app with three pages.

### Homepage

``localhost:3000``

Displays a simple homepage displaying a Sparta logo and message. This page should return a 200 response.

### Blog

``localhost:3000/posts``

This page displays a logo and 100 randomly generated blog posts. The posts are generated during the seeding step.

This page and the seeding is only accessible when a database is available and the DB_HOST environment variable has been set with it's location.

## Usage

Clone the app

```
npm install
npm start
```

You can then access the app on port 3000 at one of the urls given above.

## Tests

There is a basic test framework available that uses the Mocha/Chai framework

```
npm test
```

The test for posts will fail ( as expected ) if the database has not been correctly setup.

## Docker Usage

### Rquirements

Make sure you have the following installed in order for this app to work

- Docker
- Docker-compose

## Method to create a single docker container

- Create Dockerfile in the app directory
```
touch Dockerfile
```
- Open the Dockerfile and copy the following into the file. Go down the page to see the explanation for what each of the line here in the docker file does
```
FROM node:8
WORKDIR /app
COPY package.json /app
RUN npm install
COPY . /app
CMD node app.js
```
- run the Dockerfile using the following command to create a docker image. -t is tag to name the image. DON'T FORGET THE DOT AT THE END
```
docker build -t app .
```
- you can check for the docker image using the following command
```
docker image ls
```

- build the docker container using the docker image that was built. Use -p to map the port from the container to your host machine. Node-sample-app uses port 3000. You can use any available port to map it your machine. In this case, port 80 is used
```
docker run -p 80:3000 app
```
- Now go to the browser and check
```
localhost:80
```

## Method to create multiple docker containers

- You need docker-compose installed in order for it to work. Create docker-compose file in the app directory
```
touch docker-compose.yml
```
- Copy the following into the docker-compose file. This will create a docker container for the mongodb and link it to the app container that was created earlier.
```
version: "3"
services:
  app:
    container_name: app
    restart: always
    build: .
    environment:
      - DB_HOST=mongodb://mongo:27017/posts
    ports:
      - "3000:3000"
    links:
      - mongo
  mongo:
    container_name: mongo
    image: mongo
    volumes:
      - ./data:/data/db
    ports:
      - "27017:27017"
```
- Breaking this down, what we are doing here is
  - defining a service called app, and naming the container
  ```
  version: "3"
  services:
    app:
      container_name: app
  ```
  - instructing Docker to restart the container automatically if it fails,
  ```
      restart: always
  ```
  - building the app image using the Dockerfile in the current directory
  ```
      build: .
  ```
  - You can set environment variables in a service’s containers with the ‘environment’ key,
  ```
      environment:
        - DB_HOST=mongodb://mongo:27017/posts
  ```
  - mapping the host port to the container port.
  ```
      ports:
        - "3000:3000"
  ```
  - links the app service to the mongo service
  ```
      links:
        - mongo
  ```
  - We then add another service called mongo but this time instead of building our own mongo image, we simply pull down the standardmongo image from the Docker Hub registry.
  ```
      mongo:
        container_name: mongo
        image: mongo
    ```
  - mapping the host port to the container port.
    ```
      ports:
        - "27017:27017"
    ```
- To get the container up and running use the following command
```
docker-compose up
```

- Go to the browser and check
```
localhost:3000
```
- To see posts page
```
localhost:3000/posts
```

## Code Explanation

### Dockerfile
- The first thing we need to do is define from what image we want to build from. Here we will use the latest LTS (long term support) version 8 of node available from the Docker Hub:
```
FROM node:8
```
- Next we create a directory to hold the application code inside the image, this will be the working directory for your application:
```
WORKDIR /app
```
- This image comes with Node.js and NPM already installed so the next thing we need to do is to install your app dependencies using the npm binary. Note that, rather than copying the entire working directory, we are only copying the package.json file.
```
COPY package.json /app
```
```
RUN npm install
```
- To bundle your app's source code inside the Docker image, use the COPY instruction:
```
COPY . /app
```
- Last but not least, define the command to run your app using CMD which defines your runtime.
```
CMD node app.js
```
