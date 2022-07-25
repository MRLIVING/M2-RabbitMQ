## TOC
* Installation with Docker
  * Installation
     * [Create a GCE](#create-a-gce-with-container-optimized-os-image)
     * [Enter the container](#docker-commands)
     * [Add packages in the container](#add-packages-in-the-container)
     * [start/restart the RabbitMQ instance](#starting-the-rabbitmq-container)

TODO ...

## Installation with Docker
### Create a GCE with [Container-Optimized](https://cloud.google.com/container-optimized-os/docs/concepts/features-and-benefits) OS image
* [RabbitMQ3 Google containers](https://console.cloud.google.com/marketplace/product/google/rabbitmq3?project=czechrepublic-290206) => [Docker Image Repository](https://console.cloud.google.com/gcr/images/cloud-marketplace/GLOBAL/google/rabbitmq3)

### [Docker commands](https://docs.docker.com/engine/reference/run/)
* [Search the Docker Hub for images](https://docs.docker.com/engine/reference/commandline/search/)  
  `docker search ubuntu -f -is-official=true`, search official released image `ubuntu` from the Docker Hub
 
* [Run a command in a new container](https://docs.docker.com/engine/reference/commandline/run/)  
  `docker run -it ubuntu /bin/bash`

* [Attach to a running container](https://docs.docker.com/engine/reference/commandline/attach/)  
  `docker attach ${ContainerID}`  
 
* [Execute a command in a running container](https://docs.docker.com/engine/reference/commandline/exec/)  
  ```
  # enter into to a container and run in Bash shell
  docker exec -it ${ContainerID} /bin/bash
  ```
  
* Exit from a container 
  * `crtl + p, q` (keep the container alive)
  * `exit` (shutdown the container)

* [List images](https://docs.docker.com/engine/reference/commandline/images/)
  `docker images`

* List containers and IDs  
  `docker ps -a`
  
* List containers  
  `docker container ls`
  
* [List networks](https://docs.docker.com/engine/reference/commandline/network_ls/)  
  `docker network ls`
  
* [Display detailed information on one or more networks](https://docs.docker.com/engine/reference/commandline/network_inspect/)  
  `docker network inspect ${NETWORK}`

* Stop a container  
  `docker stop ${ContainerID}`

* Start a container  
  `docker start ${ContainerID}`

* Remove a container  
  `docker rm ${ContainerID}`  
  * enfore remove a container event it is runnning  
    `docker rm -f ${ContainerID}`
  
* [Remove all stopped containers](https://docs.docker.com/engine/reference/commandline/container_prune/)  
  `docker container prune`

* Remove an image    
  `docker rmi ${IMAGE_ID}`
  
* [Build an image from a Dockerfile](https://docs.docker.com/engine/reference/commandline/build/)  
  `docker build -t ${repo name}:${tag} . --no-cache`
  
  * [Dockerfile](https://docs.docker.com/engine/reference/builder/)


### Add packages in the container 
```
apt-get update && apt-get install -y procps
apt-get update && apt-get install curl
apt-get update && apt-get install -y vim
```

### [Management Plugin](https://www.rabbitmq.com/management.html) - RabbitMQ Management Web UI
* [Enable management plugin](https://www.rabbitmq.com/management.html#getting-started)
```
rabbitmq-plugins enable rabbitmq_management
```

* [Enable statistics and metrics collection](https://www.rabbitmq.com/management.html#disable-stats)
```
echo "management_agent.disable_metrics_collector = false" > /etc/rabbitmq/conf.d/management_agent.disable_metrics_collector.conf
```

* Restart RabbitMQ and Check the status
```
rabbitmqctl stop_app
rabbitmqctl start_app
rabbitmqctl status
```

### Create a new image for the container’s changes
`docker commit ${ContainerID} rabbitmq3:v3.8.23_mgt_ui`

### [Create and start a RabbitMQ container in background](https://github.com/GoogleCloudPlatform/rabbitmq-docker/blob/master/README.md#using-docker)  
`docker run --network=host -d ${IMAGE_NAME}:${TAG_NAME}`,  
i.e. `docker run --network=host -d rabbitmq3:v3.8.23_mgt_ui`

* remove enforcedly the running container if required.
`docker rm -f ${ContainerID}`

### [Add a RabbitMQ user for M2 and Grant permission](https://www.rabbitmq.com/access-control.html)
```
# enter into a container shell
docker exec -it ${ContainerID} /bin/bash

rabbitmqctl add_user ${USER_NAME}
rabbitmqctl set_user_tags ${USER_NAME} administrator
rabbitmqctl set_permissions -p / ${USER_NAME} ".*" ".*" ".*"
```

### Test the connection via HTTP API
* [Get a list of vhosts](https://rawcdn.githack.com/rabbitmq/rabbitmq-server/v3.8.23/deps/rabbitmq_management/priv/www/api/index.html)  
  `curl -i -u guest:guest http://localhost:15672/api/vhosts`

* Management Web UI  
  `http://rabbitmq.mrl.com.tw:15672/`

### Register M2 extensions into the Exchange and Queue of the RabbitMQ 
```
./bin/magento setup:upgrade
./bin/magento setup:di:compile
```





## [RabbitMQ popular commands](https://www.rabbitmq.com/rabbitmqctl.8.html) 
* delete a user  
  `rabbitmqctl delete_user ${USER_NAME}`

### [Connect RabbitMQ to Magento2](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/install-rabbitmq.html#connect-rabbitmq-to-magento-open-source-or-adobe-commerce)


## Reference
* [Getting started with M2 message queue](https://www.atwix.com/magento-2/getting-started-with-message-queues-in-magento/)
* [Asynchronous Message Queue configuration files](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/message-queues/async-message-queue-config-files.html)
* [RabbitMQ Port Access](https://www.rabbitmq.com/networking.html#ports)
* [Docker Container 基礎入門篇 1](https://azole.medium.com/docker-container-%E5%9F%BA%E7%A4%8E%E5%85%A5%E9%96%80%E7%AF%87-1-3cb8876f2b14)
* [RabbitMQ Management HTTP API](https://rawcdn.githack.com/rabbitmq/rabbitmq-server/v3.8.23/deps/rabbitmq_management/priv/www/api/index.html)
