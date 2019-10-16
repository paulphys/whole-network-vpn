# Whole-network OpenVPN with pfSense

## Introduction
> *Why ? What are the benefits of running OpenVPN on the router level?*

Well, this question is rather easy to answer. Instead of having to download and configure OpenVPN clients on each of your devices you can just configure it once on your pfSense router and never have to worry about browsing unprotected.
Other benefits of running OpenVPN on the router level include, higher level of access and customizability, your device no longer has to use its processing power to encrypt and decrypt traffic, increased security and usability.
One of the most powerful features of pfSense is it’s ability to direct your data requests through different end-points using NAT rules. pfSense is amazing as an OpenVPN client because it can selectively route any device on the network through the VPN service. We'll cover NAT rules for latency-sensitive applications further on in this guide.

This setup becomes extremely handy for use with applications which are not aware of OpenVPN protocol, eg. download managers, torrent clients, etc. Expecting privacy you should be positive that traffic won't go through your ISP's gateway in case of failure on side of VPN provider. And obviously OpenVPN client should automatically reconnect as soon as service goes live again.


> **Note**: We will be working with pfSense Version 2.4.4, but this tutorial should also apply to all other versions


## Configuration

### Download OpenVPN configuration files
Here are some links to the configuration files for the most popular VPN Providers:
 [NordVPN]: (https://nordvpn.com/ovpn/)
 [ExpressVPN]: (https://www.expressvpn.com/support/vpn-setup/pfsense-with-expressvpn-openvpn/#download)
 [IPVanish]: https://www.ipvanish.com/software/configs/
 [Surfshark]: https://account.surfshark.com/setup/manual
 [CyberGhost]: https://support.cyberghostvpn.com/hc/en-us/articles/213811885-Router-How-to-configure-OpenVPN-for-flashed-DD-WRT-routers
 [StrongVPN]: https://support.strongvpn.com/hc/en-us/articles/360011443953-DD-WRT-OpenVPN-Auto-Installer-Guide
 [PIA]: https://www.privateinternetaccess.com/helpdesk/kb/articles/where-can-i-find-your-ovpn-files

Hosting your own OpenVPN server?:
 [Check out this guide]: https://openvpn.net/community-resources/setting-up-your-own-certificate-authority-ca/

#### Configure certificates:

* Go to `System` > `Cert Manager`
* In the `CAs` tab, click the `+ Add` icon to add a new Certificate Authority
* Fill in a `Descriptive name` like “[VPN PROVIDER] CA”
* Copy and paste `Certificate data`. It can be found in one of two `.crt` files, provided by VPN service. In some cases `.ovpn` file may include Certificate Authority information between `<ca>...</ca>` tags. **Do not** include this tags. All certificates going into pfSense should have similar format:
```
-----BEGIN CERTIFICATE-----
MIIEYTCCA0mgAwIBAgIJAOP9Uyx2LzzOMA0GCSqGSIb3DQEBBQUAMH0xCzAJBgNV
BAThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherEP
MAThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBother0B
CQThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherYw
MzThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBothercT
DEThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherUg
Q0ThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherZI
hvThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBothertL
o/ThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherLM
liThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherLB
xgThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherEP
2QThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBother/o
1lThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherCB
4DThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherAU
c8ThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherUE
CBThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherVN
RTThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherBo
aWThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherUA
A4ThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherog
lpThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBother2h
z1ThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBother7W
NpThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBother8Y
HmThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherUo
brc4OSiSKdeskaqGQgWaObJCdsnB
-----END CERTIFICATE-----
```
* Click `Save`.
* Go to the `Certificates` tab and click the `+` icon to add your VPN certificate and private key.
* Fill in a `Descriptive name` like “[VPN PROVIDER] CERT”
* Copy and paste `Certificate data`. It can be found in one of two `.crt` files, provided by VPN service. In some cases `.ovpn` file may include Certificate information between `<cert>...</cert>` tags. **Do not** include this tags.
* Copy and paste `Private key data`. It can be found in `.key` file, provided by VPN service. In some cases `.ovpn` file may include private key information between `<key>...</key>` tags. **Do not** include this tags. All private keys going into pfSense should have similar format:
```
-----BEGIN PRIVATE KEY-----
MIIEYTCCA0mgAwIBAgIJAOP9Uyx2LzzOMA0GCSqGSIb3DQEBBQUAMH0xCzAJBgNV
BAThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherEP
MAThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBother0B
CQThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherYw
MzThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBothercT
DEThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherUg
Q0ThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherZI
hvThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBothertL
o/ThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherLM
liThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherLB
xgThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherEP
2QThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBother/o
1lThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherCB
4DThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherAU
c8ThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherUE
CBThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherVN
RTThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherBo
aWThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherUA
A4ThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherog
lpThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBother2h
z1ThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBother7W
NpThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBother8Y
HmThisIsOnlyAnExampleDoNotBotherThisIsOnlyAnExampleDoNotBotherUo
brc4OSiSKdeskaqGQgWaObJCdsnB
-----END PRIVATE KEY-----
```
* Click `Save`

#### Configure OpenVPN client:
* Go to `VPN` > `OpenVPN`
* Click the `Client` tab.
* Click the `+` icon to add a new client.
> **Note:** Most of the settings on this tab totally depend on VPN provider. For NordVPN, do the following:

 These are important for the how-to:
* `Interface` set to `WAN` interface.
* `Server host name resolution` needs to be checked in order for client to automatically reconnect.
* `Peer Certificate Authority` and `Client Certificate` set to previosly defined.
* `redirect-gateway def1` should persist in `Advanced` options to actually route traffic through VPN.
 These are the rest for my VPN provider:
* `Server Mode` = `Peer to Peer ( SSL/TLS )`
* `Protocol` = `UDP` => `Server port` = `1194`
* `Protocol` = `TCP` => `Server port` = `443`
* `Device mode` = `tap`
* `Description` = "[VPN Provider name]"
* `TLS Authentication`: unchecked `Enable authentication of TLS packets`

>**Note:** if your VPN provider uses TLS Authentication you should check it,
>uncheck `Automatically generate a shared TLS authentication key.` and paste your shared key. It usually can be found in `.ovpn` configuration file between `<tls-auth> ... </tls-auth>` tags. **Do not** include this tags.  Paste should look like this:
```
-----BEGIN OpenVPN Static key V1-----
4ThisIsOnlyAnExampleDoNotBother5
dThisIsOnlyAnExampleDoNotBother8
fThisIsOnlyAnExampleDoNotBother1
fThisIsOnlyAnExampleDoNotBother2
dThisIsOnlyAnExampleDoNotBother6
8ThisIsOnlyAnExampleDoNotBother2
5ThisIsOnlyAnExampleDoNotBother5
fThisIsOnlyAnExampleDoNotBotherd
8ThisIsOnlyAnExampleDoNotBother3
0ThisIsOnlyAnExampleDoNotBother5
5ThisIsOnlyAnExampleDoNotBother0
bThisIsOnlyAnExampleDoNotBother6
dThisIsOnlyAnExampleDoNotBotherc
3ThisIsOnlyAnExampleDoNotBother1
fThisIsOnlyAnExampleDoNotBother5
eThisIsOnlyAnExampleDoNotBother9
-----END OpenVPN Static key V1-----
```

* `Encryption algorithm` = `BF-CBC (128-bit)`
* `Auth Digest Algorithm` = `RSA-SHA1 (160-bit)`
* `Hardware Crypto` = `BSD cryptodev engine - RSA, DSA, DH` (Depends on CPU)
* `Compression` = `Enabled with Adaptive Compression`
* `Advanced` = `ns-cert-type server;redirect-gateway def1;persist-key;persist-tun;mute 20;explicit-exit-notify`
* `Verbosity level` = `4`
* Click `Save`

#### Validate connection status:
* Go to `Status` > `System Logs`
* Select the `OpenVPN` tab.
* Verify that you have successfully connected.
 Specifically look for `Initialization Sequence Completed` statement. It may be anywhere between other log entries but should be tagged with time when you clicked `Save` on client configuration tab.
 If you don’t see it, it means you are not connected.  Check your configuration again. Use the log to look for errors.  These are probably flags in your advance options or encryption settings. Double check that you pasted right certificates and keys.

#### Configure OpenVPN gateway interface:
* Go to `Interfaces` > `(assign)`
* In `Available network ports:` select  `ovpnc# [VPN Provider name]` according to the `Description` given on client configuration step.
* Click the `+` icon and add a new interface. It will be called `OPT#`
* Click the `OPT#` name of new interface to configure it.
* Change the name of `OPT#` into something more useful, eg. name of VPN server.
* `IPv4 Configuration Type` = `None`
* `IPv6 Configuration Type` = `None`
* You may want to decide on `Block private networks` for your setup. Mine is unchecked since this pfSense is a virtual machine in a private network.
* Click `Save`

#### Verify working gateways:
* Go to `Status` > `Dashboard`
* Look for `[VPN Provider name]` entry in `Interfaces` table
 (Alternatively `Status` > `Interfaces`)
* Verify that you have an IP Address for your VPN.
* If no, try going to `Status` > `Services` and restarting OpenVPN service by clicking the play button next to `OpenVPN client: [VPN Provider name]`
>**Note:** you may want to have OpenVPN table on dashboard to see client connection status. Click `+` icon right under `Status: Dashboard` header at the top of page, select `OpenVPN` and click `Save Settings` button.

* Go to `System` > `Routing`
* Verify that your gateways are available:
 there should be green play icon before `[VPN Provider name]_VPNV4`

> **Note:** In pfSense 2.1.x or below that entry should have IP address `Gateway` column. If no , try opening the entry, scrolling down and clicking `Save`.  That seemed to restart it.
> **Note:** In pfSense 2.2-Beta or above there probably would be `dynamic` in `Gateway` column of VPN entry.

#### Configure NAT
* Go to `Firewall` > `NAT`
* Select the `Outbound` tab.
* Note rules in automatically generated table.
* Select the `Manual Outbound NAT Rule Generation (AON - Advanced Outboud NAT)` radio button.
* Click `Save`
* Now you should see all the same rules ungrouped and editable. Verify presence of all seing earlier rules.
* By clicking `+` icon next to the rule entry, copy every rule changing only the `interface` to the one you created for VPN client `[VPN Server name]`

> **Note:** rules for VPN interface should follow the corresponding for WAN interface. Order is crucial here. That is the reason we are not able to use "convinient" `Hybrid Outbound NAT rule generation
(Automatic Outbound NAT + rules below)`. As it stated on the bottom of the page: "If hybrid outbound NAT is selected, mappings you specify on this page will be used, followed by the automatically generated ones."

> **Note:** Rule of thumb: final NAT mappings table should have 4 rules for each interface on the system except OpenVPN client's one (eg. 4x WAN + 4x LAN) (Theoretically, you may configure more then one OpenVPN client on single pfSense, but since `“redirect-gateway def1”` option redirects all the traffic, I don't believe in success of such setups).

#### Configure firewall:
From this moment you use Firewall rules to direct traffic from your IPs/networks/interfaces to either WAN gateway (for direct ISP connection) or VPN client gateway for VPN access.
I especially do not define any steps for further configuration because some pfSense version behave little bit different here and everyone's setup would be different, so you should play a bit with rules, learn how they affect your network and you will be rewarded eventually with pretty good skills and understanding of the whole picture.

## NAT rules for latency-sensitive applications
* Steam

UDP 27000 to 27015 inclusive (Game client traffic)
UDP 27015 to 27030 inclusive (Typically Matchmaking and HLTV)
TCP 27014 to 27050 inclusive (Steam downloads)
UDP 4380

Dedicated or Listen Servers
TCP 27015 (SRCDS Rcon port)

Steamworks P2P Networking and Steam Voice Chat
UDP 3478 (Outbound)
UDP 4379 (Outbound)
UDP 4380 (Outbound)

Additional Ports for Call of Duty: Modern Warfare 2 Multiplayer
UDP 1500 (outbound)
UDP 3005 (outbound)
UDP 3101 (outbound)
UDP 28960

#### Sources:
-[Level1techs](https://forum.level1techs.com/t/whole-network-vpn-with-pfsense-router-level-one-techs/114309)

-[pixelsandwidgets](http://www.pixelsandwidgets.com/2014/10/setup-pfsense-openvpn-client-specific-devices/)

-[NordVPN](https://nordvpn.com/tutorials/pfsense/pfsense-openvpn/)

