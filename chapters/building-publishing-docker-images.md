## Lab : Build a docker image for Instavote frontend vote app

[Voteapp](https://github.com/schoolofdevops/vote) is a app written in python. Its a simple, web based  application which serves as a frontend for Instavote project. As a devops engineer, you have been tasked with building an image for vote app and publish it to docker hub registry.


### Approach 1: Building docker image for voteapp manually


on the host

```
git clone https://github.com/schoolofdevops/vote
docker container run -idt --name dev -p 8000:80 python:alpine3.17 sh
cd vote
docker cp . dev:/app
docker exec -it dev sh
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
docker diff dev

docker container commit dev docker.io/<username>/vote:v1

[where <username> should be replaced with your registry username ]

docker login

docker image push docker.io/<username>/vote:v1

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
FROM python:alpine3.17

WORKDIR /app

COPY . .

RUN pip install -r requirements.txt

EXPOSE 80

CMD  gunicorn app:app -b 0.0.0.0:80
```

Build image using,

```
 docker build -t docker.io/<username>/vote:v2 .
```

where,
  <username> : your docker registry user/namespace. Replace this with the actual user


validate

```
docker image ls
docker image history docker.io/<username>/vote:v2
docker image history docker.io/<username>/vote:v1

docker container run -idt -P docker.io/<username>/vote:v2
docker ps
```

Check by connecting to your host:port to validate if vote web application shows up.


Once validated, tag and push

```
docker image tag docker.io/<username>/vote:v2 docker.io/<username>/vote:latest
docker login
docker push docker.io/<username>/vote
docker push docker.io/<username>/vote:v2

```
