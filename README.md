Purpose
======

This is a collection of commands that allows you to connect a device to a remote LAN over Zerotier.

Setup
======

- Get a [Zerotier](https://my.Zerotier.com/) account. It's free :)
- Set up a Zerotier [network](https://Zerotier.atlassian.net/wiki/spaces/SD/pages/8454145/Getting+Started+with+ZeroTier). It is a good idea to choose a "IPv4 Auto-Assign" Range that is not in use on your local LAN or the remote LAN you wish to connect to.
- [Download](https://www.Zerotier.com/download/) the Zerotier client to both your Linux drop box and the host that you want to access the remote LAN.
- Join each device to your Zerotier network (fill in ########## with your network's id):
```
sudo zerotier-cli join ##########
```
- In your Zerotier web console, authorize each device by checking the "Auth?" box next to each device. You may also want to create device descriptions to make things easier to manage.
- Ensure that you can ssh into your Linux host through its Zerotier "Managed IP". You should now be able to manage the device remotely and make ad-hoc configuration changes if need be.
- Make sure the Linux box has iptables installed.
- Make sure IP Forwarding is allowed on the Linux box:
```
sysctl -w net.ipv4.ip_forward=1
```
- To check IP Forwarding status:
```
sysctl net.ipv4.ip_forward
```

Accessing the Remote LAN
======

- This example makes the following assumptions:
    - Remote LAN is using the CIDR range 10.100.0.0/16
    - Remote DNS server is 10.100.53.53
    - Local LAN is using some other range (192.168.1.0/24)
    - Zerotier issues Managed IPs in the CIDR range 172.23.0.0/16
    - The Linux box eth0 interface is connected to the remote LAN and has the address 10.100.0.5
    - The Linux box has a Zerotier interface named ztabcd1234 and has the address 172.23.0.2
- In the "Advanced" section in Zerotier under "Managed Routes", add a route to the remote LAN. You can add multiple if need be:
    - "Destination" 10.100.0.0/16
    - "(Via)" 172.23.0.2 (your Linux box Zerotier address).
- Add this same route to your local host:
```
sudo route -n add -net 10.100.0.0/16 172.23.0.2
```
- Make sure you have correctly added the route:
```
netstat -nr
```
- You should now be able to ping the Linux box on its remote IP 10.100.0.5
- Now ssh into the Linux host and add the following iptables rules (make sure to sub the primary interface name and Zerotier interface name based on your setup):
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth0 -o ztabcd1234 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i ztabcd1234 -o eth0 -j ACCEPT
```
- You should now be able to ping arbitrary hosts on the remote LAN 10.100.0.0/16. Ping the remote DNS server, in this example 10.100.53.53, to make sure your iptables are working correctly. If you do not know your DNS settings on the Linux box, then run the following command:
```
cat /etc/resolv.conf
```
- You can now manually add the remote DNS server (and search domain) as your primary DNS on your localhost. This will allow you to have full access to the remote LAN as if you were directly plugged in.

Teardown
======

- You may want to restore things once you're done accessing the remote LAN
- Delete your manual DNS entries on your local host either through the GUI or by editing /etc/resolv.conf
- Remove your local routes that are pointing to your Linux box Zerotier IP:
```
sudo route -n delete 10.100.0.0/16
```
- Remove routes in the Zerotier web GUI
- Flush iptables on the Linux box:
```
sudo iptables -F
sudo iptables -t nat -D POSTROUTING 1
```
- Leave the Zerotier network:
```
sudo zerotier-cli leave ##########
```

Credits
======
[hKemmler](https://www.reddit.com/user/hKemmler/) on Reddit:
    https://www.reddit.com/r/zerotier/comments/9714a2/easy_way_of_bridging_lan_for_remote_access/
