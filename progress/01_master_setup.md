## Install dependencies
https://docs.fedoraproject.org/en-US/quick-docs/using-kubernetes/


```sh
sudo dnf update
sudo firewall-cmd --add-port=10250/tcp --permanent --zone=public
sudo firewall-cmd --add-port=6443/tcp --permanent --zone=public
sudo firewall-cmd --reload
```

```sh
sudo dnf install kubernetes kubernetes-client kubernetes-kubeadm -y
sudo systemctl enable --now kubelet

sudo dnf install cri-o -y
sudo systemctl enable --now crio

sudo kubeadm config images pull

sudo dnf install netcat -y
sudo dnf install ethtool -y
sudo dnf install iproute-tc -y
```

### configure dns
https://community.hetzner.com/tutorials/install-kubernetes-cluster
```sh
sudo cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

```sh
sudo modprobe overlay
sudo modprobe br_netfilter
```

```sh
sudo cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
# Allow IP forwarding for kubernetes
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward                = 1
net.ipv6.conf.default.forwarding   = 1
EOF
```


```bash
sudo sysctl --system
```


### checking if ports are open
```sh
nc -l 10250
nc kube-master 10250 -v
```


## initialise cluster
```shell
sudo kubeadm init --cri-socket unix:///var/run/crio/crio.sock --pod-network-cidr=10.244.0.0/16
```

`--pod-network` flag added later because of trouble with Flannel

#### output logs (first time, without cidr flags)
```
sudo kubeadm init --cri-socket unix:///var/run/crio/crio.sock
I0404 17:01:31.221271 3931812 version.go:256] remote version is much newer: v1.29.3; falling back to: stable-1.24
[init] Using Kubernetes version: v1.24.17
[preflight] Running pre-flight checks
	[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
	[WARNING Swap]: swap is enabled; production deployments should disable swap unless testing the NodeSwap feature gate of the kubelet
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [fedora-4gb-nbg1-1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 5.75.180.171]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [fedora-4gb-nbg1-1 localhost] and IPs [5.75.180.171 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [fedora-4gb-nbg1-1 localhost] and IPs [5.75.180.171 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 11.004454 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node fedora-4gb-nbg1-1 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node fedora-4gb-nbg1-1 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: g79d4h.uivgmi07v37osimg
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!
```


#### output logs (with flags)
```
I0406 12:02:42.130876   71838 version.go:256] remote version is much newer: v1.29.3; falling back to: stable-1.24
[init] Using Kubernetes version: v1.24.17
[preflight] Running pre-flight checks
	[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
	[WARNING Swap]: swap is enabled; production deployments should disable swap unless testing the NodeSwap feature gate of the kubelet
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [fedora-4gb-nbg1-1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.18.0.1 5.75.180.171]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [fedora-4gb-nbg1-1 localhost] and IPs [5.75.180.171 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [fedora-4gb-nbg1-1 localhost] and IPs [5.75.180.171 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 10.007769 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node fedora-4gb-nbg1-1 as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node fedora-4gb-nbg1-1 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: jguusc.gdqf8qfn3s6vdpfd
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy
```

#### output logs (with updated kubernetes version)


#### output next steps
```

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

```
### After initialising

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


### Creating CNI
https://kubevious.io/blog/post/comparing-kubernetes-container-network-interface-cni-providers

https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727

```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
#### Successful output
```sh
kubectl get pods --all-namespaces  # displayed error for flannel
```

```
NAMESPACE      NAME                                        READY   STATUS              RESTARTS   AGE
kube-flannel   kube-flannel-ds-25p74                       1/1     Running             0          13s
kube-system    coredns-57575c5f89-cwmpj                    0/1     ContainerCreating   0          50s
kube-system    coredns-57575c5f89-z2hhj                    0/1     ContainerCreating   0          50s
kube-system    etcd-fedora-4gb-nbg1-1                      1/1     Running             4          65s
kube-system    kube-apiserver-fedora-4gb-nbg1-1            1/1     Running             0          63s
kube-system    kube-controller-manager-fedora-4gb-nbg1-1   1/1     Running             0          61s
kube-system    kube-proxy-lqx4h                            1/1     Running             0          50s
kube-system    kube-scheduler-fedora-4gb-nbg1-1            1/1     Running             4          63s
```
#### troubleshooting
```sh
kubectl get pods --all-namespaces  # displayed error for flannel first time
kubectl -n kube-flannel logs kube-flannel-ds-bm7j2
```

output
```
Defaulted container "kube-flannel" out of: kube-flannel, install-cni-plugin (init), install-cni (init)
I0406 11:30:26.081712       1 main.go:209] CLI flags config: {etcdEndpoints:http://127.0.0.1:4001,http://127.0.0.1:2379 etcdPrefix:/coreos.com/network etcdKeyfile: etcdCertfile: etcdCAFile: etcdUsername: etcdPassword: version:false kubeSubnetMgr:true kubeApiUrl: kubeAnnotationPrefix:flannel.alpha.coreos.com kubeConfigFile: iface:[] ifaceRegex:[] ipMasq:true ifaceCanReach: subnetFile:/run/flannel/subnet.env publicIP: publicIPv6: subnetLeaseRenewMargin:60 healthzIP:0.0.0.0 healthzPort:0 iptablesResyncSeconds:5 iptablesForwardRules:true netConfPath:/etc/kube-flannel/net-conf.json setNodeNetworkUnavailable:true}
W0406 11:30:26.081931       1 client_config.go:618] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I0406 11:30:26.124485       1 kube.go:139] Waiting 10m0s for node controller to sync
I0406 11:30:26.124595       1 kube.go:461] Starting kube subnet manager
I0406 11:30:27.124952       1 kube.go:146] Node controller sync successful
I0406 11:30:27.125006       1 main.go:229] Created subnet manager: Kubernetes Subnet Manager - fedora-4gb-nbg1-1
I0406 11:30:27.125018       1 main.go:232] Installing signal handlers
I0406 11:30:27.125163       1 main.go:452] Found network config - Backend type: vxlan
I0406 11:30:27.125239       1 match.go:210] Determining IP address of default interface
I0406 11:30:27.126514       1 match.go:263] Using interface with name eth0 and address 5.75.180.171
I0406 11:30:27.126586       1 match.go:285] Defaulting external address to interface address (5.75.180.171)
I0406 11:30:27.126922       1 vxlan.go:141] VXLAN config: VNI=1 Port=0 GBP=false Learning=false DirectRouting=false
I0406 11:30:27.148816       1 kube.go:627] List of node(fedora-4gb-nbg1-1) annotations: map[string]string{"kubeadm.alpha.kubernetes.io/cri-socket":"unix:///var/run/crio/crio.sock", "node.alpha.kubernetes.io/ttl":"0", "volumes.kubernetes.io/controller-managed-attach-detach":"true"}
E0406 11:30:27.150188       1 main.go:332] Error registering network: failed to acquire lease: node "fedora-4gb-nbg1-1" pod cidr not assigned
I0406 11:30:27.150291       1 main.go:432] Stopping shutdownHandler...
```


##### attempts to fix (failed, had to recreate cluster with flags)

```sh
# wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
kubeadm config print init-defaults | grep serviceSubnet
# sed -i 's:10.244.0.0/16:10.96.0.0/12:g' kube-flannel.yml
# kubectl apply -f kube-flannel.yml

sudo cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep -i cluster-cidr

kubectl patch node fedora-4gb-nbg1-1 -p '{"spec":{"podCIDR":"10.17.0.0/16"}}'
```

```sh
kubeadm config print init-defaults
```

```yaml
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  ```

### Coredns not starting
```sh
sudo systemctl restart crio
sudo systemctl restart kubelet
kubectl rollout restart -n kube-system deployment/coredns
```

```
sudo ip link delete cni0 type bridge
```

### Resetting everything

```shell
sudo kubeadm reset
rm -r ~/.kube
# follow setup, including running lines as a default user
```




```sh
useradd -m ia
mkdir /home/ia/.ssh
cp /root/.ssh/authorized_keys /home/ia/.ssh
echo "ia ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
chown -R ia:ia /home/ia/.ssh
passwd ia
su ia
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown ia:ia $HOME/.kube/config
```

