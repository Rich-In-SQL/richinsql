---
title: 'PowerShell & Corporate Proxy'
date: 2019-06-25T17:47:58+01:00
author: Rich
layout: post
permalink: /powershell-corporate-proxy
categories:
  - powershell
tags:
  - gallery
  - powershell
  - proxy
---
I have been having a problem at work where when I try to either install or update a module on my work laptop I get the following error message.

![](/img/PowershellProxy.jpg)

A quick Google of the error will return a plethora of information on [how to fix](https://stackoverflow.com/questions/14263359/access-web-using-powershell-and-proxy) the issue, with the main focus being on the following two lines;

```
    $webclient=New-Object System.Net.WebClient
    $webclient.Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials
```

For the majority of people, this seemed to resolve the issue and they could go about there business, for me however it didn't resolve the problem, at least it wasn't the sole fix.

![](/img/PowershellProxy2.jpg)

#### Offline Installer

I had a little bit of time this morning so I thought I would have a go at Installing dbatools using the offline method, I loaded up PowerShell and typed the command

```
    Invoke-WebRequest -Uri powershellgallery.com/api/v2/package/dbatools -OutFile c:\temp\dbatools.zip
```

However, I was presented with the following.

![](/img/PowershellProxy3.jpg)

What actually appeared to be happening was [powershellgallery.com](https://www.powershellgallery.com/) had been blocked by an upstream web proxy policy, I requested that [powershellgallery.com](https://www.powershellgallery.com/)/* be white listed for the admin team, waited a little while and tried again.

![](/img/PowershellProxy4.jpg)

Wallah, it works now. I still need the two magic lines of code, which are in my PowerShell profile now.

Hopefully this helps someone else.