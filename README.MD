### Append the following at the end of /etc/ufw/after.rules (replace eth0 with your external facing interface):

### Put Docker behind UFW
```
*filter
:DOCKER-USER - [0:0]
:ufw-user-input - [0:0]

-A DOCKER-USER -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A DOCKER-USER -m conntrack --ctstate INVALID -j DROP
-A DOCKER-USER -i eth0 -j ufw-user-input
-A DOCKER-USER -i eth0 -j DROP
COMMIT
```

And undo any and all of:
```
Remove "iptables": "false" from /etc/docker/daemon.json
Revert to DEFAULT_FORWARD_POLICY="DROP" in /etc/default/ufw
Remove any docker related changes to /etc/ufw/before.rules
```
Be sure to test that everything comes up fine after a reboot.

I still believe Docker's out of the box behavior is dangerous and many more people will continue to unintentionally expose internal services to the outside world due to Docker punching holes in otherwise safe iptables configs.

(edit: I didn't see the need to set MANAGE_BUILTINS=no and IPV6=no, or to fiddle with /etc/ufw/before.init)

////===============SECOND SOLUTION========================
```
# BEGIN UFW AND DOCKER
*filter
:ufw-user-forward - [0:0]
:DOCKER-USER - [0:0]
-A DOCKER-USER -j RETURN -s 10.0.0.0/8
-A DOCKER-USER -j RETURN -s 172.16.0.0/12
-A DOCKER-USER -j RETURN -s 192.168.0.0/16

-A DOCKER-USER -j ufw-user-forward

-A DOCKER-USER -j DROP -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -d 192.168.0.0/16
-A DOCKER-USER -j DROP -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -d 10.0.0.0/8
-A DOCKER-USER -j DROP -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -d 172.16.0.0/12
-A DOCKER-USER -j DROP -p udp -m udp --dport 0:32767 -d 192.168.0.0/16
-A DOCKER-USER -j DROP -p udp -m udp --dport 0:32767 -d 10.0.0.0/8
-A DOCKER-USER -j DROP -p udp -m udp --dport 0:32767 -d 172.16.0.0/12

-A DOCKER-USER -j RETURN
COMMIT
# END UFW AND DOCKER
```
