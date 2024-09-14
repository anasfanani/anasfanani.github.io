---
language: en
title: Mitm Proxy And Proxy Chain Linux Programs In Ubuntu
tags:
  - Proxy
  - MITM
categories:
  - Linux
date: 2024-04-01 08:00:00+07:00
description: Set up an SSL-enabled MITM proxy on Ubuntu and chain programs using
  Proxychains4 to inspect and/or modify any network input/output.
---
This is how to set up MITM proxy and proxychains4 in Linux Ubuntu.

## Setup Mitm Proxy

Update and install mitmproxy

```bash
sudo apt update
sudo apt install mitmproxy -y
```

Run and install certificate

Ref : [Certificates](https://docs.mitmproxy.org/stable/concepts-certificates/#quick-setup)

Run with :

```bash
mitmweb --no-web-open-browser --web-host 0.0.0.0
```

Then install certificate

```bash
curl --proxy 127.0.0.1:8080 --cacert ~/.mitmproxy/mitmproxy-ca-cert.pem https://example.com/
openssl x509 -in ~/.mitmproxy/mitmproxy-ca-cert.pem -inform PEM -out ~/.mitmproxy/mitmproxy-ca-cert.crt
sudo mkdir /usr/local/share/ca-certificates/extra
sudo cp  ~/.mitmproxy/mitmproxy-ca-cert.crt /usr/local/share/ca-certificates/extra/mitmproxy-ca-cert.crt
sudo update-ca-certificates
```

Verify certificate and https mitm is working

```bash
curl https://ipinfo.io -x 0.0.0.0:8080 -v
```

## Setup Proxychains4

```bash
sudo apt install proxychains4 -y
sudo sed -i -e 's/proxy_dns/# proxy_dns/g' -e 's/socks4\s\+127.0.0.1\s\+9050/# socks4        127.0.0.1 9050/g' -e '$a\http    0.0.0.0   8080' /etc/proxychains4.conf 
proxychains4 curl https://ipinfo.io 
```
