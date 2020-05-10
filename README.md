## ssl-dist-optfile-nsn6lRs3dqo
https://groups.google.com/forum/#!topic/rabbitmq-users/nsn6lRs3dqo

## Scenario

Set up TLS-encrypted distributed Erlang in a two (or more) node cluster. In this example, the node names will be:

```
rabbit-1@shostakovich
rabbit-2@shostakovich2
```

Since both nodes are running on the same host and using the same `epmd` process, I must differentiate the first part of the node name (`rabbit-1` and `rabbit-2`). If you have two separate machines, you can use the node name `rabbit@HOSTNAME`. A lot of the hassle in the following configuration is due to running two nodes on the same host. Names and ports can't conflict, and non-default paths must be specified using environment variables. In a production system, configuration would be in `/etc/rabbitmq` and `systemd` would start RabbitMQ.

## `/etc/hosts`

```
127.0.0.1	localhost	shostakovich    shostakovich2
```

## Clone this project

```
git clone https://github.com/lukebakken/ssl-dist-optfile-nsn6lRs3dqo.git
cd ssl-dist-optfile-nsn6lRs3dqo
git submodule update --init
```

## Generate certs

```
cd tls-gen/basic

make CN=shostakovich
cp -v result/* ../../shostakovich

make CN=shostakovich2
cp -v result/* ../../shostakovich2
```

The certs for `shostakovich` and `shostakovich2` will be signed by their own self-signed root CA cert. To ensure these nodes trust each other, create a concatenated file like this:

```
cat shostakovich/ca_certificate.pem shostakovich2/ca_certificate.pem > ca_certificates.pem
```

## RabbitMQ configuration files

You will have to edit these files to match your environment because they contain specific paths from my environment:

```
shostakovich/rabbitmq.conf
shostakovich/rabbitmq-env.conf
shostakovich/ssl_dist.config

shostakovich2/rabbitmq.conf
shostakovich2/rabbitmq-env.conf
shostakovich2/ssl_dist.config
```

## "Install" RabbitMQ

I have Erlang in my PATH, and I am using the `generic-unix` archive, extracted locally:

```
mkdir -p ~/issues/rmq-generic-unix
cd ~/issues/rmq-generic-unix/
curl -LO https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.3/rabbitmq-server-generic-unix-3.8.3.tar.xz
tar xf rabbitmq-server-generic-unix-3.8.3.tar.xz
cd rabbitmq_server-3.8.3
```

## Enable plugins

```
cd /path/to/extracted/rabbitmq_server-3.8.3

# Note: this will enable plugins for both nodes, since they will both read the following file:
# ./etc/rabbitmq/enabled_plugins

./sbin/rabbitmq-plugins enable rabbitmq_peer_discovery_common rabbitmq_management rabbitmq_top
```

## Starting the nodes

* `shostakovich`

In one terminal -

```
cd /path/to/extracted/rabbitmq_server-3.8.3

RABBITMQ_CONFIG_FILE="$HOME/issues/rabbitmq-users/ssl-dist-optfile-nsn6lRs3dqo/shostakovich/rabbitmq.conf" \
    RABBITMQ_CONF_ENV_FILE="$HOME/issues/rabbitmq-users/ssl-dist-optfile-nsn6lRs3dqo/shostakovich/rabbitmq-env.conf" \
    RABBITMQ_ALLOW_INPUT=true \
    ./sbin/rabbitmq-server
```

* `shostakovich2`

In another terminal -

```
cd /path/to/extracted/rabbitmq_server-3.8.3

RABBITMQ_CONFIG_FILE="$HOME/issues/rabbitmq-users/ssl-dist-optfile-nsn6lRs3dqo/shostakovich2/rabbitmq.conf" \
    RABBITMQ_CONF_ENV_FILE="$HOME/issues/rabbitmq-users/ssl-dist-optfile-nsn6lRs3dqo/shostakovich2/rabbitmq-env.conf" \
    RABBITMQ_ALLOW_INPUT=true \
    ./sbin/rabbitmq-server
```

## Querying node status

```
cd /path/to/extracted/rabbitmq_server-3.8.3

# query as though we're on node shostakovich
RABBITMQ_CONF_ENV_FILE="$HOME/issues/rabbitmq-users/ssl-dist-optfile-nsn6lRs3dqo/shostakovich/rabbitmq-env.conf" \
    ./sbin/rabbitmqctl -n rabbit-1@shostakovich status

# query as though we're on node shostakovich2
RABBITMQ_CONF_ENV_FILE="$HOME/issues/rabbitmq-users/ssl-dist-optfile-nsn6lRs3dqo/shostakovich2/rabbitmq-env.conf" \
    ./sbin/rabbitmqctl -n rabbit-2@shostakovich2 status
```
