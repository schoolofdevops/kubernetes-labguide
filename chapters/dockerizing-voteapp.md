## Lab : Build a docker image for Instavote frontend vote app

[Voteapp](https://github.com/schoolofdevops/vote) is a app written in python. Its a simple, web based  application which serves as a frontend for Instavote project. As a devops engineer, you have been tasked with building an image for vote app and publish it to docker hub registry.


### Approach 1: Building docker image for facebooc app manually


on the host

```
git clone https://github.com/schoolofdevops/vote
docker container run -idt --name vote -p 8000:80 python:2.7-alpine sh
cd vote
docker cp . vote:/app
docker exec -it vote sh
```

inside the container

```
cd /app
pip install -r requirements.txt
gunicorn app:app -b 0.0.0.0:80
```

Validate by accessing http://IPADDRESS:8000

on the host

```
docker diff fb

docker container commit vote <docker_id>/vote:v1

docker login

docker image push <docker_id>/vote:v1

```





### Approach 2: Building image with Dockerfile


Change into facebooc directory which containts the source code.  This assumes you have already cloned the repo. If not, clone it from https://github.com/schoolofdevops/facebooc

```
cd vote
ls

app.py  requirements.txt  static  templates
```

Add/create  Dockerfile the the same directory (vote) with the following content,

```
# Using official python runtime base image
FROM python:2.7-alpine

# Set the application directory
WORKDIR /app

# Install our requirements.txt
ADD requirements.txt /app/requirements.txt
RUN pip install -r requirements.txt

# Copy our code from the current folder to /app inside the container
ADD . /app

# Make port 80 available for links and/or publish
EXPOSE 80

# Define our command to be run when launching the container
CMD  gunicorn app:app -b 0.0.0.0:80 --log-file - --access-logfile - --workers 4 --keep-alive 0
```

Build image using,

```
 docker build -t <docker_id>/vote:v2 .
```

where,
  <docker_id> : your docker registry user/namespace. Replace this with the actual user


validate

```
docker image ls
docker image history <docker_id>/vote:v2
docker image history <docker_id>/vote:v1

docker container run -idt -P <docker_id>/vote:v2
docker ps
```

Check by connecting to your host:port to validate if vote web application shows up.


Once validated, tag and push

```
docker image tag <docker_id>/vote:v2 <docker_id>/vote:latest
docker login
docker push <docker_id>/vote
```
