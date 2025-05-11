

```yaml
systemctl stop kubelet
systemctl stop containerd
iptables --flush
iptables -tnat --flush
systemctl start kubelet
systemctl start containerd
```


https://github.com/kubernetes/dashboard/issues/3709

>[@floreks](https://github.com/floreks) seems to be an issue with the fact my pod network was 192.168.0.0/16 and my nodes 192.168.50.10, 192.168.50.11, .. (inside that range). When I updated the CIDR of my pod network to 172.16.0.0/16 the deployment of the dashboard went well.


https://tufora.com/tutorials/kubernetes/change-kubernetes-pods-cidr

https://docs.oracle.com/en-us/iaas/Content/ContEng/Concepts/contengcidrblocks.htm

https://stackoverflow.com/questions/45903123/kubernetes-set-service-cidr-and-pod-cidr-the-same

