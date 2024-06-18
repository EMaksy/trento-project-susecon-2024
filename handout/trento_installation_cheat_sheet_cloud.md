# Manual Installation Cheat Sheet on the Cloud.

This is a handout for the Manual installation guide for [Trento](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md).
As the original guide covers different options of Trento installation, this is a short and compact version of all the required commands in order to install Trento.

## Manual Installation Agenda

1. [Install Trento manually on SLE 15 SP5 virtual machine](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md)
1. Install and configure [Prometheus](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#install-prometheus-optional), [PostgreSQL](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#install-postgresql) and [RabbitMQ](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#install-rabbitmq)
1. Install and configure [Trento Web and Wanda by using RPM packages](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#install-trento-using-rpm-packages)
1. Creating a [Self-Signed Certificate](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#option-1-creating-a-self-signed-certificate)
1. [Install and configure NGINX](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#install-and-configure-nginx)
1. [Accessing the Trento UI](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md#accessing-the-trento-web-ui)

## Access machine
1: Download a key for accessing lab machine.

```bash
https://paste.opensuse.org/pastes/f3aad960b1b5
```

2: Open PowerShell in Windows

3: Access Lab machine with ssh
### Connect to lab machine:


```bash
ssh trento@<<LAB_MACHINE_ADRESS>> -i "C:\Users\User\Downloads\id_rsa_susecon24"
```

Example:

```bash
ssh trento@trento.susecon24.3.com -i "C:\Users\User\Downloads\id_rsa_susecon24"
```

## Install Prometheus

### Enable Package Hub for Prometheus

```bash
sudo SUSEConnect --product PackageHub/15.5/x86_64
```

### Add the prometheus user/group

```bash
sudo groupadd --system prometheus
```

```bash
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```

### Install prometheus using zypper

```bash
sudo zypper in golang-github-prometheus-prometheus
```

### Edit prometheus config

```bash
sudo vim /etc/prometheus/prometheus.yml
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
sudo systemctl enable --now prometheus
```

Check health status

```bash
sudo systemctl status prometheus
```

## Install PostgreSQL

### Install PostgreSQL RPM

```bash
sudo zypper in postgresql-server
```

### Enable and start PostgreSQL server

```bash
sudo systemctl enable --now postgresql
```

### Switch to postgres user and activate PostgreSQL interface

```bash
sudo su - postgres
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
sudo vim /var/lib/pgsql/data/pg_hba.conf
```

Add on top of the configuration:

```bash
host   wanda                      wanda_user    0.0.0.0/0   md5
host   trento,trento_event_store  trento_user   0.0.0.0/0   md5
```

### Open network interface for Trento

```bash
sudo vim /var/lib/pgsql/data/postgresql.conf
```

Add at the beginning of the config file:

```bash
listen_addresses = '*'
```

### Restart PostgreSQL

```bash
sudo systemctl restart postgresql
```

Check status of PostgreSQL service

```bash
sudo systemctl status postgresql.service
```

## Install RabbitMQ

### Install RabbitMQ RPM

```bash
sudo zypper install rabbitmq-server
```

### Edit and configure rabbitmq config

```bash
sudo vim /etc/rabbitmq/rabbitmq.conf
```

Add on top of config:

```bash
listeners.tcp.default = 5672
```

### Start rabbitmq

```bash
sudo systemctl enable --now rabbitmq-server
```

Check health status

```bash
sudo systemctl status rabbitmq-server
```

### Configure RabbitMQ

Create a new RabbitMQ user

```bash
sudo rabbitmqctl add_user trento_user trento_user_password
```

### Create a virtual host for rabbitmq

```bash
sudo rabbitmqctl add_vhost vhost
```

### Set permissions for Trento RabbitMQ user

```bash
sudo rabbitmqctl set_permissions -p vhost trento_user ".*" ".*" ".*"
```

## Install Trento using RPM packages

### Install Trento-Web/Wanda RPM

```bash
sudo zypper install trento-web trento-wanda
```

### Edit Trento Web configuration

Edit a trento web configuration

```bash
sudo vim /etc/trento/trento-web
```

Copy and paste this configuration to `/etc/trento/trento-web`

> **Note** : Adjust TRENTO_WEB_ORIGIN env `trento.susecon24.<<ENTER_YOUR_NUMBER_HERE>>.com` to your dedicated number.
>
> Example: TRENTO_WEB_ORIGIN=trento.susecon24.1.com

```
AMQP_URL=amqp://trento_user:trento_user_password@localhost:5672/vhost
DATABASE_URL=ecto://trento_user:web_password@localhost/trento
EVENTSTORE_URL=ecto://trento_user:web_password@localhost/trento_event_store
ENABLE_ALERTING=false
CHARTS_ENABLED=true
PROMETHEUS_URL=http://localhost:9090
ADMIN_USER=admin
ADMIN_PASSWORD=test1234
ENABLE_API_KEY=false
PORT=4000
TRENTO_WEB_ORIGIN=trento.susecon24.<<ENTER_YOUR_NUMBER_HERE>>.com
SECRET_KEY_BASE=rd9yv6K3DLqovfCzA+qjX6T2YM1gVYVxl+e/fx3gXWHc6WFBkF3Fi9AEEsZGubE3
ACCESS_TOKEN_ENC_SECRET=ejkJuJSrzX9QL2VD5Lb2epho2pCRhDSqpfKASXEtgvGye0qDltJrU1ZGHY8oim2E
REFRESH_TOKEN_ENC_SECRET=CKdeaee2IBoQA1zxVqXlqa8a2oWYGkFPmlkBvsM0yYBa78dViwj2oGxw802QXisi
```

Create and adjust wanda configuration.

```bash
 sudo vim /etc/trento/trento-wanda
```

```
CORS_ORIGIN=http://localhost
AMQP_URL=amqp://trento_user:trento_user_password@localhost:5672/vhost
DATABASE_URL=ecto://wanda_user:wanda_password@localhost/wanda
PORT=4001
SECRET_KEY_BASE=rd9yv6K3DLqovfCzA+qjX6T2YM1gVYVxl+e/fx3gXWHc6WFBkF3Fi9AEEsZGubE3
ACCESS_TOKEN_ENC_SECRET=ejkJuJSrzX9QL2VD5Lb2epho2pCRhDSqpfKASXEtgvGye0qDltJrU1ZGHY8oim2E
```

Enable Trento server

```bash
sudo systemctl enable --now trento-web trento-wanda
```

Validate trento health status

```bash
sudo curl http://localhost:4000/api/readyz; sudo curl http://localhost:4000/api/healthz; echo
sudo curl http://localhost:4001/api/readyz; sudo curl http://localhost:4001/api/healthz; echo
```

## Move provided SSL Certificate/Key

Move SSL certificate and key to correct directory:

Move SSL key:

```bash
sudo mv <<PATH_TO_KEY>>/trento.susecon24.<<ENTER_YOUR_NUMBER_HERE>>.key /etc/ssl/private/
```

Example:

```bash
sudo mv trento.susecon24.1.key /etc/ssl/private/
```

Move SSL certificate:

```bash
sudo mv <<PATH_TO_CERTIFICATE>>/trento.susecon24.<<ENTER_YOUR_NUMBER_HERE>>.crt /etc/ssl/certs/
```

Example:

```bash
sudo mv trento.susecon24.1.crt /etc/ssl/certs/
```

## Install and configure NGINX

Install NGINX rpm

```bash
sudo zypper install nginx
```

Start NGINX

```bash
sudo systemctl enable --now nginx
```

Create and configurate Trento:

```bash
sudo vim /etc/nginx/conf.d/trento.conf
```

Create and adjust configuration file at `/etc/nginx/conf.d/trento.conf`

Adjust in NGINX config:

- server_name
- ssl_certificate
- ssl_certificate_key

```
server {
    # Redirect HTTP to HTTPS
    listen 80;
    server_name trento.susecon24.<<ENTER_YOUR_NUMBER_HERE>>.com;
    return 301 https://$host$request_uri;
}

server {
    # SSL configuration
    listen 443 ssl;
    server_name trento.susecon24.<<ENTER_YOUR_NUMBER_HERE>>.com;

    # Adjust path of certificate
    ssl_certificate /etc/ssl/certs/trento.susecon24.<<ENTER_YOUR_NUMBER_HERE>>.crt;
    ssl_certificate_key /etc/ssl/private/trento.susecon24.<<ENTER_YOUR_NUMBER_HERE>>.key;

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
sudo nginx -t
```

Reload NGINX configuration

```bash
sudo systemctl reload nginx
```

## Access Trento

Access Trento through browser by visiting `trento.susecon24.<<ENTER_YOUR_NUMBER_HERE>>.com`.

Exmaple `https://trento.susecon24.1.com

## Learn more about Trento

Thanks for following the guide, feel free to learn more about [Trento](https://www.trento-project.io/) or at the [official Trento documentation](https://documentation.suse.com/sles-sap/trento/html/SLES-SAP-trento/index.html).

### Additional Sources:

- Trento project: https://github.com/trento-project
- Trento-web: https://github.com/trento-project/web
- Trento-wanda https://github.com/trento-project/wanda
- Trento-agent: https://github.com/trento-project/agent
- Trento helm chart: https://github.com/trento-project/helm-charts
- Trento ansible: https://github.com/trento-project/ansible
- Trento documentation: https://github.com/trento-project/docs
- Trento installation guide: https://github.com/trento-project/docs/blob/main/guides/manual-installation.md
