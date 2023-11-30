---
title: SSH to a Remote Server without Public IP
tags:
  - SSH
  - CloudFlare
  - Free
categories:
  - Linux
date: 2022-12-21 22:40:53+07:00
description: SSH to a Remote Server is need a Public IP, I will share how to do
  this without Public IP. Using cloudflare tunnel
image: ssh.png
---
There many method to do this, i will share first method is with cloudflared tunnel.

Lets start.

# Cloudflare Zero Trust

## Requirements

You will need account and a domain.

## Step By Step

### Go to dashboard

Goto dashboard <https://one.dash.cloudflare.com/> 

![tunnels link](1.png "Tunnels Link")

### Create tunnels

Click on a button

![create tunnels](2.png)

### Configure SERVER

#### Name your tunnels

Give random name

![create  tunnels name](3.png)

#### Install connector

Here you need to copy the command and execute in server

![configure server zerotrust token](4.png)

Example :

```
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb && 

sudo dpkg -i cloudflared.deb && 

sudo cloudflared service install TOKEN
```

the azure server doestn have sudo, so remove the sudo

```
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb && 

dpkg -i cloudflared.deb && 

cloudflared service install TOKEN
```

![paste command](5.png)

You will got message `INF Linux service for cloudflared installed successfully`

#### Route tunnels

Go back to cloudflare then setup a Route 

![setup route](6.png)

Why use localhost:2222 ?
Because my port ssh server is 2222, default port is 22, so if you use default port, use a localhost:22

Save, and see : 

![active tunnels](7.png)

### Configure CLIENT

#### Setup cloudflared

Download cloudflared for client from <https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/>

I am using windows , so i download Windows version

<https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64.exe>

Save to any folder , example `C:/App/bin`, then configure your PATH variable

Click START, type `path`

![configure path](8.png)

Then follow this step : 

![follow this](9.png)

Rename to `cloudflared.exe` 

![rename](10.png)

#### Setup SSH

Open SSH configuration

Goto `C:UsersYOUR USERNAME.ssh`

![screenshoot ssh  folder](11.png)

if you have a config file, then modify `config` file, else create new file,fi

Paste code like this

```
Host CHANGE_ME
  HostName CHANGE_ME
  User root
  ProxyCommand C:/App/bin/cloudflared.exe access ssh --hostname %h
```

change CHANGE_ME to your host, example : mytunnelfromzaure.anasfanani.com

Example :

```
Host mytunnelfromzaure.anasfanani.com
  HostName mytunnelfromzaure.anasfanani.com
  User root
  ProxyCommand C:/App/bin/cloudflared.exe access ssh --hostname %h
```

#### Connect SSH

Open your command prompt, type `ssh root@mytunnelfromzaure.anasfanani.com`

![enter image description here](12.png)

## Conclusion

All step is not simple, if you need a simple step you can use other method

More Method will updated, so stay tuned, dont forget comment