---
language: en
title: Hide default CPanel redirect with CloudFlare
tags:
  - CloudFlare
  - CPanel
categories:
  - Developer
date: 2023-09-06 15:29:24+07:00
description: Hide cPanel redirect (/cpanel,/whm,/webmail) with Cloudflare. This
  will help to protect your cPanel login information and your real IP address
  from potential hackers.
image: ""
---
cPanel is a popular hosting control panel that provides a variety of features and tools to help you manage your website. However, the default cPanel redirect can be a security risk. This redirect sends users to the cPanel login page when they enter your domain name in their browser. This can make it easier for hackers to find your cPanel login information and your real IP address.

To hide the default cPanel redirect, you can use Cloudflare. Cloudflare is a content delivery network (CDN) that can help improve the performance and security of your website. To hide the default cPanel redirect with Cloudflare, follow these steps:

## Requirements

* A Cloudflare account.
* Your website must already be on Cloudflare.

## Steps
### Login to CloudFlare 

Go to the Cloudflare dashboard: https://dash.cloudflare.com/.
### Create Rules
Click on the **Rules** tab and select **Transform Rules**.

Scroll down to the **Custom Records** section.

Click on the **Create Rule** button.

In the **Name** field, enter a name for the rule, such as `Disable Default Redirect CPanel`.

In the **When incoming requests match…** section, select **Custom filter expression** and click on **Edit Expression**.
Enter the following value for the expression: 
```
 (http.request.uri.path eq "/cpanel") or (http.request.uri.path
    eq "/whm") or (http.request.uri.path eq "/webmail") 
```
This expression will match any request that comes in to the Cloudflare proxy with a URI path of `/cpanel`, `/whm`, or `/webmail`.    

In the **Path** section, select **Rewrite to…** and select **Static**. Then, enter `/not-found` in the field below. This will tell Cloudflare to rewrite any requests that match the expression to a page with the URI path of `/not-found`.

In the **Query** section, select **Preserve**. This will ensure that any query parameters that are included in the request are also  included in the rewritten request.

Click on the **Save** button.


That's it! Cloudflare will now redirect any requests that match the expression to a page with the URI path of `/not-found`. This will help to hide your cPanel login information and your real IP address from potential hackers.
