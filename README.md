mongodb-automation
==================

mongodb-automation is a docker image that allows you to deploy MongoDB instances using [MMS Automation service](https://mms.mongodb.com).

The image provides a pre-installed and configured MMS Automation agent.


Examples
--------

### Running a single container

Lunch a single mongodb-automation container where we are going to deploy one or more mongod instances

    docker run -d \
        --name my_mongod \
        --hostname mymongod \
        -p 27017:27017 \
        -v '/etc/ssl/certs:/etc/ssl/certs' \
        mcascallares/mongodb-automation:latest \
        --mmsBaseUrl https://mms.mongodb.com \
        --mmsGroupId=<your_mms_group_id> \
        --mmsApiKey=<your_mms_api_key>


From MMS interface configure the container with the desired options


### Running a 3-nodes replica set across multiple containers

To run across multiple containers you need some discovery mechanism to provide connectivity among docker containers. A common approach is to use logical names and DNS.

In this example I will use [Skydock](https://github.com/crosbymichael/skydock) to provide DNS resolution. The DNS server will be listening on 172.17.42.1:53.

    # running a DNS container with Skydns
    docker run -d \
        -p 172.17.42.1:53:53/udp \
        --name skydns crosbymichael/skydns \
        -nameserver 8.8.8.8:53 \
        -domain docker


    # running Skydock container to hook docker events with DNS updates
    docker run -d \
        -v /var/run/docker.sock:/docker.sock \
        --name skydock crosbymichael/skydock \
        -ttl 30 \
        -environment dev \
        -s /docker.sock \
        -domain docker \
        -name skydns


    # running 3 mongod processes in 3 different containers, one agent per container.
    docker run -d \
        --name mongod1 \
        -h mongod1.mongodb-automation.dev.docker \
        --dns 172.17.42.1 \
        -p 27017:27000 \
        -v '/etc/ssl/certs:/etc/ssl/certs' \
        mcascallares/mongodb-automation:latest \
        --mmsBaseUrl https://mms.mongodb.com \
        --mmsGroupId=<your_mms_group_id> \
        --mmsApiKey=<your_mms_api_key>


    docker run -d \
        --name mongod2 \
        -h mongod2.mongodb-automation.dev.docker \
        --dns 172.17.42.1 \
        -p 27018:27000 \
        -v '/etc/ssl/certs:/etc/ssl/certs' \
        mcascallares/mongodb-automation:latest \
        --mmsBaseUrl https://mms.mongodb.com \
        --mmsGroupId=<your_group_id> \
        --mmsApiKey=<your_mms_api_key>


    docker run -d \
        --name mongod3 \
        -h mongod3.mongodb-automation.dev.docker \
        --dns 172.17.42.1 \
        -p 27019:27000 \
        -v '/etc/ssl/certs:/etc/ssl/certs' \
        mcascallares/mongodb-automation:latest \
        --mmsBaseUrl https://mms.mongodb.com \
        --mmsGroupId=<your_group_id> \
        --mmsApiKey=<your_mms_api_key>




Misc
----

- [Docker and certificate issues](http://blog.bwhaley.com/ca-certificates-for-docker-busybox-containers)

