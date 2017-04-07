# <a name="about"></a>About

This image contains an installation of RabbitMQ, open-source message broker.
[Official Image Launcher Page](https://console.cloud.google.com/launcher/details/google/rabbitmq3).

Pull command:
```shell
gcloud docker -- pull gcr.io/orbitera-dev/rabbitmq3
```

Dockerfile for this image can be found [here](https://github.com/GoogleCloudPlatform/rabbitmq-docker/tree/master/3.6/).

# <a name="table-of-contents"></a>Table of Contents
* [Using Kubernetes](#using-kubernetes)
  * [How to use RabbitMQ server instance](#how-to-use-rabbitmq-server-instance-kubernetes)
    * [Start single RabbitMQ container](#start-single-rabbitmq-container-kubernetes)
    * [Conncect to RabbitMQ using rabbitmqctl](#conncect-to-rabbitmq-using-rabbitmqctl-kubernetes)
    * [Run with persistent data volumes](#run-with-persistent-data-volumes-kubernetes)
* [Using Docker](#using-docker)
  * [How to use RabbitMQ server instance](#how-to-use-rabbitmq-server-instance-docker)
    * [Start single RabbitMQ container](#start-single-rabbitmq-container-docker)
    * [Conncect to RabbitMQ using rabbitmqctl](#conncect-to-rabbitmq-using-rabbitmqctl-docker)
    * [Run with persistent data volumes](#run-with-persistent-data-volumes-docker)
* [References](#references)
  * [Ports](#references-ports)
  * [Environment Variables](#references-environment-variables)
  * [Volumes](#references-volumes)

# <a name="using-kubernetes"></a>Using Kubernetes

## <a name="how-to-use-rabbitmq-server-instance-kubernetes"></a>How to use RabbitMQ server instance

This section describes how to spin up a RabbitMQ service using this image.

### <a name="start-single-rabbitmq-container-kubernetes"></a>Start single RabbitMQ container

Copy the following content to `pod.yaml` file, and run `kubectl create -f pod.yaml`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: some-rabbitmq
  labels:
    name: some-rabbitmq
spec:
  containers:
    - image: rabbitmq3
      name: rabbitmq
      env:
        - name: RABBITMQ_ERLANG_COOKIE
          value: unique-erlang-cookie
```

Run the following to expose the ports:
```shell
kubectl expose pod some-rabbitmq --name some-rabbitmq-4369 \
  --type LoadBalancer --port 4369 --protocol TCP
kubectl expose pod some-rabbitmq --name some-rabbitmq-5671 \
  --type LoadBalancer --port 5671 --protocol TCP
kubectl expose pod some-rabbitmq --name some-rabbitmq-5672 \
  --type LoadBalancer --port 5672 --protocol TCP
kubectl expose pod some-rabbitmq --name some-rabbitmq-25672 \
  --type LoadBalancer --port 25672 --protocol TCP
```

RabbitMQ will be accessible on your localhost using on ports 5671, 5672 or by rabbtmqctl binary.

For information about how to retain your RabbitMQ data across restarts, see [Run with persistent data volumes](#run-with-persistent-data-volumes-kubernetes).

### <a name="conncect-to-rabbitmq-using-rabbitmqctl-kubernetes"></a>Conncect to RabbitMQ using rabbitmqctl

```shell
kubectl run \
  some-ctl \
  --image rabbitmq3 \
  -env="RABBITMQ_ERLANG_COOKIE=unique-erlang-cookie" \
  --rm --attach --restart=Never \
  -it \
  -- /bin/bash
```

For remote/cross-container administration via rabbitmqctl you should set RABBITMQ_ERLANG_COOKIE. And run "rabbitmqctl"

### <a name="run-with-persistent-data-volumes-kubernetes"></a>Run with persistent data volumes

We can store data on persistent volumes, this way the installation remains intact across restarts.

Copy the following content to `pod.yaml` file, and run `kubectl create -f pod.yaml`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: some-rabbitmq
  labels:
    name: some-rabbitmq
spec:
  containers:
    - image: rabbitmq3
      name: rabbitmq
      env:
        - name: RABBITMQ_ERLANG_COOKIE
          value: unique-erlang-cookie
      volumeMounts:
        - name: rabbitmq-data
          mountPath: /var/lib/rabbitmq
  volumes:
    - name: rabbitmq-data
      persistentVolumeClaim:
        claimName: rabbitmq-data
---
# Request a persistent volume from the cluster using a Persistent Volume Claim.
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: rabbitmq-data
  annotations:
    volume.alpha.kubernetes.io/storage-class: default
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi
```

Run the following to expose the ports:
```shell
kubectl expose pod some-rabbitmq --name some-rabbitmq-4369 \
  --type LoadBalancer --port 4369 --protocol TCP
kubectl expose pod some-rabbitmq --name some-rabbitmq-5671 \
  --type LoadBalancer --port 5671 --protocol TCP
kubectl expose pod some-rabbitmq --name some-rabbitmq-5672 \
  --type LoadBalancer --port 5672 --protocol TCP
kubectl expose pod some-rabbitmq --name some-rabbitmq-25672 \
  --type LoadBalancer --port 25672 --protocol TCP
```

# <a name="using-docker"></a>Using Docker

## <a name="how-to-use-rabbitmq-server-instance-docker"></a>How to use RabbitMQ server instance

This section describes how to spin up a RabbitMQ service using this image.

### <a name="start-single-rabbitmq-container-docker"></a>Start single RabbitMQ container

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.
```yaml
version: '2'
services:
  rabbitmq:
    image: rabbitmq3
    environment:
      RABBITMQ_ERLANG_COOKIE: unique-erlang-cookie
    ports:
      - '4369:4369'
      - '5671:5671'
      - '5672:5672'
      - '25672:25672'
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-rabbitmq \
  -e RABBITMQ_ERLANG_COOKIE=unique-erlang-cookie \
  -p 4369:4369 \
  -p 5671:5671 \
  -p 5672:5672 \
  -p 25672:25672 \
  -d \
  rabbitmq3
```

RabbitMQ will be accessible on your localhost using on ports 5671, 5672 or by rabbtmqctl binary.

For information about how to retain your RabbitMQ data across restarts, see [Run with persistent data volumes](#run-with-persistent-data-volumes-docker).

### <a name="conncect-to-rabbitmq-using-rabbitmqctl-docker"></a>Conncect to RabbitMQ using rabbitmqctl

```shell
docker run \
  --name some-ctl \
  -e RABBITMQ_ERLANG_COOKIE=unique-erlang-cookie \
  --rm \
  -it \
  rabbitmq3 \
  /bin/bash
```

For remote/cross-container administration via rabbitmqctl you should set RABBITMQ_ERLANG_COOKIE. And run "rabbitmqctl"

### <a name="run-with-persistent-data-volumes-docker"></a>Run with persistent data volumes

We can store data on persistent volumes, this way the installation remains intact across restarts. Assume that `/var/lib/rabbitmq` is the persistent directory on the host.

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.
```yaml
version: '2'
services:
  rabbitmq:
    image: rabbitmq3
    environment:
      RABBITMQ_ERLANG_COOKIE: unique-erlang-cookie
    ports:
      - '4369:4369'
      - '5671:5671'
      - '5672:5672'
      - '25672:25672'
    volumes:
      - /var/lib/docker/volumes:/var/lib/rabbitmq
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-rabbitmq \
  -e RABBITMQ_ERLANG_COOKIE=unique-erlang-cookie \
  -p 4369:4369 \
  -p 5671:5671 \
  -p 5672:5672 \
  -p 25672:25672 \
  -v /var/lib/docker/volumes:/var/lib/rabbitmq \
  -d \
  rabbitmq3
```

# <a name="references"></a>References

## <a name="references-ports"></a>Ports

These are the ports exposed by the container image.

| **Port** | **Description** |
|:---------|:----------------|
| TCP 4369 | a peer discovery service used by RabbitMQ nodes and CLI tools |
| TCP 5671 | AMQP 0-9-1 and 1.0 clients without TLS |
| TCP 5672 | AMQP 0-9-1 and 1.0 clients with TLS |
| TCP 25672 | used by Erlang distribution for inter-node and CLI tools communication |

## <a name="references-environment-variables"></a>Environment Variables

These are the environment variables understood by the container image.

| **Variable** | **Description** |
|:-------------|:----------------|
| RABBITMQ_ERLANG_COOKIE | RabbitMQ nodes and CLI tools (e.g. rabbitmqctl) use a cookie to determine whether they are allowed to communicate with each other. For two nodes to be able to communicate they must have the same shared secret called the Erlang cookie. |

## <a name="references-volumes"></a>Volumes

These are the filesystem paths used by the container image.

| **Path** | **Description** |
|:---------|:----------------|
| /var/lib/rabbitmq | All RabbitMQ files are installed here. |
