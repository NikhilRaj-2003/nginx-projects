🐳 NGINX Reverse Proxy with Docker Compose — Go and Python Services
===================================================================

Introduction
============

This project sets up a **reverse proxy using NGINX** in Docker to route traffic to two backend services:
- A **Golang** service on `/service1`
- A **Python Flask** service on `/service2`

All services run in Docker containers, orchestrated via **Docker Compose**. NGINX routes all requests via a single port (`8080`) based on path prefixes.

Prerequisites
-------------

*   Docker
*   Docker Compose
*   Git

Steps for Building the Project:
-------------------------------

**step 1: install Docker and Clone the repo into your workspace**

1.  Firstly install dockers , i am using ubuntu so i will be using (_APT)_ as my package manager.

```
sudo apt install docker -y
```

2. Then clone the github repository into your local workspace.And also change the directory to your repo.

```
`git clone https://github.com/NikhilRaj-2003/nginx-projects.git
```
```
cd nginx-projects
```

**step 2: Create 2 Dockerfile for Python and Golang**

1.  Golang (Dockerfile)

```

# Build stage
FROM golang:1.20-alpine AS builder
WORKDIR /app
COPY . .
RUN go mod tidy
RUN go build -o service1 main.go
# Final image
FROM alpine
COPY --from=builder /app/service1 /service1
EXPOSE 8081
CMD ["/service1"]
```

2. Python (Dockerfile)

```

FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
EXPOSE 8082
CMD ["python", "app.py"]
```

**step 3 : Under nginx directory edit the nginx.config file**

1.  open the nginx.config file and paste the required context.

```

events {}
http {
    server {
        listen 8080;
        location /service1/ {
            proxy_pass http://service1:8081/;
        }
        location /service2/ {
            proxy_pass http://service2:8082/;
        }
    }
}
```

2. Create a Dockerfile to create a nginx image .

```

FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
```

3. Then create a docker-compose file (docker-compose.yml)

```

version: '3'
services:
  service1:
    build: ./service-1
    container_name: service1
    ports:
      - "8081:8081"
  service2:
    build: ./service-2
    container_name: service2
    ports:
      - "8082:8082"
  nginx:
    build: ./nginx
    container_name: nginx-reverse-proxy
    ports:
      - "8080:8080"
    depends_on:
      - service1
      - service2

```

4. Later use the docker-compose command to **build all images** and to **start all containers .**

```
docker-compose up --build
```

then you get a output like this after running the command .

**Step 4 : Test the Endpoints manually**
1. Take the EC2 instance IP-Address and try to access using the port number (8080) from your browser .

```
http://<EC2_PUBLIC_IP>:8080/service1
http://<EC2_PUBLIC_IP>:8080/service2
```

2. Then whenever you give service1 after the port number has shown in above it provides the output from service1 .

![output of service1](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*cvuN7JLIzzEk-r2H8iJ6Gw.png)

3. Same goes for service-2 also whenever you enter service2 after the port number as shown , it provides the output from service2.

![output of service2](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*CnG8WPCoF5sTfVaql31-MA.png)

Conclusion
----------

This project involved building and deploying two microservices (Go and Python) with Docker Compose and routing them through an NGINX reverse proxy on a single port (`8080`). Hosted on an AWS EC2 instance, the setup demonstrated containerization, service orchestration, and basic cloud deployment
