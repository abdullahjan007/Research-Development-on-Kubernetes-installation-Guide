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


# Question: kaal ko mainey ye installation tou kr li thi... but mujhe sara process samjhao kay ye work kaisey kr rha ha like kaisey chal rha ha mera kubernetes.

Let’s break this down step by step, taake tumhe clearly samajh aa jaaye ke tumhara Kubernetes cluster actually chal kaise raha hai 🧠👇

⚙️ 1. Tumhara Base System — Windows + WSL2

Tum Windows use kar rahe ho, jisme:

Docker Desktop installed hai ✅

Docker Desktop internally WSL2 (Ubuntu) ka use karta hai Linux containers run karne ke liye.

Toh jab tum docker command Ubuntu ke andar chalate ho,<br></br>
wo actually Windows ke Docker daemon se baat kar raha hota hai —<br></br>
via Unix socket: /var/run/docker.sock.<br></br>

So:

Windows → Docker Desktop → WSL2 Ubuntu (Linux layer)

🧩 2. Kind (Kubernetes in Docker) kya karta hai?

Kind = Kubernetes IN Docker<br></br>
Matlab Kind ek real Kubernetes cluster banata hai,<br></br>
lekin ye har node (control-plane ya worker) ko ek Docker container ke form me run karta hai.<br></br>

Tumhara setup:

cka-lab-prep-control-plane  → ek container (master)<br></br>
cka-lab-prep-worker         → ek container (worker1)<br></br>
cka-lab-prep-worker2        → ek container (worker2)<br></br>


Yani, ye teeno “virtual machines” nahi hain — ye Docker containers hain,<br></br>
jo apas me Docker’s virtual network bridge ke through connected hain.<br></br>

🧠 3. Inside the Control Plane Container

Control-plane container ke andar ye processes chal rahe hote hain (as separate pods):

Component	Role<br></br>
kube-apiserver	API endpoint (kubectl yahan connect karta hai)<br></br>
etcd	Key-Value store (Kubernetes ka database)<br></br>
kube-scheduler	Pods ko nodes par assign karta hai<br></br>
kube-controller-manager	Cluster state manage karta hai<br></br>
kubelet	Node-level agent<br></br>
kube-proxy	Networking component<br></br>

Yani, ye container tumhara “master node” ban gaya.<br></br>

🔗 4. Worker Containers ka role

Do worker containers (cka-lab-prep-worker, cka-lab-prep-worker2) ke andar:

kubelet

kube-proxy

container runtime (containerd)

run karte hain.

Ye control-plane se connect hote hain via internal Docker network.

🌐 5. Communication Flow

When you run:

kubectl get nodes


Then flow is:

kubectl → kubeconfig context (kind-cka-lab-prep)
      ↓
kube-apiserver (control-plane container)
      ↓
etcd (cluster state)
      ↓
Response: List of Nodes (control-plane + 2 workers)


So everything you do (kubectl apply, create, get pods)<br></br>
actually goes through kube-apiserver inside that container.

💾 6. Where Cluster Data Lives

All cluster state (pods, deployments, services, secrets, etc.)<br></br>
stored inside etcd, which lives at:

/var/lib/etcd/member


inside the control-plane container.

When you ran:

etcdctl snapshot save /var/lib/etcd/snapshot.db


you literally backed up the entire cluster database as a file.

⚡ 7. Kubelet Static Pods in Kind

Kind ke control-plane me /etc/kubernetes/manifests/ me .yaml files hoti hain:

etcd.yaml<br></br>
kube-apiserver.yaml<br></br>
kube-scheduler.yaml<br></br>
kube-controller-manager.yaml


Ye “static pod manifests” hain.<br></br>
kubelet inhe dekhta hai aur automatically start karta hai.<br></br>
Agar file me koi change karo → pod restart ho jaata hai (Kind me systemctl nahi chalta).

🧰 8. Why Kubectl Works Everywhere

kubectl tumhare Ubuntu system me installed hai,<br></br>
aur iska config file ye hota hai:

~/.kube/config


Jisme kind ne automatically context add kar diya:

kind-cka-lab-prep → https://127.0.0.1:port


kubectl us endpoint par API requests bhejta hai,<br></br>
jo Docker network ke andar control-plane container ke kube-apiserver par forward hota hai.

🚀 9. Whole Architecture Overview (Diagram Mental Model)<br></br>
+------------------------------------------------------+
| Windows 11                                           |
|   └── Docker Desktop                                 |
|         └── WSL2 Ubuntu (Linux)                      |
|              ├── kindest/node:v1.34.0 containers     |
|              │     ├── control-plane (master)        |
|              │     │    ├── kube-apiserver           |
|              │     │    ├── etcd                     |
|              │     │    ├── controller-manager        |
|              │     │    └── scheduler                |
|              │     ├── worker1                       |
|              │     └── worker2                       |
|              └── kubectl (CLI, outside containers)   |
+------------------------------------------------------+

✅ 10. Why This Setup Is Perfect for CKA

Because it behaves just like a real Kubernetes cluster:

Real API server, scheduler, etcd

Multi-node networking

Persistent state in etcd

Static pod manifests in /etc/kubernetes/manifests

Works with all kubectl and etcdctl commands

Only difference: sab kuch Docker containers ke andar ho raha hai,<br></br>
VMs ke andar nahi — that’s why it’s lightweight & fast ⚡
 
