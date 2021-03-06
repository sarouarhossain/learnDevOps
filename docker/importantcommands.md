## Pull
--------------------
* ***docker pull image_name***
* ***docker images*** will show all images in your local

## list of containers
--------------------
* ***docker ps -a*** show all running and stopped containers
* ***docker ps -aq*** list of all containers, only id

## run
--------------------
* ***docker run image_name:version***

**Note:** 
* ***-d*** means detached mode
* ***-p*** means port [ ***-p host_port:conatiner_port*** ], used to bind container port to our machine port
* ***--name*** used give a name to the port [***--name port_name***]

start
-----------------
* ***docker start container_id***

### start vs run 

* ***run*** -> means creating a container
* ***start*** -> means running a stopped container

logs
------------
* ***docker logs container_id/container_name***

## log in to a container
----------------------
* ***docker exec -it container_id bash***

**Note:** **-it** means interactive terminal

## stop
--------
* ***docker stop container_id*** [stop a single container] 
* ***docker stop $(docker ps -aq)*** [stop all containers]
* ***docker rm $(docker ps -aq)*** [remove all containers]
* ***docker rmi image_id*** 
* ***docker rmi $(docker images -q)*** [remove all images]
* ***docker image prune*** [will remove all dangling images. dangling image means this is not tagged and used by any containers]
* ***docker image prune -a*** [will remove not only dangling but also all unreferenced images]
* ***docker container rm $(docker container ls -aq)***

## volumes
--------------
* ***docker volume ls***
* ***docker volume rm volume_id***
* ***docker volume prune*** [remove all volumes. ]
 
**Note:** to bypass the prompt need to use **--force** or **-f**

## networks
----------------
* ***docker network ls***
* ***docker network rm network_id***
* ***docker network prune*** [remove all networks]

**Note:** to bypass the prompt need to use **--force** or **-f**

## docker-compose
* ***docker-compose up*** or ***docker-compose up -d***
* ***docker-compose build*** or ***docker-compose up --build***

