# Network 

I need to connect a machine from a residential building. 

##### 1. Fixed IP
The most straightforward solution is getting a fixed ip and port forwarding from isp, but exposing my home network is something I would avoid unless there are no other alternatives.

##### 2. Tunnel or Relay vps
The next easy option seems to be using a Cloudflare tunnel. I am not sure though if this will be sufficient, but I will try over the weekend.

###### Outcome
Cloudflare tunnel offers two configurations: 
1. [public hostname](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/routing-to-tunnel/) - which binds to a single port, therefore not applicable since kubernetes needs several
2. [private network](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/private-net/)
	1. With `cloudflared` - seems also not applicable
	> Cloudflare Tunnel using `cloudflared` only proxies traffic initiated from a user to a server

	2. by [installing](https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/deployment/manual-deployment/) `warp` on the other servers and setting up [routing](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/private-net/cloudflared/#3-route-private-network-ips-through-warp) - will try next
##### 3. VPN
Third option is to create a virtual private network and connect nodes to it (I guess all of them). I feel like this will not be a straightforward thing for me to do given my skills in linux networking. I would gladly learn this in the future but right now it would be best if I can get 2. working easily

- https://docs.hetzner.com/cloud/apps/list/wireguard/



https://community.hetzner.com/tutorials/install-and-configure-wireguard-vpn
https://community.hetzner.com/tutorials/install-and-configure-wireguard-vpn-debian


In the end I used this for clients
https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04

