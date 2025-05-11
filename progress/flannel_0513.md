`kube-flannel.yml`

```yml
 net-conf.json: |
    {
      "Network": "10.8.0.0/24", # this is my configuration for wireguard
      "Backend": {
        "Type": "vxlan" # maybe this needs to be changed?
      }
    }
```


![](2024-05-13_22.27.49.png)