# Handout manual installation SUSECON 2024

This is a handout for the Manual installation guide for [Trento](https://github.com/trento-project/docs/blob/main/guides/manual-installation.md).

The main reason for this is to provide references and helpers to make the installation easier to follow.

## Manual Installation Agenda

1. [Install Trento manually on SLE 15 SP5 virtual machine](https://github.com/trento-project/docs/blob/update_manual_install_docs/guides/manual-installation.md#installation)
2. Install and configure [Prometheus](https://github.com/trento-project/docs/blob/update_manual_install_docs/guides/manual-installation.md#install-prometheus-optional), [PostgreSQL](https://github.com/trento-project/docs/blob/update_manual_install_docs/guides/manual-installation.md#install-postgresql) and [RabbitMQ](https://github.com/trento-project/docs/blob/update_manual_install_docs/guides/manual-installation.md#install-rabbitmq)
3. Install and configure [Trento Web and Wanda by using RPM packages](https://github.com/trento-project/docs/blob/update_manual_install_docs/guides/manual-installation.md#install-trento-server-components)
4. Creating a [Self-Signed Certificate](https://github.com/trento-project/docs/blob/update_manual_install_docs/guides/manual-installation.md#option-1-creating-a-self-signed-certificate)
5. [Install and configure NGINX](https://github.com/trento-project/docs/blob/update_manual_install_docs/guides/manual-installation.md#install-and-configure-nginx)
6. [Accessing the Trento UI](https://github.com/trento-project/docs/blob/update_manual_install_docs/guides/manual-installation.md#accessing-the-trento-web-ui)
7. [Run Demo Trento Agents to populate the web UI](https://github.com/EMaksy/trento-project-susecon-2024?tab=readme-ov-file#test-trento-by-using-fake-agents-to-simulate-trento)


## Add trento-web and trento-wanda repository
Currently, the trento rpm's are in the release process and will be soon available for SLE15 SP5. As this process is not finished, add the alternative source repository.
```
sudo zypper ar -f https://download.opensuse.org/repositories/devel:/sap:/trento:/factory/SLE_15_SP5/ devel:sap:trento:factory
```
After adding the Open Build Service libary with trento's rpm, it is a must to refresh the repository:
```
sudo zypper ref
```

```As next step the user is asked about the new Repository:

New repository or package signing key received:

  Repository:       devel:sap:trento:factory
  Key Fingerprint:  6218 F543 A429 F63F 1401 CD8F 809F 4A42 31FF 744A
  Key Name:         devel:sap OBS Project <devel:sap@build.opensuse.org>
  Key Algorithm:    RSA 2048
  Key Created:      Sat Jan 28 19:38:18 2023
  Key Expires:      Mon Apr  7 20:38:18 2025
  Rpm Name:         gpg-pubkey-31ff744a-63d56b9a



    Note: Signing data enables the recipient to verify that no modifications occurred after the data
    were signed. Accepting data with no, wrong or unknown signature can lead to a corrupted system
    and in extreme cases even to a system compromise.

    Note: A GPG pubkey is clearly identified by its fingerprint. Do not rely on the key's name. If
    you are not sure whether the presented key is authentic, ask the repository provider or check
    their web site. Many providers maintain a web page showing the fingerprints of the GPG keys they
    are using.

Do you want to reject the key, trust temporarily, or trust always? [r/t/a/?] (r):
```

Press t to trust and install Trento rpm's

```zypper install trento-web trento-wanda```

## Generate and Configure Secret Keys for Trento
The next step requires to configure trento-web and trento-wanda.
To create a file with secrets in the current directory execute: 

```
{ 
  echo "SECRET_KEY_BASE=$(openssl rand -out /dev/stdout 48 | base64)"; 
  echo "ACCESS_TOKEN_ENC_SECRET=$(openssl rand -out /dev/stdout 48 | base64)"; 
  echo "REFRESH_TOKEN_ENC_SECRET=$(openssl rand -out /dev/stdout 48 | base64)"; 
} > secrets.txt && \
file_path="$(pwd)/secrets.txt" && \
echo -e "Trento web and wanda keys have been generated and saved to \033[0;31m$file_path\033[0m"
```
Output the secrets: 
```
cat $file_path
```
Expected output:
```
SECRET_KEY_BASE=UdqS94jVOBE38c8qzEHUxFVOuLLCZWO/CdIeMDfFDzoVwgUornIwyuwoLAaPyd1M
ACCESS_TOKEN_ENC_SECRET=mTS7b61Wa95sJkJRMq72+SkankPdAk1KhjHou5mdNIup4sBnLzcg3SCaqbxwYxPD
REFRESH_TOKEN_ENC_SECRET=X1l8SSk4UR6lZDZJkXdxAwDOH0mr0QZ9f6Lm0KFfhTf/fTnqe40d4TCvD+EjHT6q
```

## Trento Login data
If you followed the manual installation guide, this are the login data for trento-web which is reachable under https://trento.example.com/

```
username: admin
password: test1234
```


## Test Trento by using fake agents to simulate trento
In order to simulate trento for this demo, we will use [barbecue  docker images](https://github.com/trento-project/barbecue) to simulate real agents installed on a host system.

First check if docker is installed: 
```
docker --version
```


If docker is missing then enable the container`s module
```
SUSEConnect --product sle-module-containers/15.5/x86_64
```

```
zypper install docker
```
```
systemctl enable --now docker
```

Open trento web console by visiting https://trento.example.com/settings and copy the API key.

>Note: Replace TRENTO_API_KEY=<API_KEY> with Trento's setting key in the docker commands.

Start the first fake agent
```
sudo docker run -d --hostname hana_node01 --network=host -e TRENTO_API_KEY=<API_KEY> -e TRENTO_FACTS_SERVICE_URL=amqp://trento_user:trento_user_password@localhost:5672/vhost  -e TRENTO_SERVER_URL=http://localhost:4000 ghcr.io/trento-project/barbecue-hana_node01
```
After executing the first Docker command, a new host should be added at https://trento.example.com/hosts.

Start the second fake agent host:
```
sudo docker run -d --hostname hana_node02 --network=host -e TRENTO_API_KEY=<API_KEY> -e TRENTO_FACTS_SERVICE_URL=amqp://trento_user:trento_user_password@localhost:5672/vhost  -e TRENTO_SERVER_URL=http://localhost:4000 ghcr.io/trento-project/barbecue-hana_node02
```

After executing both commands, a cluster will be shown at 
https://trento.example.com/clusters. 

Test checks execution on the newly added Cluster

# Sources: 

- https://github.com/trento-project
- https://github.com/trento-project/web
- https://github.com/trento-project/wanda
- https://github.com/trento-project/agent
- https://github.com/trento-project/docs
- https://github.com/trento-project/helm-charts
- https://github.com/trento-project/ansible
- https://github.com/trento-project/docs/blob/main/guides/manual-installation.md
- https://www.trento-project.io/
