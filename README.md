# docker-challenge-template


## Challenge One

1. **Static Web Page Preparation**:
    - Within the `challenge1` directory, I created an `index.html` file that includes my name and student ID, which will be the static content served by the Docker container.
    
2. **Dockerfile Configuration**:
    -  I wrote a Dockerfile to define the setup of the Docker container. This file specifies the official Nginx image as the base, copies the `index.html` file to the correct location within the container, and exposes port 80 for HTTP requests.

```
FROM nginx
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

1. **Docker Image Building**:
    - Using the command line from within directory, I ran the `docker build -t my-static-site .` command, which creates a Docker image named `my-static-site` based on the instructions in the Dockerfile.
<img width="1512" alt="Screenshot 2024-03-25 at 9 13 38 PM" src="https://github.com/SamuelKebert/docker-challenge-template/assets/118232574/ff5973c0-910b-435e-8683-7ef265b16f26">

4. **Docker Container Launching**:
    - After building the image, I started a Docker container with the command `docker run --name my-static-website -d -p 8080:80 my-static-site`. This command assigns the container a name, runs it in detached mode, and maps port 8080 on my local machine to port 80 on the container.
    
5. **Web Server Testing**:
    - To verify the container was serving the web page correctly, I opened my web browser and navigated to `http://localhost:8080`. The `index.html` page loaded, confirming that Nginx was correctly serving the static content.
<img width="762" alt="Screenshot 2024-03-25 at 9 46 11 PM" src="https://github.com/SamuelKebert/docker-challenge-template/assets/118232574/e107e491-f571-4323-af60-fb271d495083">

## Challenge Two 
1. **Dockerfile Creation:**
	- Within the `challenge2` directory, I crafted a Dockerfile. The Dockerfile starts with specifying the Node.js image and proceeds with instructions to set up the application environment.

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
   <img width="971" alt="Screenshot 2024-03-26 at 3 34 50 PM" src="https://github.com/SamuelKebert/docker-challenge-template/assets/118232574/dceb7b12-de98-4e0e-b9e7-138c0b310c94">

5. **Testing the Dynamic Application:** 
	- To verify the application's functionality, I uses `http://localhost:8080/api/books` and `http://localhost:8080/api/books/1` from a web browser. The JSON responses confirmed the application was dynamically generating content and the reverse proxy was correctly routing the requests
	<img width="744" alt="Screenshot 2024-03-30 at 2 41 32 PM" src="https://github.com/SamuelKebert/docker-challenge-template/assets/118232574/05e188de-555a-45ab-925f-450f64616c11">

## Challenge Three

1. **Set ENV File:**
	- Create a .env file in the root directory of your project. This file will store all the environment variables your containers need for the docker-compose.
```
# Database Configuration for MariaDB
MYSQL_ROOT_PASSWORD=
MYSQL_DATABASE=
MYSQL_USER=
MYSQL_PASSWORD=
MYSQL_HOST=

# Application Database Configuration
DB_ROOT_PASSWORD=
DB_DATABASE=
DB_USERNAME=         
DB_PASSWORD=
DB_HOST=

```
 
1. **Set Docker-Compose File:**
	- Create and configure a docker-compose.yml file to define and link the services: nginx for web serving, node-service for the application backend, and db for the MariaDB database. These services were interconnected through a shared Docker network, facilitating seamless communication and data exchange.
```
version: '3.8'

services:
  nginx:
    build: ./nginx
    ports:
      - "8080:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - node-service

  node-service:
    build: ./api
    environment:
      - "DB_ROOT_PASSWORD=${DB_ROOT_PASSWORD}"
      - "DB_DATABASE=${DB_DATABASE}"
      - "DB_USERNAME=${DB_USERNAME}"
      - "DB_PASSWORD=${DB_PASSWORD}"
      - "DB_HOST=${DB_HOST}"
    depends_on:
      - db

  db:
    build: ./db
    environment:
      - "MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}"
      - "MYSQL_DATABASE=${MYSQL_DATABASE}"
      - "MYSQL_USER=${MYSQL_USER}"
      - "MYSQL_PASSWORD=${MYSQL_PASSWORD}"
      - "MYSQL_HOST=${MYSQL_HOST}"
    ports:
      - "3307:3306"
    volumes:
      - db-data:/var/lib/mysql
    command: --default-authentication-plugin=mysql_native_password

volumes:
  db-data:
```

3. **Adjuct Dockerfiles:**
	-Refine the Dockerfiles in both the /api and /db directories to ensure essential configurations and dependencies are correctly incorporated into their respective Docker images. In the /api directory's Dockerfile, adjustments were made to include all necessary Node.js dependencies by copying the package.json and associated files. This setup guarantees that the application has all required modules when the container starts. Similarly, the Dockerfile for the database service in the /db directory was modified to correctly path database initialization scripts and configurations, ensuring MariaDB is properly configured upon launch. 
4. **Testing:**
	- Access the application by navigating to http://localhost:8080/api/books and http://localhost:8080/api/books/1 in a web browser. This URL is directed through the Nginx server, which is configured to proxy requests to the Node.js application running on a Docker-managed network.
<img width="1512" alt="Screenshot 2024-04-23 at 10 16 46 AM" src="https://github.com/SamuelKebert/docker-challenge-template/assets/118232574/639be44a-ab5e-401b-a886-c00a63bcac02">

<img width="1512" alt="Screenshot 2024-04-23 at 10 21 58 AM" src="https://github.com/SamuelKebert/docker-challenge-template/assets/118232574/5b1c4499-a436-474a-83a2-2f05712f199d">


## Challenge Four 
1. **Scaling Preparation:**
	-Prepare node-service for horizontal scaling by configuring it to run multiple instances.Prepare node-service for horizontal scaling by configuring it to run multiple instances.
```
services:
  node-service:
    build: ./api
    environment:
      - "DB_ROOT_PASSWORD=${DB_ROOT_PASSWORD}"
      - "DB_DATABASE=${DB_DATABASE}"
      - "DB_USERNAME=${DB_USERNAME}"
      - "DB_PASSWORD=${DB_PASSWORD}"
      - "DB_HOST=${DB_HOST}"
    deploy: 
      replicas: 3 
```

3. **Ngnix Configuration:**
	-Configure Nginx to distribute incoming traffic evenly across multiple instances of node-service. Modify the nginx.conf to define an upstream server block and update the server configuration to proxy requests to this upstream
```
events {
    worker_connections  1024;
}

http {
    resolver 127.0.0.11 valid=5s;  # Docker's internal DNS server for service discovery

    upstream loadbalancer {
        server node-service:8080;  # Points to the Node.js service
    }

    server {
        listen 80;
        server_name localhost;

        location / {
            root /usr/share/nginx/html;  # Serve static files from here
            index index.html index.htm;
            try_files $uri $uri/ =404;  # Attempt to serve file directly or return 404
        }

        location /api {
            proxy_pass http://loadbalancer;  # Proxy API requests to Node.js
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

5. **Testing:** 
	-Use the Docker Compose command to launch multiple instances of your application. By executing the command below, you will initiate three instances of the node-service
```
docker-compose up --scale node-service=3 -d
```
<img width="583" alt="Screenshot 2024-04-24 at 5 42 24 PM" src="https://github.com/SamuelKebert/docker-challenge-template/assets/118232574/58481137-cbcb-4194-ac51-5e5477e568dd">
	- : Perform multiple HTTP GET requests to the endpoint by executing
```
 http://localhost:8080/api/stats
```
	- This endpoint returns the hostname or container ID, providing a clear indication of which node-service instance processed each request.
 
<img width="1512" alt="Screenshot 2024-04-23 at 2 55 16 PM" src="https://github.com/SamuelKebert/docker-challenge-template/assets/118232574/9b8b414a-361f-4c73-aebb-5a872e76fe2c">


