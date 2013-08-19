Assuming you have a router with OpenWRT running and the following files, 
place them in the following places:

HIPNET_client.ipk - your compiled package (located in the /bin directory), 
install with 'opkg install'. Attached version is most up to date binary.

The remaining scripts in this directory should be placed in the corresponding directories on the router.


dhclient-script - /usr/sbin/dhclient-script

radvd - /etc/config/radvd

network - /etc/config/network

dhclient6 - /etc/init.d/dhclient6

dhcpd6 - /etc/init.d/dhcpd6



Example

Here is an example command to run from the router to install everything assuming file are in you user directory.

scp localuser@192.168.1.134:/Users/localuser/router_dns_client.ipk .; opkg install router_dns_client.ipk; scp localuser@192.168.1.134:/Users/localuser/dhclient-script /usr/sbin/dhclient-script; scp localuser@192.168.1.134:/Users/localuser/radvd /etc/config/radvd

Enable client

In order for the HIPNET client to run on boot run the following command once on each router.

'/etc/init.d/dhclient6 enable'

