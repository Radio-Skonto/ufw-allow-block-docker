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

