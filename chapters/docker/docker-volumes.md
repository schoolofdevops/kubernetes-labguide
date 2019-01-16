# Lab: Persistent Volumes with Docker


### Types of volumes

  * automatic volumes
  * named volumes
  * volume binding


Automatic Volumes

```
docker container run  -idt --name vt01 -v /var/lib/mysql  alpine sh
docker inspect vt01 | grep -i mounts -A 10
```


Named volumes

```
docker container run  -idt --name vt02 -v db-data:/var/lib/mysql  alpine sh
docker inspect vt02 | grep -i mounts -A 10
```


Volume binding

```
mkdir /root/sysfoo
docker container run  -idt --name vt03 -v /root/sysfoo:/var/lib/mysql  alpine sh
docker inspect vt03 | grep -i mounts -A 10
```

Sharing files between host and the container
```
ls /root/sysfoo/
touch /root/sysfoo/file1
docker exec -it vt03 sh
ls sysfoo/
```
