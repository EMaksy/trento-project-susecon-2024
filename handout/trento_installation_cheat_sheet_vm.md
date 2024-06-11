# Manual Install Cheat Sheet VM SLE_15_SP5

This is a handout for the Manual installation guide for [Trento](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md).
As the original guide covers diffrent options of Trento installation, this is a short and compact version of all the required commands in order to install Trento.

## Manual Installation Agenda

1. [Install Trento manually on SLE 15 SP5 virtual machine](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md)
1. Install and configure [Prometheus](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#install-prometheus-optional), [PostgreSQL](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#install-postgresql) and [RabbitMQ](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#install-rabbitmq)
1. Install and configure [Trento Web and Wanda by using RPM packages](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#install-trento-using-rpm-packages)
1. Creating a [Self-Signed Certificate](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#option-1-creating-a-self-signed-certificate)
1. [Install and configure NGINX](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#install-and-configure-nginx)
1. [Accessing the Trento UI](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#accessing-the-trento-web-ui)

## Access machine

```bash
ssh root@<<IP_ADDRESS>>
```

## Install text editor `Vim`

```bash
zypper in vim
```

## Install Prometheus

### Enable Package hub for Prometheus

```bash
SUSEConnect --product PackageHub/15.5/x86_64
```

### Add the prometheus user/group

```bash
groupadd --system prometheus
```

```bash
useradd -s /sbin/nologin --system -g prometheus prometheus
```

### Install prometheus using zypper

```bash
 zypper in golang-github-prometheus-prometheus
```

### Edit prometheus config

```bash
 vim /etc/prometheus/prometheus.yml
```

Save configuration to `/etc/prometheus/prometheus.yml`.

```bash
global:
  scrape_interval: 30s
  evaluation_interval: 10s

scrape_configs:
  - job_name: "http_sd_hosts"
    honor_timestamps: true
    scrape_interval: 30s
    scrape_timeout: 30s
    scheme: http
    follow_redirects: true
    http_sd_configs:
      - follow_redirects: true
        refresh_interval: 1m
        url: http://localhost:4000/api/prometheus/targets
```

### Enable and start the prometheus service:

```bash
 systemctl enable --now prometheus
```

Check health status

```bash
 systemctl status prometheus
```

### Open firewall for Prometheus

```bash
firewall-cmd --zone=public --add-port=9090/tcp --permanent
firewall-cmd --reload
```

## Install PostgreSQL

### Install PostgreSQL rpm

```bash
 zypper in postgresql-server
```

### Enable and start PostgreSQL server

```bash
 systemctl enable --now postgresql
```

Check if postgres is running:
```bash
systemctl status postgresql.service 
```

### Switch to postgres user and activate PostgreSQL interface

```bash
 su - postgres
psql
```

### Create Trento database

```bash
CREATE DATABASE wanda;
CREATE DATABASE trento;
CREATE DATABASE trento_event_store;
```

### Create users

```bash
CREATE USER wanda_user WITH PASSWORD 'wanda_password';
CREATE USER trento_user WITH PASSWORD 'web_password';
```

### Grant required privileges

```bash
\c wanda
GRANT ALL ON SCHEMA public TO wanda_user;
\c trento
GRANT ALL ON SCHEMA public TO trento_user;
\c trento_event_store;
GRANT ALL ON SCHEMA public TO trento_user;
\q
```

### Exit PostgreSQL user and switch back to root user

```bash
exit
```

### Edit PostgreSQL config

```bash
 vim /var/lib/pgsql/data/pg_hba.conf
```

Add on top of the configuration:

```bash
host   wanda                      wanda_user    0.0.0.0/0   md5
host   trento,trento_event_store  trento_user   0.0.0.0/0   md5
```

### Open network interface for Trento

```bash
 vim /var/lib/pgsql/data/postgresql.conf
```

Add at the beginning of the config file.

```bash
listen_addresses = '*'
```

### Restart PostgreSQL

```bash
 systemctl restart postgresql
```

Check status of PostgreSQL service

```bash
 systemctl status postgresql.service
```

## Install RabbitMQ

### Install RabbitMQ RPM

```bash
 zypper install rabbitmq-server
```

### Edit and configure rabbitmq config

```bash
 vim /etc/rabbitmq/rabbitmq.conf
```

Add on top of config:

```bash
listeners.tcp.default = 5672
```

### Start rabbitmq

```bash
 systemctl enable --now rabbitmq-server
```

Open firwalld for RabbitMQ

```bash
firewall-cmd --zone=public --add-port=5672/tcp --permanent;
firewall-cmd --reload
```

Check health status

```bash
 systemctl status rabbitmq-server
```


### Configure RabbitMQ

Create a new RabbitMQ user

```bash
 rabbitmqctl add_user trento_user trento_user_password
```

### Create a virtual host for rabbitmq

```bash
 rabbitmqctl add_vhost vhost
```

### Set permissions for Trento RabbitMQ user

```bash
 rabbitmqctl set_permissions -p vhost trento_user ".*" ".*" ".*"
```

## Install Trento using RPM packages

### Add open build service repository

```bash
 zypper ar -f https://download.opensuse.org/repositories/devel:/sap:/trento:/factory/SLE_15_SP5/ devel:sap:trento:factory
```

### Install Trento-Web/Wanda RPM

```bash
 zypper install trento-web trento-wanda
```

### Edit Trento Web configuration

Edit a trento web configuration

```bash
 vim /etc/trento/trento-web
```

Copy and paste this configuration to `/etc/trento/trento-web`


```bash
AMQP_URL=amqp://trento_user:trento_user_password@localhost:5672/vhost
DATABASE_URL=ecto://trento_user:web_password@localhost/trento
EVENTSTORE_URL=ecto://trento_user:web_password@localhost/trento_event_store
ENABLE_ALERTING=false
CHARTS_ENABLED=true
PROMETHEUS_URL=http://localhost:9090
ADMIN_USER=admin
ADMIN_PASSWORD=test1234
ENABLE_API_KEY=true
PORT=4000
TRENTO_WEB_ORIGIN=trento.example.com
SECRET_KEY_BASE=rd9yv6K3DLqovfCzA+qjX6T2YM1gVYVxl+e/fx3gXWHc6WFBkF3Fi9AEEsZGubE3
ACCESS_TOKEN_ENC_SECRET=ejkJuJSrzX9QL2VD5Lb2epho2pCRhDSqpfKASXEtgvGye0qDltJrU1ZGHY8oim2E
REFRESH_TOKEN_ENC_SECRET=CKdeaee2IBoQA1zxVqXlqa8a2oWYGkFPmlkBvsM0yYBa78dViwj2oGxw802QXisi
```

Create and adjust wanda configuration.

```bash
  vim /etc/trento/trento-wanda
```

```bash
CORS_ORIGIN=http://localhost
AMQP_URL=amqp://trento_user:trento_user_password@localhost:5672/vhost
DATABASE_URL=ecto://wanda_user:wanda_password@localhost/wanda
PORT=4001
SECRET_KEY_BASE=rd9yv6K3DLqovfCzA+qjX6T2YM1gVYVxl+e/fx3gXWHc6WFBkF3Fi9AEEsZGubE3
ACCESS_TOKEN_ENC_SECRET=ejkJuJSrzX9QL2VD5Lb2epho2pCRhDSqpfKASXEtgvGye0qDltJrU1ZGHY8oim2E
```

Enable Trento server

```bash
systemctl enable --now trento-web trento-wanda
```

Check trento-web logs:

```bash
journalctl -fu trento-web
```
Check trento-wanda logs:

```bash
journalctl -fu trento-wnda
```

Validate trento health status

```bash
curl http://localhost:4000/api/readyz; curl http://localhost:4000/api/healthz; echo
curl http://localhost:4001/api/readyz; curl http://localhost:4001/api/healthz; echo
```

## Generate SSL KEY

Prepare SSL certificate

```bash
 openssl req -newkey rsa:2048 --nodes -keyout trento.key -x509 -days 5 -out trento.crt -addext "subjectAltName = DNS:trento.example.com"
```

Move generated keys

```bash
 mv trento.key /etc/ssl/private/trento.key
```

```bash
 mv trento.crt /etc/ssl/certs/trento.crt
```

## Install and configure NGINX

Install NGINX RPM

```bash
 zypper install nginx
```

Start NGINX

```bash
 systemctl enable --now nginx
```

Open firewalld for NGINX
´´´bash
firewall-cmd --zone=public --add-service=https --permanent
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
´´´

Create and configurate Trento:

```bash
 vim /etc/nginx/conf.d/trento.conf
```

Create configuration file at `/etc/nginx/conf.d/trento.conf`

```
server {
    # Redirect HTTP to HTTPS
    listen 80;
    server_name trento.example.com;
    return 301 https://$host$request_uri;
}

server {
    # SSL configuration
    listen 443 ssl;
    server_name trento.example.com;

    ssl_certificate /etc/ssl/certs/trento.crt;
    ssl_certificate_key /etc/ssl/private/trento.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;

    # Wanda rule
    location ~ ^/(api/checks|api/v1/checks|api/v2/checks|api/v3/checks)/  {
        allow all;

        # Proxy Headers
        proxy_http_version 1.1;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Cluster-Client-Ip $remote_addr;

        # Important Websocket Bits!
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_pass http://localhost:4001;
    }

    # Web rule
    location / {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;

        # The Important Websocket Bits!
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_pass http://localhost:4000;
    }
}
```

Validate configuration of NGINX

```bash
 nginx -t
```

Reload NGINX configuration

```bash
 systemctl reload nginx
```

## Access Trento

Get IP Adress of your vm:

```bash
ip a s
```

Edit `/etc/hosts` on your host machine

```bash
 sudo vim /etc/hosts
```

Add entry to configuration on top of the config file:

```
<<IP_ADRESS_OF_VM>>   trento.example.com
```

Access Trento through browser by visiting `trento.example.com`.

Use login data to access Trento:

- Username: admin
- Password: test1234

## Learn more about Trento

Thanks for following the guide, feel free to learn more about [Trento](https://www.trento-project.io/) or at the [official Trento documentation](https://documentation.suse.com/sles-sap/trento/html/SLES-SAP-trento/index.html).

## Populate Trento with a cluster

## Test Trento by using fake agents to see Trento in action

In order to simulate trento for this demo, we will use [barbecue  docker images](https://github.com/trento-project/barbecue) to simulate real agents installed on a host system.


Enable docker container`s module.
```
SUSEConnect --product sle-module-containers/15.5/x86_64
```

```
zypper install docker
```
```
systemctl enable --now docker
```

Copy the API key from https://trento.example.com/settings

Replace TRENTO_API_KEY=<API_KEY> with Trento's setting key in the docker commands.

Edit and start the first fake agent
```
docker run -d --hostname hana_node01 --network=host -e TRENTO_API_KEY=<API_KEY> -e TRENTO_FACTS_SERVICE_URL=amqp://trento_user:trento_user_password@localhost:5672/vhost  -e TRENTO_SERVER_URL=http://localhost:4000 ghcr.io/trento-project/barbecue-hana_node01
```
After executing the first Docker command, a new host should be added at https://trento.example.com/hosts.

Start the second fake agent host:
```
docker run -d --hostname hana_node02 --network=host -e TRENTO_API_KEY=<API_KEY> -e TRENTO_FACTS_SERVICE_URL=amqp://trento_user:trento_user_password@localhost:5672/vhost  -e TRENTO_SERVER_URL=http://localhost:4000 ghcr.io/trento-project/barbecue-hana_node02
```

After executing both commands, a cluster will be shown at 
https://trento.example.com/clusters. 

Test checks execution on the newly added Cluster

### Additional Sources:

- Trento project: https://github.com/trento-project
- Trento-web: https://github.com/trento-project/web
- Trento-wanda https://github.com/trento-project/wanda
- Trento-agent: https://github.com/trento-project/agent
- Trento helm chart: https://github.com/trento-project/helm-charts
- Trento ansible: https://github.com/trento-project/ansible
- Trento documentation: https://github.com/trento-project/docs
- Trento installation guide: https://github.com/trento-project/docs/blob/main/guides/manual-installation.md
