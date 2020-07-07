# kubernetes installation
Setup Kubernetes cluster, step by step

Make sure you have docker on your machine.

*Note: To set cgroups on docker, it's optional.*

```
# Set up the Docker daemon
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

# Restart Docker
systemctl daemon-reload
systemctl restart docker
```

**step 1** \
Install k8s (the version you can change if you want)
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update

# with a specific version
# sudo apt-get install -y kubelet=1.12.7-00 kubeadm=1.12.7-00 kubectl=1.12.7-00
# latest version
sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl

#init master
sudo kubeadm init --pod-network-cidr=10.138.216.179/16 --apiserver-advertise-address=10.138.0.0

#--apiserver-advertise-address = determines which IP address Kubernetes should advertise its API server on.
#--pod-network-cidr = specify the range of IP addresses for the pod network. We're using the 'flannel' virtual network.


mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl version

```
**step 2** \
Add the workers
```
sudo kubeadm join $some_ip:6443 --token $some_token --discovery-token-ca-cert-hash $some_hash

#here you can see all the nodes
kubectl get nodes

```
**step 3** \
Install the k8s network, in the official k8s page we can see all the options, here I show two k8s networks, calico and flannel, just use one of them. After this step, we can see the nodes all ready.
```
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
*If you have problems with this commnand above, you try this one:
curl -o kube-flannel.yml https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
sed -i "s/quay.io\/coreos\/flannel/quay-mirror.qiniu.com\/coreos\/flannel/g" kube-flannel.yml
kubectl apply -f kube-flannel.yml
rm -f kube-flannel.yml

# calico network
curl https://docs.projectcalico.org/manifests/canal.yaml -O
kubectl apply -f canal.yaml

#here you can see all the nodes ready
kubectl get nodes

```


Enjoy it :D
