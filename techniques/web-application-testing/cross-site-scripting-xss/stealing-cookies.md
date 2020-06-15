---
description: >-
  Exploiting web application vulnerabilities to steal authentication cookies and
  escalate privilege or assume identity of another user
---

# Stealing Cookies

## Overview

Many websites use [cookies](https://kb.iu.edu/d/agwm) to authenticate users or to keep track of users across the application after authentication has occurred. Cookies can correlate to a specific user id, and lets the server know what authority the user has to access resources or make changes. 

Often, the use of stealing cookies in the context of a CTF involves escalating privilege from a normal user account to that of an admin. This could involve stealing the cookie of an authenticated admin in order to gain the privilege of that user. To do  so, one might exploit a cross site scripting attack, or XSS, in which an administrator or other user can be tricked to clicking a link which sends the attacker his or her cookies. Read more about XSS [here](./).

## Guide to Stealing Cookies

Stealing cookies is a _server side_ attack. This means that aside from the CTF player, another user has to be interacted with to trigger the vulnerability. In CTF context, this   could mean interacting with the CTF organizers/facilitators, or it could mean the organizers have set up some kind of automated process. An example of an automated process would be automated actions on [Selenium](https://www.selenium.dev/). 

### Step 1, Find a [XSS Vulnerability ](./)

To find an [XSS vulnerability](./), you often need to test input fields on a webpage. These can be comment boxes, input forms, or anywhere else you can submit text to the server. Below are some good resources for testing various inputs which _might_ trigger an [XSS vulnerability](./). 

{% embed url="https://portswigger.net/web-security/cross-site-scripting/cheat-sheet" caption="PortSwigget Cross Site Scripting Cheat Sheet" %}

{% embed url="https://owasp.org/www-community/xss-filter-evasion-cheatsheet" caption="OWASP XSS Filter Cheat Sheet" %}

### Step 2, Create Payload

To effectively steal another user's cookie, your payload needs to do two things; it needs to collect a session cookie, and it needs to display the cookie so you can see it. The later of the two steps is different depending if you are competing in a local network CTF or one hosted over the internet. 

#### Local Network Payload

A local network CTF can be defined as a CTF in which the target CTF application and your attack machine are on the same network. These can include a [VulnHub](../../../categories/boot-2-root/vulnhub.md) [boot 2 root](../../../categories/boot-2-root/) challenge where you are running the CTF and your attack machine in the same virtualization software. 

