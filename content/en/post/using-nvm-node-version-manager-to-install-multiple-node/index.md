---
title: Using NVM (Node Version Manager) to install multiple node
tags:
  - Node.js
categories:
  - JavaScript
  - Linux
date: 2022-10-23 16:17:57+07:00
description: >-
  Well, now about node, node allows developers to write JavaScript code that
  runs directly in a computer process itself instead of in a browser


  So here i will show tutorial how to install multiple node using nvm.
image: 1200px-node.js_logo.svg.png
---
Well, now about node, node allows developers to write JavaScript code that runs directly in a computer process itself instead of in a browser

So here i will show tutorial how to install multiple node using nvm, at github, https://github.com/nvm-sh/nvm
Says : 
>`nvm` allows you to quickly install and use different versions of node via the command line.

What is nvm ?
-------------

`nvm` is a version manager for node.js, designed to be installed per-user, and invoked per-shell. nvm works on any POSIX-compliant shell (sh, dash, ksh, zsh, bash), in particular on these platforms: unix, macOS, and windows WSL.

Installing
----------
To install or update nvm, you should run the install script. To do that, you may either download and run the script manually, or use the following cURL or Wget command:

With curl :

    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash

With wget :

    wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.2/install.sh | bash


Then use it like this :

Switch to node  v16 :

    $ nvm use 16
    Now using node v16.9.1 (npm v7.21.1)
    $ node -v
    v16.9.1

Switch to node  v14 :

    $ nvm use 14
    Now using node v14.18.0 (npm v6.14.15)
    $ node -v
    v14.18.0

Install node v12 :

    $ nvm install 12
    Now using node v12.22.6 (npm v6.14.5)
    $ node -v
    v12.22.6
Simple as that!


Thanks


  [1]: https://upload.wikimedia.org/wikipedia/commons/thumb/d/d9/Node.js_logo.svg/1200px-Node.js_logo.svg.png