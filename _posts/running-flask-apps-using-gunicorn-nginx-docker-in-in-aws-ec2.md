
# Running Flask Apps using Gunicorn+ +NGINX+Docker in in AWS EC2

Overview

[Flask](http://flask.pocoo.org/) is a web framework for python that provides a simple interface for dynamically generating responses to web requests.

[Docker](https://docs.docker.com/get-started/) is an open-source application that allows administrators to create, manage, deploy, and replicate applications using containers.

The purpose of this article is to provide step-by-step instructions for running a FLASK app integrated with gunicorn and NGINX running inside a single container hosted in AWS EC2 .

### **Components**

* Flask — Python based web server backend

* Gunicorn — Python WSGI HTTP Server for web applications.

* NGINX —HTTP cache,load balancer, and reverse proxy server.

* Docker — Tool designed to make it easier to create, deploy, and run applications by using containers

* AWS EC2 — Linux machine in AWS to host the docker

## Steps

1. **Update local packages in EC2**

    sudo yum update -y

2. **Install Docker in EC2**

    **
    **sudo amazon-linux-extras install docker
    sudo service docker start
    sudo usermod -a -G docker ec2-user

Re-Login and verify the docker installation without using sudo

    docker info

**3. Directory Setup**

Create a directory for flask app and config files

    mkdir flask_docker
    cd flask_docker

**4. Create a simple Flask App**

Simple flask application with default route.

    from flask import Flask
    app = Flask(__name__)
    
    @app.route("/")
    def hello():
        return "<h1 style='color:blue'>Hello Flask and Docker!</h1>"
    
    if __name__ == "__main__":
        app.run(debug=True)

**5. Python Dependencies**

Create a requirements.txt file with all dependencies to the flask app

    flask
    gunicorn

**6. NGINX Config File**

Create a file named nginx_config.conf with below configs.

    server {
     listen 80;
     server_name <your dns>;
    location / {
     proxy_pass [http://0.0.0.0:5000](http://0.0.0.0:8000);
     }
    }

**7. Create Docker File**

We now have all the required inputs to create a Docker file. The docker file will have commands to create a flask application docker image.

* Install Python3

* Install Flask

* Install Gunicorn

* Install Nginx

* Gunicorn and Nginx configuration to serve flask application in port 80

<iframe src="https://medium.com/media/df3c13c13d252494a38cc5b24b4e4529" frameborder=0></iframe>

**8. Build Docker Image**

Create a docker image using the Dockerfile

    docker build --tag flask_docker .

Running the above command should build a docker image with all instructions from Dockerfile

![Docker Build Image](https://cdn-images-1.medium.com/max/2000/1*faYqMqRj7arNQNgtNbz17Q.png)*Docker Build Image*

**9. Run Docker Container in EC2**

    docker run --name flask_container -dit -p 80:80 flask_docker

10.** Verify the flask page in EC2 DNS**

http://<EC2_DNS>

    Hello Flask and Docker!

That's it. We now have a portable flask application integrated with gunicorn and Nginx running inside a single docker container. Sample code [here](https://github.com/dineshshan10/dishanka-docker-repo/tree/main/flask_docker).

### Bonus: Docker Commands

Docker commands are explained in detail [here](https://docs.docker.com/engine/reference/commandline/cli/). Here are some of the commands used in the article.

1. Build a Docker Image

    docker build --tag <docker_image_tag> .

2. Run a Docker Container from Image

    docker run --name <container_name> -dit -p <C:L> <docker_image>

    C- Port running in docker container
    L - Port listening to external requests

    d -detach
    p - port
    i - Keep STDIN open even if not attached
    t- Allocate a pseudo-tty

3. SSH to Running Docker

    docker exec -it flask_container bash

4. List ,Stop and Remove Containers

    # Stop the running containers
    docker stop $(docker ps -a -q)

    # Remove stopped containers

    docker rm $(docker ps -a -q)

    # List images
    docker images

5. Reload Nginx running in Docker

    docker exec -it <container_name> nginx -s reload

* FROM: Set the base image to use

* RUN: Run command and commit the ending result (container) image

* MAINTAINER: Set the author/owner data of the Dockerfile

* USER: Set the user to run the containers from the image

* ADD: Copy a file from the host into the container

* CMD: Set default commands to be executed, or passed to the ENTRYPOINT

* ENTRYPOINT: Set the default entrypoint application inside the container

* ENV: Set environment variable (e.g. key = value)

* EXPOSE: Expose a port to outside

* WORKDIR: Set the directory for the directives of **CMD** to be executed

* VOLUME: Mount a directory from the host to the container

Thanks for reading!
