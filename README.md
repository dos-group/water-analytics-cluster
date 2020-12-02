# water-analytics-cluster

This umbrella helm chart deploys a virtualized cluster environment in Kubernetes for the analytics platform of the [WaterGridSense 4.0](https://www.dos.tu-berlin.de/menue/research/watergridsense_40/) project. The deployable subcharts include: Apache Cassandra, Apache Flink, Grafana, Apache Kafka, Prometheus, Persistent Volume Provisioner, RabbitMQ, Redis, and Zookeeper.

## Using the helm charts

### Configuration

Charts are deployed using [Helm](https://helm.sh). Configure which charts to deploy in the global values.yaml by setting ``enabled: true`` for each desired technology. Cluster sizes and ports for external access can also be specified here.

Each subchart can be deployed by itself and contains its own values.yaml file with futher configurations. If deployed from the umbrella chart, values in the global values.yaml will overwrite the values in the subchart's values.yaml.

### Using the helm charts
Deploy the charts with:
```
helm install [DEPLOYMENT NAME] [CHART / ROOT DIRECTORY] -n [NAMESPACE]
```

Note: when using helm v3, the namespace must already exist.

Uninstall the charts with:
```
helm uninstall [DEPLOYMENT NAME]
```


## RabbitMQ
[RabbitMQ](https://www.rabbitmq.com) is a open source message broker that supports a range of protocols, including MQTT.

RabbitMQ can be accessed outside of the Kubernetes cluster using the follow credentials:

```
user: user
pass: [password]
host: [any node ip in the kubernetes clubster]
```
Ports can be set in the global values.yaml file and can not already be in use:
```
amqp:     30672
amqp/ssl: 30671
mqtt:     31672
mqtt/ssl: 31671
```

For security reasons, the password for the default user is randomly generated at deployment.
To view the password, run the following command inside the cluster.
``echo "Password: $(kubectl get secret --namespace [namespace] rabbitmq -o jsonpath="{.data.rabbitmq-password}" | base64 --decode)"``

Additional users and permissions can be added using the rabbitmqctl command line tool, executed directly from a rabbitmq pod:
``kubectl exec rabbitmq-0 -- rabbitmqctl add_user "username" "password"``
``kubectl exec rabbitmq-0 -- rabbitmqctl set_permissions -p "/" "username" ".*" ".*" ".*"``
The new users should be replicated across all nodes in the cluster.

Per default, message queues are not replicated across all nodes. To enable replication across all nodes, run:
``kubectl exec rabbitmq-0 -- rabbitmqctl set_policy ha-all "^ha\." '{"ha-mode":"all"}'``

To enable TLS support, use the RabbitMQ [documentation](https://www.rabbitmq.com/ssl.html#automated-certificate-generation) to generate the certificates. Encryption can be enabled in the global values.yaml file by setting ``tls.enabled: true`` and providing the name of the Kubernetes secret in ``tls.exisitingSecret`` containing the generated certificiates.

To create the secret in Kubernetes, update the paths to the certificates and run:
```
kubectl create secret generic rabbitmq-certificates --from-file=./ca.crt --from-file=./tls.crt --from-file=./tls.key
````

A RabbitMQ [definitions](https://www.rabbitmq.com/definitions.html) file can be loaded to configure RabbitMQ in the global values.yaml file by setting ``loadDefinition.enabled: true`` and providing the Kubernetes secret name in ``exisitingSecret``.

[Helm chart source](https://github.com/bitnami/charts/tree/master/bitnami/rabbitmq)


## Kafka
[Apache Kafka](https://kafka.apache.org)

[Helm chart source](https://github.com/bitnami/charts/tree/master/bitnami/kafka)


## Flink
[Apache Flink](https://flink.apache.org)


## Zookeeper
[Apache Zookeeper](https://zookeeper.apache.org)

[Helm chart source](https://github.com/helm/charts/tree/master/incubator/zookeeper)


## Cassandra
[Cassandra](https://cassandra.apache.org) is a scalable, distributed database.

The database user and password can be specified in the global values.yaml file. Client to node encryption can be enabled by setting ``clientEncryption: true`` and providing the name of the Kubernetes secret in ``tlsEncryptionSecretName`` containing the keystore, keystore password, truststore, and truststore password.

To create the secret in Kubernetes, update the paths to the keystore and truststore and run:
```
kubectl create secret generic cassandra-tls --from-file=keystore=./path-to-keystore \
--from-file=truststore=./path-to-truststore --from-literal=keystore-password=cassandra \
--from-literal=truststore-password=cassandra
```

In this example, the name of the secret is ``cassandra-tls``.


[Helm chart source](https://github.com/bitnami/charts/tree/master/bitnami/cassandra)

## Redis
[Redis](https://redis.io)


## Prometheus
[Prometheus](https://prometheus.io)


## Grafana
[Grafana](https://grafana.com)


## Provisioner
[Helm chart source](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner/blob/master/helm/provisioner/values.yaml)
