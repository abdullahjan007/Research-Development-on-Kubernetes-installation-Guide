# Research-Development-on-Kubernetes-installation-Guide

wsl --install -d Ubuntu

lsb_release -a
 
sudo apt update && sudo apt upgrade -y
 
sudo apt update && sudo apt upgrade -y
 
sudo apt install -y curl apt-transport-https ca-certificates gnupg lsb-release

(is point pay docker desktop pay jana ha phir resources pay jana.. phir wsl integrataion pay jana aur phir ubunutu ko enable kr dena, Apply and restart)
 
docker ps

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

kubectl version --client

 
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
 
chmod +x ./kind

sudo mv kind /usr/local/bin/

kind version
 

kubectl config get-contexts

kubectl config use-context kind-cka-lab

kubectl get nodes

 
sudo usermod -aG docker $USER

 
Band kar do current Ubuntu terminal

Phir PowerShell me likho 👇
wsl --shutdown
Ab dobara Start Menu → Ubuntu open karo
(ye fresh WSL session start karega)
 
groups

 
docker ps
 
nano kind-config.yaml

kind: Cluster

apiVersion: kind.x-k8s.io/v1alpha4

nodes:

  - role: control-plane

  - role: worker

  - role: worker

 
kind create cluster --name cka-lab --config kind-config.yaml

 
kubectl get nodes

kubectl config get-contexts
kubectl config use-context kind-cka-lab
kubectl get nodes

docker exec -it cka-lab-prep-control-plane bash
 
apt update
 
apt install -y etcd-client
 
etcdctl version

export ETCDCTL_API=3<br></br>
export ETCDCTL_CACERT="/etc/kubernetes/pki/etcd/ca.crt"<br></br>
export ETCDCTL_CERT="/etc/kubernetes/pki/etcd/server.crt"<br></br>
export ETCDCTL_KEY="/etc/kubernetes/pki/etcd/server.key"<br></br>
 
etcdctl snapshot save /var/lib/etcd/snapshot.db
 
 
etcdctl snapshot status /var/lib/etcd/snapshot.db -w table

 
 
