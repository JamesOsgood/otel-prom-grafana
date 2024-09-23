# otel-prom-grafana
Use Docker Compose to build containers for otl scraping Atlas metrics

# Install pre reqs on ec2

Install git
`sudo yum install git`

Install docker and compose plugin
`sudo yum install docker`

`sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s | tr '[:upper:]' '[:lower:]')-$(uname -m) -o /usr/bin/docker-compose && sudo chmod 755 /usr/bin/docker-compose && docker-compose --version`

`sudo mv /usr/bin/docker-compose /usr/libexec/docker/cli-plugins/`

To start and use docker you need to be root or else you will get an error ( there is a workaround to this, but I haven't implemented it)

`sudo systemctl start docker`

# Set up an API key and rules in Atlas
- Log in to your MongoDB Atlas account.
- Navigate to the "Access Manager" section in the chosen project
- Create a new API key with the necessary permissions.
- Note down the API key and the public key.
- Set up the necessary IP whitelist rules to allow access from your EC2 instances.


# Open ports on the ec2 Instance

- port 3000 on the Prometheus / Grafana server. Connect to Grafana using `http://<ip>:3000/grafana`
- port 9090 on the Prometheus / Grafana server to allow the otel server to push metrics into prometheus
- port 8888 on the Otel server to allow Prometheus to scrape otel metrics

# Hack the config files

On the otel connector machine

```yaml
otlphttp/prometheus:
    endpoint: "http://prometheus:9090/api/v1/otlp"
```

line in `src\otel-collector\otel-collector-config.yaml` to point to the ec2 instance running prometheus. You'll need to open port 9090 on the prometheus ec 2 as well

On the prometheus machine, change the

```yaml
scrape_configs:
- job_name: otel-collector
  static_configs:
  - targets:
    - 'otel_collector_atlas:8888'
```

line in `src\prometheus\prometheus-config.yaml` to point to the ec2 instance running otel. You'll need to open port 888 on the otel ec2 as well


# Run the containers

`sudo docker compose --profile prom up --detach` 

will start Prometheus and Grafana

`sudo docker compose --profile otel up --detach` 

will start the OpenTelemetry Collector

# Debugging
- Otel Collector: You can check if the Otel Collector is accessing your instance by going to the API key page and looking at the last access time
- Otel Collector: Check `http://<otel ip addr>:8888/metrics`.
- Prometheus: Check `http://<prom ip addr>:9090/tsdb-status`. You should see metrics starting with `mongdb_atlas_`
