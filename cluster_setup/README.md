https://www.youtube.com/watch?v=vpEDUmt_WKA


## Commands for master node

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get update -y

sudo apt-get install -y apt-transport-https ca-certificates curl

sudo apt-get install -y docker-ce

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update -y

sudo apt-get install -y kubelet kubeadm kubectl

echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

sudo kubeadm init --pod-network-cidr=69.69.0.0/16 --ignore-preflight-errors=all

mkdir -p $HOME/.kube  
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
sudo chown $(id -u):$(id -g) $HOME/.kube/config  

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

sudo usermod -a -G docker ubuntu

/etc/resolv.conf

sudo nano /etc/netplan/99-custom-dns.yaml
```
network:
    version: 2
    ethernets:
        eth0:         
            nameservers:
                    addresses: [1.1.1.1, 1.0.0.1]
            dhcp4-overrides:
                    use-dns: false
```


## Commands for worker nodes

sudo apt-get update -y

sudo apt-get install -y docker.io

sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update -y

sudo apt-get install -y kubelet kubeadm kubectl

echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

sudo usermod -a -G docker ubuntu

/etc/resolv.conf

sudo nano /etc/netplan/99-custom-dns.yaml
```
network:
    version: 2
    ethernets:
        eth0:         
            nameservers:
                    addresses: [1.1.1.1, 1.0.0.1]
            dhcp4-overrides:
                    use-dns: false
```

sudo kubeadm join 69.69.69.194:6443 --token 26t0sl.taobbhyry60ulthi --discovery-token-ca-cert-hash sha256:1ed312efc9f6b62c9c94c56d086a259f989bddd54f337937cfd60f5a8850e35a
