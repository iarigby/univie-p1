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
sudo swapoff -a  
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
kubeadm init --control-plane-endpoint=kube-master --pod-network-cidr=10.8.0.0/24 --upload-certs
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

TODO add instruction to replace net-conf.jsjon to subnet


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



### Flannel with wireguard
deleting everything [source](https://stackoverflow.com/questions/46276796/kubernetes-cannot-cleanup-flannel)

```sh
systemctl stop kubelet
systemctl stop containerd
rm -rf /var/lib/cni/
rm -rf /run/flannel
rm -rf /etc/cni/
ifconfig cni0 down
ifconfig flannel.1 down
ip link delete cni0
ip link delete flannel.1
systemctl start containerd
systemctl start kubelet

```


```
journalctl -xeu kubelet
```

```sh
kubectl patch node nuca -p '{"spec":{"podCIDR":"10.8.0.0/24"}}'
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  labels:
    k8s-app: flannel
    pod-security.kubernetes.io/enforce: privileged
  name: kube-flannel
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: flannel
  name: flannel
  namespace: kube-flannel
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: flannel
  name: flannel
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
- apiGroups:
  - networking.k8s.io
  resources:
  - clustercidrs
  verbs:
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: flannel
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-flannel
---
apiVersion: v1
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.8.0.0/24",
      "Backend": {
        "Type": "vxlan"
      }
    }
kind: ConfigMap
metadata:
  labels:
    app: flannel
    k8s-app: flannel
    tier: node
  name: kube-flannel-cfg
  namespace: kube-flannel
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: flannel
    k8s-app: flannel
    tier: node
  name: kube-flannel-ds
  namespace: kube-flannel
spec:
  selector:
    matchLabels:
      app: flannel
      k8s-app: flannel
  template:
    metadata:
      labels:
        app: flannel
        k8s-app: flannel
        tier: node
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
      containers:
      - args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=wg0
        command:
        - /opt/bin/flanneld
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: EVENT_QUEUE_DEPTH
          value: "5000"
        image: docker.io/flannel/flannel:v0.25.1
        name: kube-flannel
        resources:
          requests:
            cpu: 100m
            memory: 50Mi
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
            - NET_RAW
          privileged: false
        volumeMounts:
        - mountPath: /run/flannel
          name: run
        - mountPath: /etc/kube-flannel/
          name: flannel-cfg
        - mountPath: /run/xtables.lock
          name: xtables-lock
      hostNetwork: true
      initContainers:
      - args:
        - -f
        - /flannel
        - /opt/cni/bin/flannel
        command:
        - cp
        image: docker.io/flannel/flannel-cni-plugin:v1.4.0-flannel1
        name: install-cni-plugin
        volumeMounts:
        - mountPath: /opt/cni/bin
          name: cni-plugin
      - args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        command:
        - cp
        image: docker.io/flannel/flannel:v0.25.1
        name: install-cni
        volumeMounts:
        - mountPath: /etc/cni/net.d
          name: cni
        - mountPath: /etc/kube-flannel/
          name: flannel-cfg
      priorityClassName: system-node-critical
      serviceAccountName: flannel
      tolerations:
      - effect: NoSchedule
        operator: Exists
      volumes:
      - hostPath:
          path: /run/flannel
        name: run
      - hostPath:
          path: /opt/cni/bin
        name: cni-plugin
      - hostPath:
          path: /etc/cni/net.d
        name: cni
      - configMap:
          name: kube-flannel-cfg
        name: flannel-cfg
      - hostPath:
          path: /run/xtables.lock
          type: FileOrCreate
        name: xtables-lock
```


https://stackoverflow.com/questions/60297810/kubelet-config-yaml-is-missing-when-restart-work-node-docker-service


https://askubuntu.com/questions/224966/how-do-i-get-resolvconf-to-regenerate-resolv-conf-after-i-change-etc-network-in
```
resolvconf -u
```