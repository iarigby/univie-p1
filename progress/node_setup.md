# Node Setup Scripts

```sh
export DEBIAN_FRONTEND=noninteractive
apt-get update && apt-get upgrade -y
# uncomment 127.0.1.1 ?
printf "10.8.0.2\tnuca\n10.8.0.3\tkube-master\n10.8.0.4\tvps-worker-1\n" >> /etc/hosts 
# start wireguard
apt install -y wireguard resolvconf
```

 **from host**

```bash
scp ~/Downloads/kube-master.conf root@kube-master:/etc/wireguard/wg0.conf
```

```bash
scp ~/Downloads/vps-worker-1.conf root@vps-worker-1:/etc/wireguard/wg0.conf
```

```bash
scp ~/Downloads/nuca.conf root@nuca:/etc/wireguard/wg0.conf
```

**back to node**

```sh
# !!!! ONLY ON HOME NODE
sed -i '/ swap / s/^/#/' /etc/fstab
```


```sh
systemctl enable --now wg-quick@wg0
# finished wireguard
apt-get install firewalld -y
systemctl enable --now firewalld
firewall-cmd --add-port=10250/tcp --permanent --zone=public
firewall-cmd --add-port=6443/tcp --permanent --zone=public
firewall-cmd --reload
# ---- #
apt-get install -y containerd
printf "\noverlay\nbr_netfilter\n" >> /etc/modules-load.d/containerd.conf
systemctl enable --now containerd
systemctl status containerd
```

```bash
printf "overlay\nbr_netfilter\n" >> /etc/modules-load.d/k8s.conf
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
# check that modules are loaded
lsmod | grep br_netfilter
lsmod | grep overlay
# all should be one
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

# install kube
# apt-transport-https may be a dummy package; if so, you can skip that package
apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
printf '\nKUBELET_EXTRA_ARGS="--cgroup-driver=cgroupfs"\n' >> /etc/default/kubelet
systemctl enable --now kubelet
```


```sh
kubeadm config images pull
kubeadm init --control-plane-endpoint=kube-master --pod-network-cidr=10.8.0.1/24 --upload-certs
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


TODO 
flannel with wireguard

https://github.com/xetys/hetzner-kube/issues/139
--iface wg0

TODO add instruction to replace net-conf.json to subnet


---

```
mkdir -p /etc/containerd
sudo sh -c "containerd config default /etc/containerd/config.toml" 

sudo sed -i 's/ SystemdCgroup = false/ SystemdCgroup = true/' 
sudo systemctl restart containerd.service
sudo systemctl restart kubelet.service
```


```
systemctl daemon-reload
kubectl taint nodes --all
```


