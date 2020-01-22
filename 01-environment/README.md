# Hadoop Platform on Docker

## Provisioning environment
The environment for this course is completely based on docker containers. 

In order to simplify the provisioning, a single docker-compose configuration is used. All the necessary software will be provisioned using Docker. 

### Prepare Environment

In the Virtual Machine, start a terminal window and execute the following commands. 

First let's add the environment variables. Make sure to adapt the network interface (**ens33** according to your environment. You can retrieve the interface name by using **ipconfig** on windows or **ifconfig* on Mac/Linux. 

```
# Prepare Environment Variables
export PUBLIC_IP=$(curl ipinfo.io/ip)
export DOCKER_HOST_IP=$(ip addr show ens33 | grep "inet\b" | awk '{print $2}' | cut -d/ -f1)
```

Now let's checkout the Workshop project from GitHub:

```
# Get the project
cd ~
git clone https://github.com/asatrya/hadoop-workshop.git
cd hadoop-workshop/01-environment/docker
```

## Start Environment

And finally let's start the environment:

```
# Make sure that the environment is not running
docker-compose down

# Startup Environment
docker-compose up -d
```

The environment should start immediately, as all the necessary images should already be available in the local docker image registry. 

The output should be similar to the one below. 

![Alt Image Text](./images/start-env-docker.png "StartDocker")

Your instance is now ready to use.

## Stop environment

To stop the environment, execute the following command:

```
docker-compose stop
```

after that it can be re-started using `docker-compose start`.

To stop and remove all running container, execute the following command:

```
docker-compose down
```

## Post Provisioning

These steps are necessary after the starting the docker environment. 

## Services accessible on Analyticsplatform Platform
The following service are available as part of the platform:

Product | Type | Service | Url
------|------| --------| -----
Hue | Development | Hue | <http://localhost:28888>
Namenode UI | Management  | Haoop HDFS | <http://localhost:50070>
Datanode-1 UI | Management  | Haoop HDFS | <http://localhost:50075>
Datanode-2 UI | Management  | Haoop HDFS | <http://localhost:50076>
Minio UI | Management | Minio | <http://localhost:9000>

