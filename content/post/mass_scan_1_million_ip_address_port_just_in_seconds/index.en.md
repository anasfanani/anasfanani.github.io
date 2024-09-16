---
language: en
title: Mass Scan 1 Million IP Address & Port Just In Seconds
tags:
  - Tools
  - Security
categories:
  - Linux
date: 2024-09-15 18:27:00+07:00
description: Ultra fast network scanner, scanning entire Internet in under 5
  minutes, transmitting 10 million packets per second, from a single machine,
  scanning Cloudflare, Cloudfront etc...
---
# Prepare

Use tools called masscan, checkout the official repository <https://github.com/robertdavidgraham/masscan>.

## Windows

Checkout from this repository <https://github.com/Arryboom/MasscanForWindows>.

Or just download directly

<https://github.com/Arryboom/MasscanForWindows/blob/master/masscan64.exe>
<https://github.com/Arryboom/MasscanForWindows/blob/master/masscan32.exe>

Then install <https://www.winpcap.org/>

## Linux

```
sudo apt update
sudo apt install libpcap-dev masscan -y
```

## Android

Compile itself with some file modification from <https://github.com/robertdavidgraham/masscan/issues/627>.

Note: in Android version, can't use 4G data to scan, must connected to other router.

# Usage

Usage is similar to `nmap`, checkout <https://github.com/robertdavidgraham/masscan#usage>.

```
root ➜ ~ $ masscan --help
MASSCAN is a fast port scanner. The primary input parameters are the
IP addresses/ranges you want to scan, and the port numbers. An example
is the following, which scans the 10.x.x.x network for web servers:
 masscan 10.0.0.0/8 -p80
The program auto-detects network interface/adapter settings. If this
fails, you'll have to set these manually. The following is an
example of all the parameters that are needed:
 --adapter-ip 192.168.10.123
 --adapter-mac 00-11-22-33-44-55
 --router-mac 66-55-44-33-22-11
Parameters can be set either via the command-line or config-file. The
names are the same for both. Thus, the above adapter settings would
appear as follows in a configuration file:
 adapter-ip = 192.168.10.123
 adapter-mac = 00-11-22-33-44-55
 router-mac = 66-55-44-33-22-11
All single-dash parameters have a spelled out double-dash equivalent,
so '-p80' is the same as '--ports 80' (or 'ports = 80' in config file).
To use the config file, type:
 masscan -c <filename>
To generate a config-file from the current settings, use the --echo
option. This stops the program from actually running, and just echoes
the current configuration instead. This is a useful way to generate
your first config file, or see a list of parameters you didn't know
about. I suggest you try it now:
 masscan -p1234 --echo
```

# Example 

Some snippet example to run scan.

## Cloudflare

Scan from all Cloudflare IP list with port 80 and rate 100000, show banners.

IP list from <https://www.cloudflare.com/ips-v4/#>

```
masscan64.exe 173.245.48.0/20 103.21.244.0/22 103.22.200.0/22 103.31.4.0/22 141.101.64.0/18 108.162.192.0/18 190.93.240.0/20 188.114.96.0/20 197.234.240.0/22 198.41.128.0/17 162.158.0.0/15 104.16.0.0/13 104.24.0.0/14 172.64.0.0/13 131.0.72.0/22 -p80 --rate=100000 --banners -oG CloudFlare80.txt
```

## Cloudfront

Scan all Cloudfront IP port 80.

IP list from <https://d7uri8nf7uskq.cloudfront.net/tools/list-cloudfront-ips>

```
masscan64.exe 120.52.22.96/27 205.251.249.0/24 180.163.57.128/26 204.246.168.0/22 111.13.171.128/26 18.160.0.0/15 205.251.252.0/23 54.192.0.0/16 204.246.173.0/24 54.230.200.0/21 120.253.240.192/26 116.129.226.128/26 130.176.0.0/17 108.156.0.0/14 99.86.0.0/16 205.251.200.0/21 13.32.0.0/15 120.253.245.128/26 13.224.0.0/14 70.132.0.0/18 15.158.0.0/16 111.13.171.192/26 13.249.0.0/16 18.238.0.0/15 18.244.0.0/15 205.251.208.0/20 65.9.128.0/18 130.176.128.0/18 58.254.138.0/25 54.230.208.0/20 3.160.0.0/14 116.129.226.0/25 52.222.128.0/17 18.164.0.0/15 111.13.185.32/27 64.252.128.0/18 205.251.254.0/24 54.230.224.0/19 71.152.0.0/17 216.137.32.0/19 204.246.172.0/24 18.172.0.0/15 120.52.39.128/27 118.193.97.64/26 18.154.0.0/15 54.240.128.0/18 205.251.250.0/23 180.163.57.0/25 52.46.0.0/18 52.82.128.0/19 54.230.0.0/17 54.230.128.0/18 54.239.128.0/18 130.176.224.0/20 36.103.232.128/26 52.84.0.0/15 143.204.0.0/16 144.220.0.0/16 120.52.153.192/26 119.147.182.0/25 120.232.236.0/25 111.13.185.64/27 3.164.0.0/18 54.182.0.0/16 58.254.138.128/26 120.253.245.192/27 54.239.192.0/19 18.68.0.0/16 18.64.0.0/14 120.52.12.64/26 99.84.0.0/16 130.176.192.0/19 52.124.128.0/17 204.246.164.0/22 13.35.0.0/16 204.246.174.0/23 3.172.0.0/18 36.103.232.0/25 119.147.182.128/26 118.193.97.128/25 120.232.236.128/26 204.246.176.0/20 65.8.0.0/16 65.9.0.0/17 108.138.0.0/15 120.253.241.160/27 64.252.64.0/18 -p443 --rate=100000 --banners -oG CLOUDFRONT_GLOBAL_IP_LIST.txt
masscan64.exe 13.113.196.64/26 13.113.203.0/24 52.199.127.192/26 13.124.199.0/24 3.35.130.128/25 52.78.247.128/26 13.233.177.192/26 15.207.13.128/25 15.207.213.128/25 52.66.194.128/26 13.228.69.0/24 52.220.191.0/26 13.210.67.128/26 13.54.63.128/26 43.218.56.128/26 43.218.56.192/26 43.218.56.64/26 43.218.71.0/26 99.79.169.0/24 18.192.142.0/23 35.158.136.0/24 52.57.254.0/24 13.48.32.0/24 18.200.212.0/23 52.212.248.0/26 3.10.17.128/25 3.11.53.0/24 52.56.127.0/25 15.188.184.0/24 52.47.139.0/24 3.29.40.128/26 3.29.40.192/26 3.29.40.64/26 3.29.57.0/26 18.229.220.192/26 54.233.255.128/26 3.231.2.0/25 3.234.232.224/27 3.236.169.192/26 3.236.48.0/23 34.195.252.0/24 34.226.14.0/24 13.59.250.0/26 18.216.170.128/25 3.128.93.0/24 3.134.215.0/24 52.15.127.128/26 3.101.158.0/23 52.52.191.128/26 34.216.51.0/25 34.223.12.224/27 34.223.80.192/26 35.162.63.192/26 35.167.191.128/26 44.227.178.0/24 44.234.108.128/25 44.234.90.252/30 -p80 --rate=100000 --banners -oG CLOUDFRONT_REGIONAL_EDGE_IP_LIST.txt
```

## Local Scan

Scan all local network, all port.

```
masscan 192.168.1.0/24 -p0-65535 --banners --rate 100000 -oG localscan.txt
```
