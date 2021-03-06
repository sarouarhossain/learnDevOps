## In All Nodes
* Enable some kernel modules. `containerd.conf` is going to instruct the OS to load `overlay` and `br_netfilter` kernel modules on startup. Those kernel module we are gonna need for container runtime containerd. 
```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay  
br_netfilter  
EOF
```

```
sudo modprobe overlay
sudo modprobe br_netfilter
```  
-> load the upper modules immediately.


* Set some settings in the OS. These settings are set when server starts up. These are some settings that we need for k8s networking to work as expecting. 
```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf  
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1  
net.bridge.bridge-nf-call-ip6tables = 1  
EOF
```  
`sudo sysctl --system` -> apply those settings immediately. 

* Install and configure containerd
```
sudo apt-get update && sudo apt-get install -y containerd  
sudo mkdir -p /etc/containerd  
sudo containerd config default | sudo tee /etc/containerd/config.toml  
sudo systemctl restart containerd
```  

* If we don't use containerd, we can use docker as well
-> Set Up repository:
`sudo apt-get update`  
```
sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
gnupg \
lsb-release
```

-> add docker's official gpg key
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

-> set up stable repository
```
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

-> install 
```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
or
```
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu
sudo apt-mark hold docker-ce
```

`sudo apt-mark hold docker-ce`

* On all nodes, disable swap.
```
sudo swapoff -a   
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```     

* Install some dependencies
```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
```  

* Add GPG key to my system
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

* Add k8s repositories to my repositories list
```
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list  
deb https://apt.kubernetes.io/ kubernetes-xenial main  
EOF
```  

* To pull down the new package definitions
`sudo apt-get update`  

* Install k8s
```
sudo apt-get install -y kubelet=1.20.1-00 kubeadm=1.20.1-00 kubectl=1.20.1-00
```  

To prevent unattended automatic upgrades-
```
sudo apt-mark hold kubelet kubeadm kubectl
```  

#### Only on Master
```
sudo kubeadm init --pod-network-cidr 192.168.0.0/16  
/// for flannel 10.244.0.0/16
mkdir -p $HOME/.kube  
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```  

Verify version  
`kubectl version`  

Install calico Network  
```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```  

Check Calico related pods  
`kubectl get pods -n kube-system`  

Or we can user flannel network
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Get the join command 
`kubeadm token create --print-join-command`  

In Node to join  
`sudo kubeadm join ..`  


## CentOS
Elevate privileges
```
sudo su
```

Disable SELinux
```
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

Enable the br_netfilter
```
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```

Ensure that the Docker dependencies are satisfied:
```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

Add docker repo
```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
```
yum install -y docker-ce
```

Set the cgroup driver for Docker to systemd, reload systemd, then enable and start Docker
```
sed -i '/^ExecStart/ s/$/ --exec-opt native.cgroupdriver=systemd/' /usr/lib/systemd/system/docker.service
systemctl daemon-reload
systemctl enable docker --now
```

Add k8s repo
```
cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
  https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

Install k8s
```
yum install -y kubelet kubeadm kubectl
```

```
systemctl enable kubelet
```

#### Only in master
```
kubeadm init --pod-network-cidr=10.244.0.0/16
```

exit sudo then do the following
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Add flannel
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Test
```
kubectl get pods --all-namespaces
```

```
kubectl get nodes
```

