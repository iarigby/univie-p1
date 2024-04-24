```sh
export DEBIAN_FRONTEND=noninteractive
apt-get update && apt-get upgrade -y
```

?  change host files
```sh
# uncomment 127.0.1.1
188.34.196.243 kube-worker
116.203.239.230 kube-master
```

```sh
apt-get install firewalld -y
systemctl enable --now firewalld
firewall-cmd --add-port=10250/tcp --permanent --zone=public
firewall-cmd --add-port=6443/tcp --permanent --zone=public
firewall-cmd --reload
```

```sh
apt-get install -y containerd
printf "\noverlay\nbr_netfilter\n" /etc/modules-load.d/containerd.conf
systemctl enable --now containerd
systemctl status containerd
```

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

```sh
modprobe overlay
modprobe br_netfilter
```

```sh
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

```

```sh
# Apply sysctl params without reboot
sysctl --system
```

check that modules are loaded

```sh
lsmod | grep br_netfilter
lsmod | grep overlay
```

all should be one
```sh
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```


```shell
# apt-transport-https may be a dummy package; if so, you can skip that package
apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

```sh
printf '\nKUBELET_EXTRA_ARGS="--cgroup-driver=cgroupfs"\n' >> /etc/default/kubelet
systemctl enable --now kubelet
```


```sh
kubeadm config images pull
kubeadm init --control-plane-endpoint=kube-master --pod-network-cidr=10.244.0.0/16 --upload-certs
# kubeadm init --control-plane-endpoint=kube-master
# kubeadm init --pod-network-cidr=10.244.0.0/16
export KUBECONFIG=/etc/kubernetes/admin.conf
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> $HOME/.bashrc
```


```sh
watch kubectl get pods --all-namespaces
```

```sh
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

or
```sh
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```


error:
- cni plugin not initialized


- try with docker now

```
sudo sh -c "containerd config default /etc/containerd/config.toml" 
sudo sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' /etc/containerd/config.toml 
sudo systemctl restart containerd.service
sudo systemctl restart kubelet.service
```



```
systemctl daemon-reload
kubectl taint nodes --all
```


on debian i had no such file resolv.conf 