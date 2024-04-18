
```
nc -l -p 10250

apt-get update && apt-get upgrade
apt-get install containerd
systemctl enable --now containerd
```

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sysctl --system
```

check that modules are loaded

```
lsmod | grep br_netfilter
lsmod | grep overlay
```

all should be one
```
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
systemctl enable --now kubelet
kubeadm config images pull
```

```sh
kubeadm init
```



```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```
kubectl get pods --all-namespaces
```

```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```


### after joining
```
kubectl -n kube-system rollout restart deployment coredns
kubectl -n kube-flannel rollout restart deployment kube-flannel
```

```
kubectl delete pod -n kube-flannel -l app=flannel
```

```
 kubectl get no $hostname -o yaml >> file.yaml | sed -i "s~$old_ip~$new_ip~" file.yaml| kubectl delete no $hostname && kubectl create -f file.yaml
```