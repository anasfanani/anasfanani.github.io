---
language: en
title: WebSocket ScrCpy (ws-scrcpy) With Termux
tags:
  - Termux
categories:
  - Android
date: 2024-04-01 08:50:00+07:00
description: Run ws-scrcpy In Termux, Remote Android phone with browser using Termux.
---
The installation is quite tricky.

## Download Termux

You can choose which termux version, but right now I'm using this.

[Download Termux 0.118.0](https://github.com/termux/termux-app/releases/download/v0.118.0/termux-app_v0.118.0+github-debug_universal.apk)[](https://github.com/termux/termux-app/releases/download/v0.118.0/termux-app_v0.118.0+github-debug_universal.apk)

## Installation

The Installation is simple, you just need to this command:

```bash
pkg update && pkg upgrade -y
pkg install openssl-tool nodejs-lts git python3 build-essential git android-tools -y
git clone https://github.com/NetrisTV/ws-scrcpy
cd ws-scrcpy
npm install -g node-gyp
npm ci
npm run start
```

Thats all.

## Troubleshoot

You may encounter a problem while install, latest version need to install pupeter *(the correct name is puppeteer and hard to write)* in ws-scrcpy, i dont know why, but quick fix is using `.puppeteerrc.cjs` for able to install pupeter, you may take a look example usage from <https://github.com/anasfanani/ilmupedia-auto-buy/blob/main/.puppeteerrc.cjs>

Create a new file in `ws-scrcpy` folder named as `.puppeteerrc.cjs` and fill with:

```javascript
const { join } = require('path');

let config = {};

if (process.env.PREFIX === '/data/data/com.termux/files/usr') {
    config.executablePath = '/data/data/com.termux/files/usr/bin/chromium-browser';
}

config.cacheDirectory = join(__dirname, '.cache', 'puppeteer');

/**
 * @type {import("puppeteer").Configuration}
 */
module.exports = config;
```

Before run `npm ci` again, you need to run this command first:

```bash
pkg install tur-repo x11-repo
pkg update
pkg install chromium
```

Then run `npm ci` again.
