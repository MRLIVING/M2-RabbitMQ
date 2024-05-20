## TOC
* [Installation with Docker](#installation-with-docker)
  * [Pull the RabbitMQ v3.8 docker image](#pull-the-rabbitmq-v38-docker-image)
  * [Create and start a RabbitMQ container in background](#create-and-start-a-rabbitmq-container-in-background)  
  * [Enter into the container](#enter-into-the-container)
    * [Add packages in the container](#add-packages-in-the-container)
    * [Management Plugin - RabbitMQ Management Web UI](#management-plugin---rabbitmq-management-web-ui)
    * [Add a RabbitMQ user for M2 and Grant permission](#add-a-rabbitmq-user-for-m2-and-grant-permission)
  * [Test the connection via HTTP API](#test-the-connection-via-http-api)
  * [Connect RabbitMQ to Magento2](#connect-rabbitmq-to-magento2)
  * [Add the Exchange and Queue into RabbitMQ via enable the Firebear M2 extension](#add-the-exchange-and-queue-into-rabbitmq-via-enable-the-firebear-m2-extension)
  

## Installation with Docker
### [Docker Quick Start](https://github.com/MRLIVING/Becca/wiki/Docker-Quick-Start)

### [Pull the RabbitMQ v3.8 docker image](#pull-the-rabbitmq-v38-docker-image)

### Create a GCE with [Container-Optimized](https://cloud.google.com/container-optimized-os/docs/concepts/features-and-benefits) OS image
* [RabbitMQ3 Google containers](https://console.cloud.google.com/marketplace/product/google/rabbitmq3?project=czechrepublic-290206) => [Docker Image Repository](https://console.cloud.google.com/gcr/images/cloud-marketplace/GLOBAL/google/rabbitmq3)

### Pull the [RabbitMQ v3.8 docker image](https://console.cloud.google.com/gcr/images/cloud-marketplace/global/google%2Frabbitmq3@sha256:20c452f900a50d27a6fab69bbe2bd33571f94dae4e23682157297102fb8325c7/details?tab=vulnz)
`docker pull gcr.io/cloud-marketplace/google/rabbitmq3:3.8`

### [Create and start a RabbitMQ container in background](https://github.com/GoogleCloudPlatform/rabbitmq-docker/blob/master/README.md#using-docker)  
`docker run --network=host -d ${IMAGE_NAME}:${TAG_NAME}`,  
i.e. `docker run --network=host -d rabbitmq3:v3.8.23_mgt_ui`

* remove enforcedly the deprecated running container if required.  
`docker rm -f ${ContainerID}`

* Create a new image for the container’s changes
`docker commit ${ContainerID} rabbitmq3:v3.8.23_mgt_ui`

### Enter into the container
`docker exec -it ${ContainerID} /bin/bash`

#### Add packages in the container 
```
apt-get update && apt-get install -y procps
apt-get update && apt-get install -y curl
apt-get update && apt-get install -y vim
```

#### [Management Plugin](https://www.rabbitmq.com/management.html) - RabbitMQ Management Web UI
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

#### [Add a RabbitMQ user for M2 and Grant permission](https://www.rabbitmq.com/access-control.html)
```
# enter into a container shell
docker exec -it ${ContainerID} /bin/bash

rabbitmqctl add_user ${USER_NAME}
rabbitmqctl set_user_tags ${USER_NAME} administrator
rabbitmqctl set_permissions -p / ${USER_NAME} ".*" ".*" ".*"
```

#### [RabbitMQ popular commands](https://www.rabbitmq.com/rabbitmqctl.8.html) 
* delete a user  
  `rabbitmqctl delete_user ${USER_NAME}`
  
  
### Test the connection via HTTP API
* [Get a list of vhosts](https://rawcdn.githack.com/rabbitmq/rabbitmq-server/v3.8.23/deps/rabbitmq_management/priv/www/api/index.html)   
  `curl -i -u guest:guest http://localhost:15672/api/vhosts`

* Management Web UI  
  `http://rabbitmq.mrl.com.tw:15672/`


### [Connect RabbitMQ to Magento2](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/install-rabbitmq.html#connect-rabbitmq-to-magento-open-source-or-adobe-commerce)

* find the `queue` section from the `${M2_ROOT}/app/etc/env.php`

```
...
'queue' =>
  array (
    'amqp' =>
    array (
      'host' => 'rabbitmq.mrl.com.tw',
      'port' => '5672',
      'user' => '${USER_NAME}',
      'password' => '${PASSWD}',
      'virtualhost' => '/'
     ),
  ),
...
```

### Listening Ports List
```
amqp | 5672
clustering | 25672
http | 15672
```

### [Add the Exchange and Queue into RabbitMQ via enable the Firebear M2 extension](https://docs.google.com/document/d/1fEzuuAJwe0w8r2uv4I3Zq72z5VGLffdAcAHZJ71yRr4/edit#heading=h.zapdgg7thjdw)
```
bin/magento module:disable Firebear_SapDbQueue
bin/magento setup:upgrade
bin/magento setup:di:compile
bin/magento setup:static-content:deploy -f
chown -R :www-data .
```

### [Create a Docker Image From a Container](https://www.dataset.com/blog/create-docker-image/)
- 先將目前 rabbitmq 設定檔案匯出，當作是重新建立時的 init 資料。（exchange & queue 皆會存在)
```
# enter into a container shell
docker exec -it ${ContainerID} /bin/bash

rabbitmqctl export_definitions /etc/rabbitmq/definitions.json
```
- 建立 IMAGE
```
docker commit ${Container_Name} ${IMAGE_NAME}
```
i.e `docker commit rabbitmq rabbitmq_prod`
- check image
```
docker images -a
```
<img width="1114" alt="image" src="https://github.com/MRLIVING/M2-RabbitMQ/assets/99633846/f600e341-7cf1-49be-b0dc-9ca9ee655f3e">

### Troubleshooting
Q: when the container was crash.
Ans:
由於 rabbitmq 與 M2 的 exchange & queue 會有關聯，故重啟時，須確保與 M2 設定已關聯，否則會需要 upgrade M2 才能成功將 M2 & rabbitmq 連線。
目前已有兩種方式可在 M2 upgrade 的情況下，將 rabbitmq 重新掛起。

1. 從已經建立好的 `rabbitmq_prod` 重新起 container (此部分是將 2024/05/20 正常運作的 container 做成 image, 其中已包含 rabbitmq 設定檔案） 
- 重新起新的 container
```
docker run -d -p 15672:15672 -p 25672:25672 -p 5672:5672 -e --name ${Container_Name}  --restart=always ${IMAGE_NAME}
```
i.e. `docker run -d -p 15672:15672 -p 25672:25672 -p 5672:5672 -e --name rabbitmq  --restart=always rabbit_prod`
***

2. 重新由 rabbitmq3/v3.8.23_mgt_ur 起 container
- 2-1. 重新起新的 container
```
docker run -d -p 15672:15672 -p 25672:25672 -p 5672:5672 -e RABBITMQ_DEFAULT_USER=${USER} -e RABBITMQ_DEFAULT_PASS=${USER_PASS} --name ${Container_Name}  --restart=always ${IMAGE_NAME}
```
i.e. `docker run -d -p 15672:15672 -p 25672:25672 -p 5672:5672 -e RABBITMQ_DEFAULT_USER=mage -e RABBITMQ_DEFAULT_PASS=mage --name rabbitmq  --restart=always rabbitmq3:v3.8.23_mgt_ui`

- 2-2. import rabbitmq exchange & queue
```
curl -u mage:mage -H "Content-Type: application/json" -X POST -T ${RABBITMQ_JSON} http://127.0.0.1:15672/api/definitions
```
i.e. `curl -u mage:mage -H "Content-Type: application/json" -X POST -T /home/dio/rabbitmq.json http://127.0.0.1:15672/api/definitions`

- 2-3. Enter into the container
```
docker exec -it ${ContainerID} /bin/bash
```
- 2-4. Enable management plugin
```
rabbitmq-plugins enable rabbitmq_management
```
- 2-5. Enable statistics and metrics collection
```
echo "management_agent.disable_metrics_collector = false" > /etc/rabbitmq/conf.d/management_agent.disable_metrics_collector.conf
```
- 2-6. Restart RabbitMQ and Check the status
```
rabbitmqctl stop_app
rabbitmqctl start_app
rabbitmqctl status
```


## Reference
* [AMQP 0-9-1 Model Explained](https://www.rabbitmq.com/tutorials/amqp-concepts.html#programmable-protocol)
  * [RabbitMQ Tutorial](https://www.rabbitmq.com/getstarted.html)
    * [Hello World!, simple queue usage](https://www.rabbitmq.com/tutorials/tutorial-one-go.html)
* [Getting started with M2 message queue](https://www.atwix.com/magento-2/getting-started-with-message-queues-in-magento/)
* [Asynchronous Message Queue configuration files](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/message-queues/async-message-queue-config-files.html)
* [RabbitMQ Port Access](https://www.rabbitmq.com/networking.html#ports)
* [Docker Container 基礎入門篇 1](https://azole.medium.com/docker-container-%E5%9F%BA%E7%A4%8E%E5%85%A5%E9%96%80%E7%AF%87-1-3cb8876f2b14)
* [RabbitMQ Management HTTP API](https://rawcdn.githack.com/rabbitmq/rabbitmq-server/v3.8.23/deps/rabbitmq_management/priv/www/api/index.html)
* [Google docker images](https://console.cloud.google.com/gcr/images/cloud-marketplace/global/google) (use Filter to get the desired image.)
* [RabbitMQ 設定匯入匯出](https://www.rabbitmq.com/docs/definitions)
  
