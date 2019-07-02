# swarmstack/loki

Docker compose file for Grafana Loki, like Prometheus but for logs. Requires Docker swarm.

## INSTALLATION

On every Docker swarm node, first install Loki's Docker log driver plugin (it's important to install the plugin before modifying /etc/docker/daemon.json)

```
docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions
```

Next, add the following (at least the log-driver and log-opts below) to the file /etc/docker/daemon.json on each Docker swarm node:

```
{
    "debug" : false,
    "log-driver": "loki",
    "log-opts": {
        "loki-url": "http://SWARM-FQDN-OR-IP:3100/api/prom/push",
        "loki-batch-size": "400"
    }
}
```

Finally, restart the Docker daemon on each swarm node (draining nodes first if necessary) to pick up the logging change:

```
docker node update --availbility drain node1.fqdn
systemctl restart docker
docker node update --availbility active node1.fqdn
```

## USAGE

```
docker stack deploy -c docker-compose.yml loki
```

[swarmstack](https://github.com/swarmstack/swarmstack) users should use docker-compose-swarmstack.yml above instead.

---

Log into Grafana as an administrator, and add a new Data Source (Configuration > Data Sources). You should only need to set the Name (Loki) and provide Grafana a URL to the Loki service. swarmstack users should set this to http://loki:3100 - otherwise change 'loki' to the IP or DNS of one of your Docker nodes.

Logs from your container stdout and stderr will now get populated into Loki by the promtail containers running on each Docker node. If you have a service running in a container which doesn't support writing it's output to stdout or stderr of the container (visible using "docker logs container_id" or "docker service logs service_id"), see [https://docs.docker.com/config/containers/logging/](https://docs.docker.com/config/containers/logging/) which explains what the official nginx container does by symbolically linking it's own access and error log files to the container's /dev/stdout and /dev/stderr files instead.
