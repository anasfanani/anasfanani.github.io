---
language: en
title: Various Open Source Proxy Installation
tags:
  - Proxy
categories:
  - Linux
date: 2023-11-30 23:58:43+07:00
description: "Test & run as much available open source proxy for testing ( does
  not permanent modify your system wide vps ). For use in the
  clash/sagernet/sing-box/nekobox/v2rayn. Available proxy : TUIC, ShadowTLS,
  ShadowSocks, Brook, RESTLS."
---
## Greeting

Test every proxy program on https://sing-box.sagernet.org/configuration/inbound/

Welcome, this is my environment :

```shell
Linux ip-xxx-xxx-xxx-xxx 6.2.0-1013-xxx #13~22.04.1-Ubuntu SMP Fri Sep  8 17:29:56 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
ubuntu@ip-xxx-xxx-xxx-xxx:~$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=22.04
DISTRIB_CODENAME=jammy
DISTRIB_DESCRIPTION="Ubuntu 22.04.3 LTS"
ubuntu@ip-xxx-xxx-xxx-xxx:~$
```

Each proxy run with `screen` program, so when client disconnected, can be returned in current workspace.

Example :

Open tuic workspace :

```shell
screen -R tuic
```

Minimize workspace :

send keyboard event `CTRL+A` then `CTRL+D`

Return workspace :

```shell
screen -R tuic
```

Usefull when need for debuging purpose.

## TUIC

Source : https://github.com/chika0801/tuic-install/tree/main

### Standalone installation

```shell
cd /root/
mkdir tuic
cd tuic
wget https://github.com/EAimTY/tuic/releases/latest/download/tuic-server-1.0.0-x86_64-unknown-linux-gnu
chmod +x tuic-server-1.0.0-x86_64-unknown-linux-gnu
```

Download config :

```shell
curl -Lo config_tuic.json https://raw.githubusercontent.com/chika0801/tuic-install/main/config_server.json
```

Or create new file `config_tuic.json` :

```shell
{
    "server": "[::]:443",
    "users": {
        "ee48f7be-6ae9-5654-9b61-8466aa8e16bc": "chika"
    },
    "certificate": "/root/tuic/cert.pem",
    "private_key": "/root/tuic/key.pem",
    "congestion_control": "bbr",
    "alpn": ["h3"],
    "log_level": "info"
}
```shell

Generate your certificate then save as `/root/tuic/cert.pem` and `/root/tuic/cert.pem`.
Or generate self-signed certificate with :

```shell
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 3650 -nodes -subj "/C=XX/ST=StateName/L=CityName/O=CompanyName/OU=CompanySectionName/CN=CommonNameOrHostname"
```

Run your TUIC server :

```shell
./tuic-server-1.0.0-x86_64-unknown-linux-gnu -c config_tuic.json
```

### Configure Client

#### Android

* Use NekoBox

```json
{
    "relay": {
        "server": "chika.example.com:443",
        "uuid": "ee48f7be-6ae9-5654-9b61-8466aa8e16bc",
        "password": "chika",
        "ip": "10.0.0.1",
        "congestion_control": "bbr",
        "alpn": ["h3"]
    },
    "local": {
        "server": "127.0.0.1:50000"
    },
    "log_level": "info"
}
```

Configure as needed.

## ShadowTLS

Reference :

* https://github.com/ihciah/shadow-tls/wiki/Run-with-Docker-Compose
* https://github.com/ihciah/shadow-tls/issues/70
* [How to Run · ihciah/shadow-tls Wiki · GitHub](https://github.com/ihciah/shadow-tls/wiki/How-to-Run#parameters)
* https://github.com/ihciah/shadow-tls/blob/master/docker-compose.yml
* [GitHub - ihciah/shadow-tls: A proxy to expose real tls handshake to the firewall](https://github.com/ihciah/shadow-tls)

**Prerequisites:**

Install Docker both server and client side.

```shell
sudo apt install docker-compose
```

**VPS :**

Create a file `docker-compose.yml` and fill it with the following code:

Example 1 :

```yaml
version: '3.5'
services:
  shadowsocks:
    image: shadowsocks/shadowsocks-libev
    restart: always
    command: /bin/sh -c 'exec ss-server -s 0.0.0.0 -p 24000 -k EXAMPLE_PASSWORD_SS -m chacha20-ietf-poly1305 -t 300'
  shadow-tls:
    image: ghcr.io/ihciah/shadow-tls:latest
    restart: always
    ports:
      - "8443:8443"
    environment:
      - MODE=server
      # - V3=1
      - LISTEN=0.0.0.0:8443
      - SERVER=shadowsocks:24000
      - TLS=cloud.tencent.com:443
      - PASSWORD=EXAMPLE_PASSWORD_ST
    depends_on:
      - shadowsocks
```

Example 2 :

```yaml
version: '3.5'
services:
  shadowsocks:
    image: ghcr.io/shadowsocks/ssserver-rust:latest
    restart: always
    command: /bin/sh -c 'exec ssserver -s "[::]:24000" -k "3SYJ/f8nmVuzKvKglykRQDSgg10e/ADilkdRWrrY>
  shadow-tls:
    image: ghcr.io/ihciah/shadow-tls:latest
    restart: always
    ports:
      - "443:443"
    environment:
      - STRICT=1
      - MODE=server
      - V3=1
      - FASTOPEN=true
      - WILDCARD_SNI=all
      - LISTEN=0.0.0.0:443
      - SERVER=shadowsocks:24000
      - TLS=google.com:443
      - PASSWORD=st
    depends_on:
      - shadowsocks
```

Run container with :

```shell
docker-compose up -d
```

### Client

#### Linux

Create file `docker-compose.yml` and fill it with the following code:

```yaml
version: '3.5'
services:
  shadow-tls:
    image: ghcr.io/ihciah/shadow-tls:latest
    restart: always
    ports:
      - "3443:3443"
    environment:
      - MODE=client
      # - V3=1
      - LISTEN=0.0.0.0:3443
      - SERVER=YOUR_VPS_IP:8443
      - TLS=cloud.tencent.com
      - PASSWORD=EXAMPLE_PASSWORD_ST
  shadowsocks:
    image: shadowsocks/shadowsocks-libev
    restart: always
    command: /bin/sh -c 'exec ss-local -b 0.0.0.0 -l 1080 -s shadow-tls -p 3443 -k EXAMPLE_PASSWORD_SS -m chacha20-ietf-poly1305 -t 300'
    ports:
      - "1080:1080"
    depends_on:
      - shadow-tls
```

Run :

```shell
docker-compose up -d
```

Verify connection success :

```shell
curl ipinfo.io -x socks5h://127.0.0.1:1080 -v
```

**Notes:**

Stop docker:

```shell
docker-compose down
```

Remove docker:

```shell
docker-compose down
sudo apt remove docker docker-compose
sudo apt autoremove
```

Connect IP_VPS:3443 with shadowsocks protocol on your mobile phones or PCs will work or connect IP_VPS:1080 with sock5h protocol on browser proxy.
I use Windows, I use Docker for Windows.

#### Android

Configuration Reference :

* [ShadowTLS - sing-box](https://sing-box.sagernet.org/examples/shadowtls/)

With NekoBox.apk then use this config template :

```json
{
  "dns": {
    "fakeip": {
      "enabled": true,
      "inet4_range": "198.18.0.0/15",
      "inet6_range": "fc00::/18"
    },
    "independent_cache": true,
    "rules": [
      {
        "disable_cache": true,
        "inbound": [
          "tun-in"
        ],
        "server": "dns-fake"
      }
    ],
    "servers": [
      {
        "address": "https://8.8.8.8/dns-query",
        "address_resolver": "dns-direct",
        "strategy": "ipv4_only",
        "tag": "dns-remote"
      },
      {
        "address": "local",
        "address_resolver": "dns-local",
        "detour": "direct",
        "strategy": "ipv4_only",
        "tag": "dns-direct"
      },
      {
        "address": "local",
        "detour": "direct",
        "tag": "dns-local"
      },
      {
        "address": "rcode://success",
        "tag": "dns-block"
      },
      {
        "address": "fakeip",
        "strategy": "ipv4_only",
        "tag": "dns-fake"
      }
    ]
  },
  "inbounds": [
    {
      "listen": "0.0.0.0",
      "listen_port": 6450,
      "override_address": "8.8.8.8",
      "override_port": 53,
      "tag": "dns-in",
      "type": "direct"
    },
    {
      "domain_strategy": "",
      "endpoint_independent_nat": true,
      "inet4_address": [
        "172.19.0.1/28"
      ],
      "mtu": 9000,
      "sniff": true,
      "sniff_override_destination": false,
      "stack": "mixed",
      "tag": "tun-in",
      "type": "tun"
    },
    {
      "domain_strategy": "",
      "listen": "0.0.0.0",
      "listen_port": 2080,
      "sniff": true,
      "sniff_override_destination": false,
      "tag": "mixed-in",
      "type": "mixed"
    }
  ],
  "log": {
    "level": "trace"
  },
  "outbounds": [
    {
      "type": "shadowsocks",
      "tag": "proxy",
      "method": "2022-blake3-chacha20-poly1305",
      "password": "3SYJ/f8nmVuzKvKglykRQDSgg10e/ADilkdRWrrY9HU=",
      "detour": "shadowtls-out",
      "multiplex": {
        "enabled": false,
        "max_connections": 4,
        "min_streams": 4
      }
    },
    {
      "domain_strategy": "",
      "type": "shadowtls",
      "tag": "shadowtls-out",
      "server": "IP_VPS",
      "server_port": 443,
      "version": 3,
      "password": "st",
      "tls": {
        "enabled": true,
        "insecure": true,
        "server_name": "google.com",
        "utls": {
          "enabled": true,
          "fingerprint": "chrome"
        }
      }
    },
    {
      "tag": "direct",
      "type": "direct"
    },
    {
      "tag": "bypass",
      "type": "direct"
    },
    {
      "tag": "block",
      "type": "block"
    },
    {
      "tag": "dns-out",
      "type": "dns"
    }
  ],
  "route": {
    "auto_detect_interface": true,
    "rules": [
      {
        "outbound": "dns-out",
        "port": [
          53
        ]
      },
      {
        "inbound": [
          "dns-in"
        ],
        "outbound": "dns-out"
      },
      {
        "ip_cidr": [
          "224.0.0.0/3",
          "ff00::/8"
        ],
        "outbound": "block",
        "source_ip_cidr": [
          "224.0.0.0/3",
          "ff00::/8"
        ]
      }
    ]
  }
}
```

## Shadowsocks-rust

Shadowsocks rust version is more updated version.

### Server

Reference : https://crates.io/crates/shadowsocks-rust

Create config :

```json
{
   "server":"0.0.0.0",
   "server_port":80,
   "password":"x",
   "method":"chacha20-ietf-poly1305"
}
```

Download binary :

```shell
mkdir /root/shadowsocks/
cd /root/shadowsocks/
wget https://github.com/shadowsocks/shadowsocks-rust/releases/download/v1.16.2/shadowsocks-v1.16.2.x86_64-unknown-linux-musl.tar.xz
tar -xf shadowsocks-v1.16.2.x86_64-unknown-linux-musl.tar.xz
chmod +x ssserver
./ssserver -c config.json
```

### Client

Android :
Use NekoBox.apk

## Brook

Reference :

* [GitHub - txthinking/brook: A cross-platform programmable network tool. 一个跨平台可编程网络工具.](https://github.com/txthinking/brook)

### Server

Quick download and run :

```shell
wget https://github.com/txthinking/brook/releases/download/v20230606/brook_linux_amd64
chmod +x brook_linux_amd64 
./brook_linux_amd64 server -l :80 -p x
```

Other method to run is install nami for download brook :

```shell
bash <(curl https://bash.ooo/nami.sh)
nami install brook
brook server -l :9999 -p hello
```

Or you can download from latest release : [Releases · txthinking/brook · GitHub](https://github.com/txthinking/brook/releases)

### Client

At example I'm using cli version :

```shell
./brook_linux_arm7 client -s="IP_SERVER:80" -p x --socks5="127.0.0.1:1080" --udpovertcp
```

Then verify connection success with `curl` command :

```shell
curl -x socks5h://127.0.0.1:1080 http://ipinfo.io -v
```

Various Client can download here : [Releases · txthinking/brook · GitHub](https://github.com/txthinking/brook/releases)

Quick Link :

* [Apple](https://apps.apple.com/us/app/brook-network-tool/id1216002642)
* [Android](https://github.com/txthinking/brook/releases/latest/download/Brook.apk)
* [MAC](https://apps.apple.com/us/app/brook-network-tool/id1216002642)
* [Windows](https://github.com/txthinking/brook/releases/latest/download/Brook.exe)
* [Linux](https://github.com/txthinking/brook/releases/latest/download/Brook.bin)
* \[OpenWRT]([Releases · txthinking/brook · GitHub](https://github.com/txthinking/brook/releases))

### Various Running Mode

In order to use SSL, lets generate self-signed certificate for testing environment, my current *brook* workspace is `/root/brook/`.

```shell
openssl req -x509 -newkey rsa:4096 -keyout /root/brook/key.pem -out /root/brook/cert.pem -sha256 -days 3650 -nodes -subj "/C=XX/ST=StateName/L=CityName/O=CompanyName/OU=CompanySectionName/CN=CommonNameOrHostname"
```

#### WebSocket

Server Command :

```shell
./brook_linux_amd64 wsserver -l :80 -p x
```

Client Command :

```shell
./brook_linux_arm7 wsclient -s="ws://IP_SERVER:80" -p x --socks5="127.0.0.1:1080"
```

#### WebSocket Secure

Server Command :

```shell
./brook_linux_amd64 wssserver --domainaddress example.com:443 --cert /root/brook/cert.pem --certkey /root/brook/key.pem -p x
```

Client Command :

```shell
./brook_linux_arm7 wssclient --wssserver "wss://example.com" --address "IP_SERVER:443" --insecure -p x --socks5="127.0.0.1:1080"
```

#### QUIC Server

Server Command :

```shell
./brook_linux_amd64 quicserver --domainaddress example.com:443 -p x
```

Client Command :

```shell
./brook_linux_arm7 quicclient --quicserver "quic://example.com:443" --address "IP_SERVER:443" --insecure -p x --socks5="127.0.0.1:1080" 
```

## RESTLS

Ref :

https://github.com/3andne/restls

### Run shadowsocks server

Create `config.json` in `/root/shadowsocks` :

```json
{
   "server":"0.0.0.0",
   "server_port":8888,
   "password":"x",
   "method":"chacha20-ietf-poly1305"
}
```

Download and unzip Shadowsocks-rust :

```shell
wget https://github.com/shadowsocks/shadowsocks-rust/releases/download/v1.16.2/shadowsocks-v1.16.2.x86_64-unknown-linux-musl.tar.xz
tar -xf shadowsocks-v1.16.2.x86_64-unknown-linux-musl.tar.xz
chmod +x ssserver
./ssserver -c config.json
```

Dont forget to run as screen.

### Run restls server

Latest release : https://github.com/3andne/restls/releases

```shell
mkdir /root/restls
cd /root/restls
wget https://github.com/3andne/restls/releases/download/v0.1.1/restls-x86_64-unknown-linux-musl
chmod +x restls-x86_64-unknown-linux-musl
```

### Client

You will need this [Release Clash.Meta with Restls Support · 3andne/Clash.Meta · GitHub](https://github.com/3andne/Clash.Meta/releases/tag/v0.1.0-pre4-restls)

Example Configuration :

```yaml
port: 7890
socks-port: 7891
redir-port: 7892
mixed-port: 7893
tproxy-port: 7895
ipv6: false
mode: rule
log-level: silent
allow-lan: true
external-controller: 0.0.0.0:9090
## external-ui: /path/to/ui/folder/
## external-ui-name: xd
## external-ui-url: "https://github.com/MetaCubeX/metacubexd/archive/refs/heads/gh-pages.zip"
secret: ""
bind-address: "*"
unified-delay: true
profile:
  store-selected: true
dns:
  enable: true
  ipv6: false
  enhanced-mode: redir-host
  listen: 0.0.0.0:7874
  nameserver:
    - 8.8.8.8
    - 1.0.0.1
    - https://dns.google/dns-query
  fallback:
    - 1.1.1.1
    - 8.8.4.4
    - https://cloudflare-dns.com/dns-query
    - 112.215.203.254
  default-nameserver:
    - 8.8.8.8
    - 1.1.1.1
    - 112.215.203.254
proxies:
  - name: restls-tls13
    type: ss
    server: 54.169.105.185
    port: 443
    cipher: chacha20-ietf-poly1305
    password: x
    plugin: restls
    plugin-opts:
        host: "www.microsoft.com" # Must be a TLS 1.3 server
        password: x
        version-hint: "tls13"
        client-id: chrome # One of: chrome, ios, firefox or safari
  - name: restls-tls12
    type: ss
    server: 54.169.105.185
    port: 443
    cipher: chacha20-ietf-poly1305
    password: x
    plugin: restls
    plugin-opts:
        host: "vscode.dev" # Must be a TLS 1.2 server
        password: x
        version-hint: "tls12"
        client-id: firefox # One of: chrome, ios, firefox or safari
proxy-groups:
  - name: RESTLS
    type: select
    proxies:
      - restls-tls13
      - restls-tls12
      - DIRECT
rules:
  - MATCH,RESTLS
```

The connection is unstable.

## Quick Install

[GitHub - aleskxyz/reality-ezpz: Install sing-box/xray and configure vless / tuic / hysteria2 for reality or tls (letsencrypt) over different transport protocols (tcp, http, grpc and websocket) with user management capability in CLI, TUI and Telegram bot by a single command in docker compose!](https://github.com/aleskxyz/reality-ezpz)
