---
language: en
title: Redirect all URL while migrate to new Domain/Blog
tags:
  - CloudFlare
categories:
  - Developer
date: 2023-09-07 15:38:37+07:00
description: "Automatic redirect all url and domain to new domain with
  CloudFlare, example : https://anasfanani.id/p?q=yes to
  https://anasfanani.com/p?q=yes"
image: ""
---
If you plan to migrate to new domain, this article is for you. follow me the steps.

## Requirements

* CloudFlare account.
* Your domain allready on CloudFlare.

## Steps

Go to the cloudflare dashboard and select your old domain.

Navigate to **Rules** > **Redirect Rules**.

Click on **Create Rule** button.

Input your *Rule name* (required), as example i will input `movedomain`

When incoming requests match… select **All incoming requests**

Then in **URL redirect** input this : 


Name| Value
|--|--|
|Type|Dynamic|
|Expression|concat("https://yournewdomain.com", http.request.uri.path)  |
|Status code|301|



In Checkbox **Preserve query string** dont forget to tick it.

Verify redirect works with browse to your old domain name.
