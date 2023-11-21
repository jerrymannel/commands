
⛔️ Commands might be for older version. Please check software versions before running ⛔️

- [UTILS](#utils)
- [Golang](#golang)
- [Docker](#docker)
	- [CRI-Dockerd](#cri-dockerd)
- [Kubernetes](#kubernetes)
	- [Optional](#optional)
	- [Setup](#setup)
- [MongoDB](#mongodb)
	- [Common](#common)
	- [MongoDB 6.0](#mongodb-60)
	- [MongoDB 4.4](#mongodb-44)
- [Elastic](#elastic)
	- [ElasticSearch](#elasticsearch)
	- [Kibana](#kibana)
	- [Filebeat](#filebeat)
	- [Metricbeat](#metricbeat)
- [Jenkins](#jenkins)
- [Uninstall](#uninstall)


# UTILS

```sh
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl net-tools
```
# Golang

```sh
wget https://go.dev/dl/go1.21.4.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.21.4.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
go version
```

# Docker

```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## CRI-Dockerd

```sh
git clone https://github.com/Mirantis/cri-dockerd.git
cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
sudo install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
sudo cp -a packaging/systemd/* /etc/systemd/system
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
sudo systemctl daemon-reload
sudo systemctl enable cri-docker.service
sudo systemctl enable --now cri-docker.socket
```

# Kubernetes

```sh
sudo apt-get update
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.26/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.26/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

## Optional

```sh
sudo apt-mark hold kubelet kubeadm kubectl
```

## Setup

```sh
sudo systemctl start kubelet
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl taint nodes --all node-role.kubernetes.io/master-
```

# MongoDB

## Common

```sh
sudo apt-get install gnupg curl
sudo systemctl status mongod
sudo systemctl start mongod
sudo systemctl stop mongod
sudo systemctl restart mongod

replication:
  replSetName: rs0
```

## MongoDB 6.0

```sh
curl -fsSL https://pgp.mongodb.com/server-6.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg \
   --dearmor

# Ubuntu 20.04
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg ] https://repo.mongodb.com/apt/ubuntu focal/mongodb-enterprise/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-enterprise-6.0.list
# Ubuntu 22.04
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg ] https://repo.mongodb.com/apt/ubuntu jammy/mongodb-enterprise/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-enterprise-6.0.list

sudo apt-get update
sudo apt-get install -y mongodb-enterprise
```

## MongoDB 4.4

```sh
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64,s390x ] http://repo.mongodb.com/apt/ubuntu focal/mongodb-enterprise/4.4 multiverse" | \
    sudo tee /etc/apt/sources.list.d/mongodb-enterprise.list
sudo apt-get update
sudo apt-get install -y mongodb-enterprise
sudo systemctl daemon-reload
sudo systemctl start mongod
```

# Elastic

## ElasticSearch

```sh
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install elasticsearch
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
```

```sh
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic

sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana

sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
sudo /usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token-here>
```

## Kibana

```sh
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install kibana

sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable kibana.service
sudo systemctl start kibana.service
```

## Filebeat

```sh
curl -L -O https://raw.githubusercontent.com/elastic/beats/8.8/deploy/kubernetes/filebeat-kubernetes.yaml
```

## Metricbeat

```sh
curl -L -O https://raw.githubusercontent.com/elastic/beats/8.8/deploy/kubernetes/metricbeat-kubernetes.yaml
```

# Jenkins

```sh
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install fontconfig openjdk-11-jre
sudo apt-get install jenkins
```

# Uninstall

```sh
sudo apt-get purge kubelet kubeadm kubectl
sudo apt-get purge docker docker-engine docker.io containerd runc
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
sudo apt-get purge mongodb-enterprise*
sudo apt-get clean
```