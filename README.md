# Ansible Docker Swarm Cluster

Create and deploy an n-node docker swarm cluster in minutes, with [Traefik](https://containo.us/traefik/) as a reverse proxy and load balancer, [Portainer](https://www.portainer.io/) to manage containers and [Jenkins](https://www.jenkins.io/) as a CI/CD solution (this is a [customized](https://github.com/rodrigoegimenez/jenkins-docker) Jenkins docker image whose Jobs can interact with docker).

## High Availability Configuration

This setup supports multiple master nodes for high availability. The first master node initializes the swarm, and additional master nodes join as managers.

## Prerequisites

- Three or more servers running Ubuntu 20.04 connected through a private network (tests were done using [GCP](https://console.cloud.google.com/)).
- [Ansible](https://www.ansible.com/).
- SSH access to these servers using your public key (you can achieve this by simply using `ssh-copy-id <hostname>`).

## Preparation

First clone this repo to your local machine

```bash
git clone https://github.com/wildanmr/ansible-docker-swarm.git
cd ansible-docker-swarm
```

Then edit the `hosts` file by replacing the IPs of your master servers and replacing/adding as many worker nodes as you wish to join your cluster. Then create an `extra-vars.yml` using the example as a guide.

Example hosts file for HA setup:
```
[masters]
master1 ansible_host=1.2.3.4
master2 ansible_host=1.2.3.5
master3 ansible_host=1.2.3.6

[workers]
node-1 ansible_host=2.3.4.5
node-2 ansible_host=3.4.5.6

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

## Running

Once finished editing your `hosts` file you can run all the required commands using the provided `Makefile` that will launch the appropriate playbook:

```bash
make swarm-cluster
```

This command will start running the imported playbooks in the order they appear:

```ansible
- import_playbook: docker-dependencies.yml
- import_playbook: main.yml
- import_playbook: masters.yml
- import_playbook: workers.yml
- import_playbook: traefik/traefik.yml
- import_playbook: portainer/portainer.yml
- import_playbook: jenkins/jenkins.yml
```

After a couple minutes the cluster will be up and initialized and ansible will start to install traefik and the other services asking a prompt for user, password and email (for traefik), and for the domain names for all three services (to use with traefik reverse proxy). Make sure you point these (sub)domains to the first master's public IP address.

A couple of more minutes and you should have your cluster up and running.

## Acknowledgement

This idea is heavily influenced by [Docker Swarm Rocks](https://dockerswarm.rocks/). Traefik and portainer's compose files were obtained from this guide.

Another source of inspiration, specially for the ansible part, was [this](https://www.digitalocean.com/community/tutorials/how-to-create-a-kubernetes-cluster-using-kubeadm-on-ubuntu-18-04) DigitalOcean's guide to create a Kubernetes Cluster.
