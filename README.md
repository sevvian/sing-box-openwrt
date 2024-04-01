In this guide we will install the package sing box on ImmortalWrt 23.05.1

Recommended router at least 128 MB RAM ( 256 preferably ) and a memory of more than 16 MB, the way to install sing-box in RAM ( is suitable for devices with a small amount of ROM < 16 Mb ) will also be described.

Sing-Box —is a free and open-source proxy platform that allows users to bypass internet censorship and access blocked websites. It is an alternative to V2ray and XRAY. It can be used with various V2Ray clients on platforms such as Windows, macOS, Linux, Android, and iOS.

In addition to supporting Shadowsocks, Trojan, Vless, and Socks protocols, it also supports new protocols such as ShadowTLSv3,Hysteria2,Tuic and NaiveProxy.

Pay attention to this point, to install Sing-box in the router, use the builds taken for the passwall.

openwrt-passwall-build

The manual will include:

Setting sing-box
For the ImmortalWrt version 23.05.1 , You must enter the following commands:

Update the list of packages:

 opkg update
Next, we install the necessary for work sing box kernel modules and compatibility package with iptables:

 opkg install kmod-inet-diag kmod-netlink-diag kmod-tun iptables-nft
We are waiting for the installation to complete, the packages occupied about 1MB of memory.

Next, go to the installation sing box

 opkg install sing-box
The package takes about 10MB, so it cannot be installed on devices with 16 MB of ROM without additional manipulations. If the package is successfully installed, we proceed to configure the connection.

Configuring sing-box for Hysteria2+Vless+gRPC
Next, go to the configuration file, by default this /etc/sing-box/config.json, but available when installed /etc/sing-box/config.json.example need to create yourself.

/etc/sing-box/config.json
Example config.json:

config.json.example

Configuration writes in log /tmp/sing-box.log warnings and errors, raises stocks proxy on port 1080, raises the tunnel tun0 using kmod-tun, parameter "auto_route": true defines shadows as the default route, replace the value with false if this is not required.

Next, check the configuration performance:

 sing-box check -c /etc/sing-box/config.json
If everything is correct, the team will not make mistakes.

Next, we check the work of proxy:

 sing-box run -c /etc/sing-box/config.json
If everything worked, we can add sing box to start the start, for this we enter the commands:

/etc/init.d/sing-box enable
/etc/init.d/sing-box start
create a start-up service for sing box.
Create a file /etc/init.d/sing-box as follows:

#!/bin/sh /etc/rc.common
#
# Copyright (C) 2022 by nekohasekai <contact-sagernet@sekai.icu>
# 
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
# 
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
# 
# You should have received a copy of the GNU General Public License
# along with this program. If not, see <http://www.gnu.org/licenses/>.
#

START=99
USE_PROCD=1

#####  ONLY CHANGE THIS BLOCK  ######
PROG=/usr/bin/sing-box 
RES_DIR=/etc/sing-box/ # resource dir / working dir / the dir where you store ip/domain lists
CONF=./config.json   # where is the config file, it can be a relative path to $RES_DIR
#####  ONLY CHANGE THIS BLOCK  ######

start_service() {
  sleep 10 
  procd_open_instance
  procd_set_param command $PROG run -D $RES_DIR -c $CONF

  procd_set_param user root
  procd_set_param limits core="unlimited"
  procd_set_param limits nofile="1000000 1000000"
  procd_set_param stdout 1
  procd_set_param stderr 1
  procd_set_param respawn "${respawn_threshold:-3600}" "${respawn_timeout:-5}" "${respawn_retry:-5}"
  procd_close_instance
  iptables -I FORWARD -o tun+ -j ACCEPT
  echo "sing-box is started!"
}

stop_service() {
  service_stop $PROG
  iptables -D FORWARD -o tun+ -j ACCEPT
  echo "sing-box is stopped!"
}

reload_service() {
  stop
  sleep 5s
  echo "sing-box is restarted!"
  start
}
We make the file executable:

chmod +x /etc/init.d/sing-box
Then add to the startup:

/etc/init.d/sing-box enable
/etc/init.d/sing-box start
Now, in order to activate the kill switch function on the router, so that the exchanged traffic only passes through the Singbox, we must do the following steps:

1.Click on Network → Interfaces, then click on the button of the Add new interface... In the opened page, we choose a name for the interface (for example, tun0) In the protocol section, we select Unmanaged and finally, in the device section, we select the desired connection, which is called nekoray-tun based on our configuration. And we create the Intended interface.

2.Click on Network → Firewall, in General Settings in the section Zones click on the button of the Add Zone In the name field, choose a name for the new zone (for exmaple, Sing_box) and set the values ​​of Input, Output and Forward equal to the accept value. and activate the Masquerading option and set the value of Covered networks equal to the interface we created. And the last change that should be applied in the zones section is related to the LAN zone Click on the edit button in the LAN zone and set the value of Allow forward to destination zones equal to the zone that we created for the Sing_box.

This setting is over :)
