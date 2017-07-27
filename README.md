# Auto creating DOCKER SWARM clusters on DigitalOcean and running services

This small project focuses on creating docker nodes, initialize docker swarm, creating a sample web service and traffic generation. The examples adds advanced features like labels, attachable overlays, etc ... to make it a little more advanced. 

You're required to generate an API token in DigitalOcean control panel.


The script uses the latest version of docker-machine.
See https://docs.docker.com/machine/install-machine/ for installation guidelines. 
Obtain the latest version at https://github.com/docker/machine/releases/


```
#!/bin/bash
docker-machine ls

#obtain an API token https://cloud.digitalocean.com/settings/api/tokens
export TOKEN=<insert API token here>

#create 1st manager nodes
docker-machine create --driver digitalocean --digitalocean-access-token=$TOKEN manager1
eval $(docker-machine env manager1)
docker swarm init --advertise-addr $(docker-machine ip manager1)

export SWARM_MANAGER_JOIN_TOKEN=$(docker swarm join-token -q manager)
export SWARM_WORKER_JOIN_TOKEN=$(docker swarm join-token -q worker)
export SWARM_MANAGER_IP=$(docker-machine ip manager1)

#create 1 to 2 worker nodes
for N in `seq 1 2`; do
docker-machine create --driver digitalocean --digitalocean-access-token=$TOKEN  worker$N
eval $(docker-machine env worker$N)
docker swarm join --token $SWARM_WORKER_JOIN_TOKEN $SWARM_MANAGER_IP:2377
done


```

When finished, point your docker client to a SWARM master
```
eval $(docker-machine env manager1)
```

List the docker nodes and swarm status
```
docker node ls

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
6jhi6g2jm1wxr43voophkcfr0     worker1             Ready               Active              
ikha8tk61t2ojvyfycaxa8jcq *   manager1            Ready               Active              Leader
madeihv8i14uzjl1jiloq9s6c     worker2             Ready               Active              
```



