# swarmstack/loki

Docker compose file for Grafana Loki, like Prometheus but for logs. Requires Docker swarm.

## INSTALLATION

On every one of your Docker nodes, create a directory and cron job that will populate container names:

```
mkdir -p /etc/promtail/targets
crontab -e
```

and add the following line to your root cron (or cron of a user that can execute /bin/docker):

```
* * * * *  docker ps --no-trunc --format '- targets: ["{{.ID}}"]\n  labels:\n    container_name: "{{.Names}}"' > /etc/promtail/targets/promtail-targets.yaml
```

Non-[swarmstack](https://github.com/swarmstack/swarmstack) users will need to edit promtail-docker-config.yaml and change the clients url from http://loki:3100/.. to the IP or DNS of one of your Docker nodes.

## USAGE

Finally, after adding the above directory and cron job to all of your Docker hosts:

```
docker stack deploy -c docker-compose.yml loki
```

[swarmstack](https://github.com/swarmstack/swarmstack) users should use docker-compose-swarmstack.yml above instead.

---

Log into Grafana as an administrator, and add a new Data Source (Configuration > Data Sources). You should only need to set the Name (Loki) and provide Grafana a URL to the Loki service. swarmstack users should set this to http://loki:3100 - otherwise change 'loki' to the IP or DNS of one of your Docker nodes.

Logs from your container stdout and stderr will now get populated into Loki by the promtail containers running on each Docker node. If you have a service running in a container which doesn't support writing it's output to stdout or stderr of the container (visible using "docker logs container_id" or "docker service logs service_id"), see [https://docs.docker.com/config/containers/logging/](https://docs.docker.com/config/containers/logging/) which explains what the official nginx container does by symbolically linking it's own access and error log files to the container's /dev/stdout and /dev/stderr files instead.
