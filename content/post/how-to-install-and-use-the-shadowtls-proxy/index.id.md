---
language: id
title: Cara install dan menggunakan ShadowTLS Proxy
tags:
  - Proxy
categories:
  - Linux
date: 2023-10-07 20:28:20+07:00
description: Langkah cepat untuk menginstall ShadowTLS Proxy di Server dan
  Client menggunakan Docker, Windows/Linux Ubuntu
---
# ShadowTLS - a better TLS masquerading proxy

This article is translated from original page here [https://www.ihcblog.com/a-better-tls-obfs-proxy/](https://www.ihcblog.com/a-better-tls-obfs-proxy/ "https\://www.ihcblog.com/a-better-tls-obfs-proxy/") 

If you want skip read this just go click [Quick Start](#quick-start)

This article mainly analyzes the currently popular Trojan protocol and tries to propose a better solution based on the characteristics of the current middleman.

The implementation of this solution is ShadowTLS, and you can find the full code and precompiled binaries on [Github](https://github.com/ihciah/shadow-tls "Github") .

To hide traffic characteristics, one way is not to expose any characteristics, that is, shadowsocks: this type of protocol also encrypts the protocol header for transmission, so no obvious characteristics can be observed. The second way is to hide yourself among others. The simplest way is to disguise it as HTTP or TLS traffic, corresponding to simple-obfs and Trojan respectively.

Method 1 is now relatively easy to identify. It does not hit any protocol and the timing characteristics are consistent with web traffic. It is enough to think that it is this type of traffic. Method 2 has become more and more mainstream in recent years, and the most widely used one is the Trojan protocol (simple-obfs just adds an http protocol header at the beginning, which is too easy to identify and will not be analyzed here).

## How Trojans work

What the Trojan wants to do is encapsulate the traffic into a normal TLS traffic. Because TLS traffic is encrypted, it is difficult for a middleman to identify whether it is ordinary web traffic or encapsulated proxy traffic. In order to be more similar, Trojan also defends against active detection, and the browser can directly open the corresponding web page and respond normally.

So the main problems it wants to solve here are these:

1. Proxy request bearer: To be able to encode the proxy request into binary, the server side must be able to decode the request, establish a remote connection and relay traffic according to the request.
2. Distinguish between client and active probe traffic: Some means is needed to distinguish client requests and active probe requests, and handle them differently.
3. Subsequent processing of client and active detector traffic: The differentiated traffic is processed separately. Client traffic needs to be carried using the TLS protocol, and active detector traffic also needs to be able to act like http.

The official protocol specification is written here: [The Trojan Protocol](https://trojan-gfw.github.io/trojan/protocol.html "The Trojan Protocol") . Solving problem 1 is very simple, because the upper layer is exposed to the socks5 proxy, so just package the socks5 proxy request header directly (similar to shadowsocks).

The focus is on questions 2 and 3. The method here is to first establish a TLS session, and then use the first 56 bytes in the TLS connection for authentication. If these 56 bytes match a certain hash result of our preshared key, then we think that this traffic is sent by our client.

It is obvious that people will notice a problem here: As an attacker, after establishing a TLS session and sending an HTTP request smaller than 56 bytes, I can determine whether it is a Trojan server by determining whether it is stuck? Because it takes 56 bytes to distinguish who I am, routing cannot be done before the data reaches 56 bytes.

In fact this problem does not exist. Let's take a look at the details of the protocol design: this 56 byte is hex(SHA224(password)), and CRLF will be sent later. Isn't it strange? Why does a binary protocol need CRLF, something that only text protocols use? And isn't it more efficient to directly send SHA224 binary results than to send hex? In fact, this is the subtlety of protocol design.

This CRLF is actually designed to correspond to HTTP traffic. When processing on the server side, directly read_until CRLF, and then routing can be done. Because HTTP traffic must be processed after it sends CRLF.

So after reading the first CRLF, either hex(SHA224(password)) has been sent, or the first line of the HTTP request has been sent. In either case, we can already make routing distinctions. For example, if we find that the data is not enough for 56 bytes, we can directly determine it as actively detecting traffic without having to wait for 56 bytes to be received. And why hex is needed is to avoid the accidental inclusion of CRLF in the hash result affecting our judgment.

By the way, a digression: When researching this protocol, I read the implementation of the trojan c and go versions. In fact, there is a problem with the implementation of the golang version. Maybe the author did not get these tricks in the protocol design. It directly does a read. If the data is not enough or the hash does not match, it is determined as active detection of traffic. But we cannot take reading 56 bytes at a time for granted. TCP is a stream protocol, and reading 1 byte at a time is the return result that complies with the POSIX specification.

> Errata: After correction by netizen RPRX, the statement here is indeed wrong. What is read and written here is not the naked TCP flow but the TLS flow. TLS traffic itself is framed, which theoretically guarantees one-to-one correspondence between reading and writing.Errata: After correction by netizen RPRX, the statement here is indeed wrong. What is read and written here is not the naked TCP flow but the TLS flow. TLS traffic itself is framed, which theoretically guarantees one-to-one correspondence between reading and writing.
>
> Golang's official TLS library exposes an interface to the outside world `io.Reader`. `io.WriterThis` interface is a streaming interface and the officially implemented TLS library does not guarantee a corresponding relationship. Therefore, the statement here should be corrected to rely on specific behaviors under specific conditions rather than dependence on the interface.

## What's wrong with Trojan

Everything looks fine? We package all data into TLS, and the outside world cannot tell what the encrypted data is. It seems that we have been requesting a certain web site, and if we browse the web, the timing characteristics of the proxy traffic are also web traffic.

If you ignore some implementation features, the only thing exposed here is the SNI and the corresponding certificate. The target domain name of our request will be exposed in TLS Client Hello, and it may not be normal to request a niche domain name with large traffic for a long time.

## better ways to disguise

Is there a better way to disguise it? If we want to use TLS, we have to handle the handshake ourselves; if we want to handshake, we have to issue a certificate with our own domain name. It seems to be an unsolvable problem. .

Oh wait! We are just pretending to be TLS traffic, who says we really need to use TLS?

So can we do a "TLS show" to show the middleman? The server can directly proxy this performance data to the whitelisted servers of some large companies or institutions, so that the handshake seen by the middleman is a handshake with a legitimate certificate and a whitelisted domain name. After the handshake is completed, the client and server switch modes and use the established connection to transmit custom data.

Switching mode requires that both parties can sense the end of the handshake. Here we force the use of TLS1.2. After observing a Change Cipher Spec packet, read another Handshake packet to mark the completion of the handshake.

We don't want to implement data encryption and proxy protocol encapsulation ourselves, so the custom data here is directly processed by shadowsocks. Our ShadowTLS works as a wrapper for shadowsocks traffic. For the client, it adds a layer of handshake data to the traffic, and for the server, it strips off this layer of handshake data.

So far, if we assume the middleman:

1. No analysis of traffic after handshake
2. No active detection

Then our protocol works very efficiently. From the packet capture, we can see that we are really communicating with a trusted domain name through TLS from a man-in-the-middle perspective. According to feedback, this version from late August to early October 2022 has helped some people get rid of QoS issues for domain names.

## Problems with ShadowTLS protocol (v1)

Previously we only did a very simple "performance" and made two assumptions, but in fact these two assumptions are not true. We need to be able to deal with both of these issues.

### Dealing with traffic analysis

Normal TLS data will be communicated using the Application Data encapsulation package after the handshake is completed. Directly forwarding the shadowsocks data stream is completely inconsistent with the TLS protocol, and even wireshark will highlight subsequent data packets to indicate a problem. It is not difficult to solve this problem. We only need to encapsulate and decapsulate on both sides.

### Respond to active detection

If we are to be able to deal with active detection, we need to be able to do two things (the same as Trojan needs to do):

1. Distinguish between client traffic and active detection traffic
2. Correctly respond to proactive probe traffic

We need the client to give something special to judge that this is our client traffic. In order to avoid active detection, we must introduce a pre-shared key. But how?

In the Trojan protocol, just send the hash of the password directly. But we only have plaintext channels available here, so sending the hash of the password directly obviously exposes the password, which means that the password is no longer meaningful; and it is completely unable to prevent data replay.

## ShadowTLS v2 protocol design

### Server Challenge

Based on the plain text channel, we can only authenticate in the form of challenge-response. Normally, if we want to authenticate the client, we need the server to send a challenge. But in fact we cannot do this, because a normal https server cannot send back a challenge after the TLS handshake.

So can the challenge be hidden in a normal handshake? The requirements for the challenge are very simple, just be random and uncontrollable by the client. My idea is that in fact, the data sent by the server during the handshake process itself can be used as a challenge: it has random data, such as server random, and it is not controllable by the client.

Here I use all the data sent by the server during the handshake process as a challenge (of course, server random can also be used, but this requires parse TLS packets and awareness of TLS protocol details, which is a bit troublesome to implement and may introduce feature differentiation in details), like this The dependence on the details of the TLS protocol can be weakened as much as possible, so there is no need to rely on the details of the handshake behavior of TLS1.2.

### Client Response

We have a challenge, so how to respond? Obviously we need to authenticate the pre-shared key, so we `hmac(data, key)` can use it directly as the response (it can be simply understood as `hash(data+key)`, but it is better in terms of security, it can be obtained by streaming calculation, and there is no need to cache data).

How to send this Response data back? If used as separate packets, new distinguishing characteristics are introduced. So here we put this Response in the header of the first Application Data packet and send it to the Server side.

For this hmac, I use the first 8 bytes of hmac-sha1, and the security is good enough.

### Application Data

During the data forwarding process, Application Data is encapsulated and decapsulated. The question that needs to be considered here is, how big is a single Application Data packet under normal circumstances? In the current implementation, a buffer size is determined directly by tapping the head. However, in order to prevent this packet size from becoming a feature, it is necessary to investigate the implementation of the TLS library and capture packets to observe and determine a reasonable maximum value.

### Handling active probe traffic

We can simplify the server model as follows: connect to the handshake server by default; switch to the data server if the hmac authentication passes.

For active detection traffic, it is impossible for it to guess the 8 byte hmac correctly, so it will never switch to the data server. In order to avoid unnecessary hash calculations, when the first N Application Data packets fail to pass the hmac verification (N is taken here instead of 1 because it is not sure whether the Application Data is sent, it must mark the end of the handshake), it will switch directly to Direct proxy, no further attempts to hmac calculation and verification will be made.

The detailed protocol design is written here , you can refer to it if you are interested.

### ShadowTLS vs. Trojan

Compared with Trojan, ShadowTLS does not need to issue a certificate by itself (you can directly use the trusted domain name of a large company or institution), nor does it need to start a disguised HTTP service by itself (because the data is directly forwarded to the website corresponding to the trusted domain name). Using a trusted domain name can Further weaken the features and hide the trees in the forest.

Both ShadowTLS and Trojan can handle active detection, and when opened directly using a browser, the HTTP page can be accessed normally.

## Go further

UPDATED AT 2022-11-13

More than a month has passed since the v2 implementation was released, and ShadowTLS has achieved good results: when Trojan was banned on a large scale in the past period, ShadowTLS is still available. Currently, both ShadowRocket and Surge support this protocol (although I still have no money to buy Surge).

But in fact, there are many areas that can be improved:

### TLS fingerprinting issue

For Server, we forward traffic directly, and there is no fingerprint problem; but Client is implemented by ourselves, and we expect it to look the same as a browser or other normal client, but in fact it may not be similar enough. If you capture the packet and look at the Client Hello packet sent by Chrome, you can clearly see that it contains a lot of Extension fields, and these fields are not automatically attached when we use rustls; and the default selection of Cipher and Hash for different clients Lists etc are different.

So one area that can be improved is to provide multiple Client TLS Profiles for users to choose from.

### Traffic hijacking problem

[This issue](https://github.com/ihciah/shadow-tls/issues/30 "This issue") mentioned a real problem: What if someone hijacks client-side traffic to the handshake server?

First, who does the Client trust? Until it completes the TLS handshake, it behaves like a normal TLS Client. To gain the client's trust, you first need to be able to pass certificate verification and complete the TLS handshake. In addition to our Server, the people who can do this include the handshake server itself, and other intermediaries that proxy the handshake.

I think we can assume here that the handshake server is not controllable by the middleman, and its certificate cannot be held by the middleman. So now the focus is on the middlemen who act as agents of handshake like us. The man-in-the-middle does not need to decrypt the traffic, its goal is to identify whether we are a normal connection. So although it does not get the decryption key, it can still hijack and replay the traffic to achieve its goal:

1. Directly hijack the entire connection to the handshake server (this is the attack method mentioned in the issue): the secret will be exposed after the Client completes the protocol switch, and the handshake server will return Encrypted Alert.
2. Normally proxy traffic, but secretly lose or reorder an Application Data: Normally, Encrypted Alert should be returned, but because we do not do message Authentication or Encryption, we are actually not aware of this matter. This matter will Being thrown to lower level services. We rely on the underlying service to disconnect to return Encrypted Alert.
3. Observing disconnected connections: As mentioned in 2, we need to properly handle the problem of disconnected connections, whether it is a normal shutdown or an abnormal shutdown. But the current implementation does not send Encrypted Alert.
4. Merge adjacent Application Data: Under normal circumstances, there will be a sequence number and MAC inside the TLS protocol, but our encapsulation currently does not, so if the hijacker merges adjacent Application Data and the connection is still normal, then it can also be found to be disguised TLS.

However, these problems (except problem 3) need to be able to hijack traffic on the main link to take effect. If there are no other hints, hijacking all outbound TLS traffic is still very risky. So I think this problem is actually not a big one.

### What's Better Protocol?

We can simply fix some of the problems mentioned earlier (these are implementation issues, not protocol issues): provide Client TLS Profile and send Encrypted Alert when the connection is closed. But what about the remaining traffic hijacking issues?

#### Against attacks that hijack directly to the handshake server

We can see that the key to the problem is that the Client did not authenticate the Server (only the certificate was authenticated). The server needs to indicate its identity. If it is placed in an Extension in Server Hello, it may become an obvious feature; if it is directly included in subsequent traffic, it will also confuse the detector and prevent normal decryption.

We need a place like this: it is sent by the Server, it is a random number itself, and it has no effect if we modify it. It is best to send it after sending the Server Random (this way you can use Server Random to defend against replay attacks). We can actually hide some things on IP packets, but this requires us to have system administrator rights and will introduce stronger environmental restrictions, so we try our best to find such places on top of TCP.

So we can find a hiding point that meets the conditions: Session ID (Session ID for TLS 1.3 only). Since we fully trust that Server Random must be Random, we can do something simpler here: if the relayed Server Hello packet contains a 32-bit Session ID, replace the ID with the HMAC of Server Random. Since TLS 1.2 defaults to this field being empty, we cannot rashly insert this value for TLS 1.2 to avoid becoming a feature.

#### Reshape for traffic

Our Application Data encapsulation can be reshaped, but real TLS traffic cannot, so we need to refer to the TLS approach in this layer of encapsulation and carry some data for verification. This kind of verification helps to find problems 2 and 4 mentioned above, and then you only need to respond to the Alert and disconnect.

Of course, these have not been implemented yet (until 2022-11-13). If you are interested, you are welcome to raise an issue to claim your contribution!

Implementing fixes and preventing direct hijacking to the handshake server can maintain compatibility with older versions of Client, but if you want to add MAC to Application Data, you have to change the protocol (maybe v3) ~

## Summarize

[ShadowTLS](https://github.com/ihciah/shadow-tls "ShadowTLS") is implemented based on Rust + Monoio, and can bring better IO performance based on io_uring and thread-per-core models (but because Monoio currently does not support Windows, Windows users cannot use it for the time being, and it is recommended to use wsl ) .

In summary, this article attempts to analyze the mainstream TLS-based proxy protocol, proposes a better protocol design for its possible defects, and provides the corresponding implementation. You can find the corresponding code here .

## Quick Start

I'm having trouble understanding the original tutorial from here https://github.com/ihciah/shadow-tls/wiki/Run-with-Docker-Compose
then I looked for a way how I can try shadowtls running on a vps server and I can use it in seconds.

I saw the issue in the shadowtls repository https://github.com/ihciah/shadow-tls/issues/70

there he gives a good quick start to trying out shadow tls. Maybe you can follow the method from that page directly or follow the method that I tried which is currently successful.

**Prerequisites:**

* A VPS & Client with Docker and Docker Compose installed.

**Instructions:**

1. On the VPS, create a file called `docker-compose.yml` and fill it with the following code:

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

2. Replace the following values with your own (I suggest just copy and paste on your server, you can change later):

* `EXAMPLE_PASSWORD_SS`: The password for your Shadowsocks.
* `EXAMPLE_PASSWORD_ST`: The password for your ShadowTLS.
* `TLS`: The TLS domain for your ShadowTLS.

3. Run the following command to start the Docker containers:

```sh
docker-compose up -d
```

4. On your client computer, create a file called `docker-compose.yml` and fill it with the following code:

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

5. Replace the following values with your own (I suggest just copy and paste on your client machine, you can change later):

* `YOUR_VPS_IP`: The IP address of your VPS.
* `EXAMPLE_PASSWORD_SS`: The password for your Shadowsocks server.
* `EXAMPLE_PASSWORD_ST`: The password for your ShadowTLS account.
* `TLS`: The TLS domain for your ShadowTLS account.

6. Run the following command to start the Docker containers:

```sh
docker-compose up -d
```

Once the Docker containers have started, you can verify the connection is success with curl :

To test the connection, run the following command:

```sh
curl ipinfo.io -x socks5h://127.0.0.1:1080 -v
```

If the connection is successful, you should see your IP address and other information about your connection.

**To stop the Docker containers:**

```sh
docker-compose down
```

Connect cn_vps:3443 with shadowsocks protocol on your mobile phones or PCs will work or connect cn_vps:1080 with sock5h protocol on browser proxy.
I use Windows, I use Docker for Windows.
Thank you, this post is just for personal reference if one day I forget how to install shadowtls.
