# Lab: Docker Networking


### Host Networking

  * bridge
  * host
  * peer
  * none



Examine the existing network

```
docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
b3d405dd37e4        bridge              bridge              local
7527c821537c        host                host                local
773bea4ca095        none                null                local
```



Creating new network

```
docker network create -d bridge mynet
```

validate
```
docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
b3d405dd37e4        bridge              bridge              local
7527c821537c        host                host                local
4e0d9b1a39f8        mynet               bridge              local
773bea4ca095        none                null                local
```


```
docker network inspect mynet


[
   {
       "Name": "mynet",
       "Id": "4e0d9b1a39f859af4811986534c91527146bc9d2ce178e5de02473c0f8ce62d5",
       "Created": "2018-05-03T04:44:19.187296148Z",
       "Scope": "local",
       "Driver": "bridge",
       "EnableIPv6": false,
       "IPAM": {
           "Driver": "default",
           "Options": {},
           "Config": [
               {
                   "Subnet": "172.18.0.0/16",
                   "Gateway": "172.18.0.1"
               }
           ]
       },
       "Internal": false,
       "Attachable": false,
       "Ingress": false,
       "ConfigFrom": {
           "Network": ""
       },
       "ConfigOnly": false,
       "Containers": {},
       "Options": {},
       "Labels": {}
   }
]
```
#### Launching containers in different bridges

Launch two containers **nt01** and **nt02** in **default** bridge network

```
docker container run -idt --name nt01 alpine sh
docker container run -idt --name nt02 alpine sh
```

Launch two containers **nt03** and **nt04** in **mynet** bridge network

```
docker container run -idt --name nt03 --net mynet alpine sh
docker container run -idt --name nt04 --net mynet alpine sh
```


Now, lets examine if they can interconnect,

```

docker exec nt01 ifconfig eth0
docker exec nt02 ifconfig eth0
docker exec nt03 ifconfig eth0
docker exec nt04 ifconfig eth0

```

This is what I see

nt01 :  172.17.0.18

nt02 :  172.17.0.19

nt03 :  172.18.0.2

nt04 :  172.18.0.3



Create a table with the ips on your host.  Once you do that,

Try to,

  * ping from **nt01** to **nt02**  
  * ping from **nt01** to **nt03**  
  * ping from **nt03** to **nt04**  
  * ping from **nt03** to **nt02**  


e.g.

[replace ip addresses as per your setup]
```
docker exec nt01  ping 172.17.0.19

docker exec nt01  ping 172.18.0.2

docker exec nt03  ping 172.17.0.19

docker exec nt03  ping 172.18.0.2


```

Clearly, these two are two differnt subnets/networks even though running on the same host. **nt01** and **nt02** can connect with each other, whereas **nt03**  and **nt04** can connect. But connection between containers attached to two different subnets is not possible.



#### Using None Network Driver

```
docker container run -idt --name nt05 --net none alpine sh

docker exec -it nt05 sh

ifconfig
```



#### Using Host Network Driver


```
docker container run -idt --name nt05 --net host  alpine sh

docker exec -it nt05 sh

ifconfig
```


### Observe docker bridge, routing and port mapping

Exercise: Read about [**netshoot** utility here](https://github.com/nicolaka/netshoot)


Launch netshoot and connect to the host network

```

docker run -it --net host --privileged  nicolaka/netshoot

```

Examine port mapping,

```
iptables -nvL -t nat
```

Traverse host port to container ip and port.

Observe docker bridge and routing with the following command,

```
brctl show

ip route show

```
