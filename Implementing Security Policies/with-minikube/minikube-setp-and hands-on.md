- --enable-admission-plugins=

sudo get ns


# install minikube on vm

## Install Minikube on VM / Ec2 Instance: 


We will launch an EC2 Medium Instance (2vCPU) and install Minikube on it. To install Minikube, Docker is a prerequisite.
The bellow command needs to install for the Deployment server

```bash
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
sudo snap install kubectl --classic
minikube start --driver=docker

# verify minikube setup or not
kubectl get pods -n kube-system

```
# vim install
```bash
sudo apt-get update
sudo apt-get install -y vim

```


# NamespaceAutoProvision policy hands-on lab

# check 

```bash
kubectl get pods -n kube-system
kubectl get pods -n kube-system -l component=kube-apiserver -o yaml | grep enable-admission-plugins

minikube ssh
docker@minikube:~$ cd /etc/kubernetes/manifests/
docker@minikube:/etc/kubernetes/manifests$ ls -lrt
total 20
-rw------- 1 root root 2635 Sep  4 06:19 etcd.yaml
-rw------- 1 root root 4099 Sep  4 06:19 kube-apiserver.yaml
-rw------- 1 root root 3418 Sep  4 06:19 kube-controller-manager.yaml
-rw------- 1 root root 1657 Sep  4 06:19 kube-scheduler.yaml

docker@minikube:~$ sudo vim /etc/kubernetes/manifests/kube-apiserver.yaml
docker@minikube:~$ kubectl get pods -n kube-system
-bash: kubectl: command not found
docker@minikube:~$ logout
ubuntu@ip-172-31-41-242:~$ kubectl get pods -n kube-system

ubuntu@ip-172-31-41-242:~$ kubectl get pods -n kube-system -l component=kube-apiserver -o yaml | grep enable-admission-plugins
      - --enable-admission-plugins=NamespaceLifecycle,NamespaceAutoProvision
ubuntu@ip-172-31-41-242:~$ sudo vim auto-prov.yml
ubuntu@ip-172-31-41-242:~$ ls
auto-prov.yml  minikube-linux-amd64  snap
ubuntu@ip-172-31-41-242:~$ cat auto-prov.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
  namespace: auto-created-ns  # 🎯 Doesn't exist yet!
spec:
  containers:
  - name: nginx
    image: nginx
ubuntu@ip-172-31-41-242:~$ kubectl get ns
NAME              STATUS   AGE
default           Active   15m
kube-node-lease   Active   15m
kube-public       Active   15m
kube-system       Active   15m
ubuntu@ip-172-31-41-242:~$ kubectl apply -f auto-prov.yml
pod/nginx-demo created
ubuntu@ip-172-31-41-242:~$ kubectl get ns
NAME              STATUS   AGE
auto-created-ns   Active   5s
default           Active   16m
kube-node-lease   Active   16m
kube-public       Active   16m
kube-system       Active   16m
ubuntu@ip-172-31-41-242:~$ kubectl get pods
No resources found in default namespace.
ubuntu@ip-172-31-41-242:~$ kubectl get pods -n auto-created-ns
NAME         READY   STATUS    RESTARTS   AGE
nginx-demo   1/1     Running   0          24s
ubuntu@ip-172-31-41-242:~$

```


