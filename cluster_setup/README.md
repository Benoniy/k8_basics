### Commands for worker nodes

https://www.youtube.com/watch?v=vpEDUmt_WKA

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

sudo kubeadm join 69.69.69.194:6443 --token 26t0sl.taobbhyry60ulthi --discovery-token-ca-cert-hash sha256:1ed312efc9f6b62c9c94c56d086a259f989bddd54f337937cfd60f5a8850e35a
