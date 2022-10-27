---
title: OpenVPN (UDP) DPI Bypass
date: 2022-10-26 14:10:00 +0800
categories: [Networking]
tags: [networking, dpi]
render_with_liquid: false
image:
  path: /assets/dpi.jpg
  width: 800
  height: 500
  alt: Responsive rendering of Chirpy theme on multiple devices.
---
## How does DPI (Deep Packet Inspection) identify OpenVPN in the first place?

So to answer that question we have to fully understand what takes place when your packets are scanned by DPI, and how OpenVPN itself is flagged. DPI, not to be confused with SPI (Stateful Packet Inspection), scans both `header` and the `data` of a packet.  

This allows for `application data` to be filtered or blocked by DPI which is notoriously used in countries like China, Egypt, and other censorship advocating nation states. 

## How specifically does DPI identify a OpenVPN connection among other traffic

Due to the cryptographic nature of OpenVPN, the TLS (1.2) handshake requires the Client Key to be transmitted in plain text to the OpenVPN server. Unfortunately because of that reason DPI is able to identify those said keys and block the `TLS Handshake` itself.

![Screen Shot 2022-10-26 at 11.27.24 PM.png](/assets/Screen Shot 2022-10-26 at 11.27.24 PM.png)


## How do we mask our traffic?

To put it straight, you'd have to change the `udp` packet's application data (or in our case obsfucate the cryptographic keys) somehow, or you could use a encrypted connection to a machine outside DPI to handle any blocked networking. This would require some money or atleast setting up an VPS and or something similar to get working. 

![Screen Shot 2022-10-26 at 11.46.23 PM.png](/assets/Screen Shot 2022-10-26 at 11.46.23 PM.png)

## Bypassing DPI using Shadowsocks and UDPrelay along with OpenVPN

In essence what we are doing is masking our udp traffic in its entirety by sending it too Shadowsocks servers and enabling the socks proxy option inside our OpenVPN config file. To do so make sure to add thise line to your config file (you can add it to the very top for convience)
```
socks-proxy 127.0.0.1 1080
```

Now install Shadowsocks 

```bash
 sudo apt update
 sudo apt install shadowsocks-libev
 sudo apt-get install --no-install-recommends gettext build-essential autoconf libtool libpcre3-dev asciidoc xmlto libev-dev libc-ares-dev automake libmbedtls-dev libsodium-dev pkg-config

```

What you want to do now is find free Shadowsocks server infastructure to use. It'll take some looking, but if you get stuck try looking at (https://sshocean.com). Once you got the server information you can do a couple things, either launch the cli or use the gui app which you can install with this command.

```bash
wget https://github.com/shadowsocks/shadowsocks-qt5/releases/download/v3.0.1/Shadowsocks-Qt5-3.0.1-x86_64.AppImage
```

If you do decide to use the cli, I made a script that will allow the config file to be created seamlessly and it also includes a built in `ss://` link parser to read the info and make your own config file (doesn't support plugin reading; for that just decode the `base64` and input the flags manually).

Once you got your config setup its now time to connect to the Shadowsocks server and connect to your chosen OpenVPN server alongside. 

```bash
shadowsocks-libev -c ## PathToShadowsocksCONFIG -u

## If plugins are present use, and specify "--plugin-opts" in config file
shadowsocks-libev -c ## PathToShadowsocksCONFIG -u --plugin PluginName

```

Now when you connect to your openVPN server DPI should be unable to identify the TLS handshake. Here's a OpenVPN connection done in egypt just yesterday done in testing.

![egyptpwn.png](/assets/egyptpwn.png)



