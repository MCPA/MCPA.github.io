---
layout: post
title: "2015 SANS Holiday Hack Part 3"
category: reconaissance shodan Holiday Hack
author: "Leander"
date:   2016-01-01 12:22
---
![login](/images/2015-Holiday-Hack/2015-holiday-hack-part3-login.png)

The second part of the holiday hack required you to [return to the Dosis neighborhood](https://quest.holidayhackchallenge.com/) again, find Tom H, and find/confirm each SuperGnome IP address with Tom H. For this challenge you will need to create a [shodan](http://shodan.io) account.
<!--break-->

------  

## Introduction

![intro](/images/2015-Holiday-Hack/2015-holiday-hack-part3-intro.png)

This challenge is meant to ensure that the participants in the Holiday Hack don't get into trouble by targeting the wrong host machines. The SANS organizers find a very creative way to do that by essentially creating a scavenger hunt for your targets. The biggest clue is in the challenge description and talking to Dan. Which should tip off players to search for shodan.

![dan](/images/2015-Holiday-Hack/2015-holiday-hack-part3-dan.png)

## Solution

Shodan is a search engine that looks for internet-connected devices. The search engine also discovers who is connected to the devices, where they are located, and who is using them. Last, it will also do some basic enumeration and banner grabbing. My first search on the Shodan website was for ```supergnome```, since the challenge tells me I need to look for a series of machines by that name. Which gives me 5 results. Each result is powered by SuprGnome by AtnasCorp, and AtnasCorp happens to be the compan behind the WiFi Gnomes. The next step is to verify the IP addresses with Dan. Afterwards, we have now confirmed our five targets.

![solved](/images/2015-Holiday-Hack/2015-holiday-hack-part3-solved.png)

## Analysis

5. What are the IP addresses of the give SuperGnomes scattered around the world, as verified by Tom Hessman in the Dosis neighborhood?  
  * SuperGnome01 - 52.2.229.189
  * SuperGnome02 - 54.34.3.80
  * SuperGnome03 - 52.64.191.71
  * SuperGnome04 - 52.192.152.132
  * SuperGnome05 - 54.233.105.81
6. Where is each SuperGnome located geographically?
  * SuperGnome01 - Ashburn, United States
  * SuperGnome02 - Boardman, United States
  * SuperGnome03 - Sydney, Australia
  * SuperGnome04 - Tokyo, Japan
  * SuperGnome05 - Brazil