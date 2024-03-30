# docker-challenge-template


## Challenge One

1. **Static Web Page Preparation**:
    - Within the `challenge1` directory, I created an `index.html` file that includes my name and student ID, which will be the static content served by the Docker container.
    
2. **Dockerfile Configuration**:
    -  I wrote a Dockerfile to define the setup of the Docker container. This file specifies the official Nginx image as the base, copies the `index.html` file to the correct location within the container, and exposes port 80 for HTTP requests.
	- ```
```
FROM nginx
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

1. **Docker Image Building**:
    - Using the command line from within directory, I ran the `docker build -t my-static-site .` command, which creates a Docker image named `my-static-site` based on the instructions in the Dockerfile.
    ![[Screenshot 2024-03-25 at 9.13.38 PM.png]]
4. **Docker Container Launching**:
    - After building the image, I started a Docker container with the command `docker run --name my-static-website -d -p 8080:80 my-static-site`. This command assigns the container a name, runs it in detached mode, and maps port 8080 on my local machine to port 80 on the container.
    
5. **Web Server Testing**:
    - To verify the container was serving the web page correctly, I opened my web browser and navigated to `http://localhost:8080`. The `index.html` page loaded, confirming that Nginx was correctly serving the static content.
    -![[Screenshot 2024-03-25 at 9.46.11 PM.png]]
## Challenge Two 
1. **Dockerfile Creation:**
	- Within the `challenge2` directory, I crafted a Dockerfile. The Dockerfile starts with specifying the Node.js image and proceeds with instructions to set up the application environment.
	- ```
```
FROM node:latest

WORKDIR /usr/src/app

COPY package*.json ./ 

RUN npm install 

COPY . .

ENV PORT=8080

EXPOSE 8080  

CMD [ "node", "server.js" ]
```
    
2. **Nginx Configuration for Reverse Proxy:** 
	- An `nginx.conf` file was created to configure Nginx as a reverse proxy. The significance of this configuration is to enable Nginx to handle client requests and pass them to the Node.js application container, thus enhancing security and performance.

```
server {

listen 80;

location / {

proxy_pass http://app:3000/api/;

proxy_http_version 1.1;

proxy_set_header Upgrade $http_upgrade;

proxy_set_header Connection 'upgrade';

proxy_set_header Host $host;

proxy_cache_bypass $http_upgrade;
}
}
```
    This setup ensures that all traffic coming to the Nginx server on port 80 is correctly forwarded to the Node.js application running on port 3000.
    
3. **Orchestrating Services with Docker Compose:**
	- I proceeded to define the services in a `docker-compose.yml` file. This file orchestrates the Nginx reverse proxy service and the Node.js application service, linking them together and exposing the Nginx service on port 8080 to the host.
```
version: '3.8'

services:

nginx:

image: nginx:latest

ports:

- '8080:80'

volumes:

- ./nginx.conf:/etc/nginx/conf.d/default.conf

depends_on:

- api

api:

build: .

ports:

- '3000:3000'
```
	
    
4. **Launching the Services:** 
	- Using the command `docker-compose up --build`, I initiated the build process for the Docker image and subsequently started the Nginx and Node.js services. This command is pivotal as it compiles the application and makes it accessible through Nginx.
    ![[Screenshot 2024-03-26 at 3.34.50 PM.png]]
5. **Testing the Dynamic Application:** 
	- To verify the application's functionality, I uses `http://localhost:8080/api/books` and `http://localhost:8080/api/books/1` from a web browser. The JSON responses confirmed the application was dynamically generating content and the reverse proxy was correctly routing the requests
	![[Pasted image 20240330131736.png]]