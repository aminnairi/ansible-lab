# ansible-lab

Test lab for running NGINX on multiple servers locally using Docker for provisioning & Ansible for deployment

## Requirements

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Ansible](https://www.ansible.com/)
- [Bash](https://www.gnu.org/software/bash/)

## Setup the Docker Compose environment

```bash
touch docker-compose.override.yaml
```

```yaml
services:
  server1:
    build:
      args:
        SSH_KEY_PUBLIC: "YOUR_PUBLIC_SSH_KEY_HERE"
  server2:
    build:
      args:
        SSH_KEY_PUBLIC: "YOUR_PUBLIC_SSH_KEY_HERE"
```

## Build the lab

```bash
docker compose build
```

> *Note: This step is only needed if you didn't previously build the Docker Compose services. If you update the `docker/Dockerfile` file, you'll need to run this command again as running the Docker Compose services does not necessary implies building the services again.*

## Start the lab

```bash
docker compose up --detach
```

> *Note: This is only needed here because our servers are provisioned using Docker & Docker Compose, in a real-life scenario, your servers would probably be already provisionned before you try to deploy to them.*

## Get the servers' IP addresses

```bash
# Header for the inventory
echo "[webservers]" > ./ansible/inventory
# IP address setup for the first web server defined in the docker-compose.yaml
docker container inspect server1 | grep IPAddress | tail -1 | cut -d " " -f 22 | sed 's/"\(.*\?\)",/\1/' >> ./ansible/inventory
# IP address setup for the second web server defined in the docker-compose.yaml
docker container inspect server2 | grep IPAddress | tail -1 | cut -d " " -f 22 | sed 's/"\(.*\?\)",/\1/' >> ./ansible/inventory
```

> *Note: This step is only needed in the context of our lab, you should probably have the list of your created server's IP address either by peeking at your cloud provider's administration interface or by another tool that auto generates them like Terraform. Only run this once per Docker Compose services start or restart.*

## Run the playbook

```bash
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i ./ansible/inventory ./ansible/playbook.yaml
```

> *Note: The first time our computer connects to an SSH server, it will ask for confirmation for adding the server key to the list of known hosts, this is bad for automation purposes because if we have a hundred servers to deploy, it will keep asking this for the whole hundreds! Feel free to remove this environment variable if you need extra security steps on servers that you don't own and trust.*

## Check the result

```bash
curl http://$SERVER1_IP
curl http://$SERVER2_IP
```

> *Note: Replace `$SERVER1_IP` is the IP address of the `server1` and `$SERVER2_IP` is the IP address of the `server2`.*

## Stop the lab

```bash
docker compose down --remove-orphans --volumes --timeout 0
```

## Clean the know hosts

```bash
sed -i "/$SERVER1_IP/d" $HOME/.ssh/known_hosts
sed -i "/$SERVER2_IP/d" $HOME/.ssh/known_hosts
```

> *Note: Replace `$SERVER1_IP` is the IP address of the `server1` and `$SERVER2_IP` is the IP address of the `server2`. This step is optional if you don't want to clean your `known_hosts` file.*
