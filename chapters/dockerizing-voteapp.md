## Lab : Build a docker image for Instavote frontend vote app

[Voteapp](https://github.com/schoolofdevops/vote) is a app written in python. Its a simple, web based  application which serves as a frontend for Instavote project. As a devops engineer, you have been tasked with building an image for vote app and publish it to docker hub registry.


### Approach 1: Building docker image for voteapp manually


on the host

```
git clone https://github.com/schoolofdevops/vote
docker container run -idt --name build -p 8000:80 python:2.7-alpine sh
cd vote
docker cp . build:/app
docker exec -it build sh
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
docker diff build

docker container commit build <docker_id>/vote:v1

docker login

docker image push <docker_id>/vote:v1

```





### Approach 2: Building image with Dockerfile


Change into vote  directory which containts the source code.  This assumes you have already cloned the repo. If not, clone it from https://github.com/schoolofdevops/vote

```
cd vote
ls

app.py  requirements.txt  static  templates
```

Add/create  Dockerfile the the same directory (vote) with the following content,

```
FROM python:2.7-alpine

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt

EXPOSE 80

CMD  gunicorn app:app -b 0.0.0.0:80
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
