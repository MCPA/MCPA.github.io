---
layout: post
title: "2015 SANS Holiday Hack Part 5"
category: pcap analysis attribution xor base64 image decoding Holiday Hack
author: "Leander"
date:   2016-01-02 12:22
---
![login](/images/2015-Holiday-Hack/2015-holiday-hack-part5-login.png)

The last part of the SANS 2015 Holiday Hack requires to you analyze everything found so far. All the pcaps recovered, any images, and any correlated information. It's useful to utilize wireshark and ImageMagick. Last, describe the process used to during discovery and attribution.
<!--break-->

------  

## Introduction

The last part of the Holiday Hack requires to to analyze all the information collected thus far and find the Mastermind behind the ATNAS Corp plot. I retrieved all zip files posted on each SuperGnome and extracted the information from them. The numbered zip files, such as <em>20141226101055.zip</em>, contained a packet capture of an email. Each SuperGnome email contained different data, but each one was from <em>c@atnascorp.com</em>. Each SuperGnome also contained a "factory_cam" zip file. 

## Analysis

9. Based on evidence you recover from the SuperGnomes’ packet capture ZIP files and any staticky images you find, what is the nefarious plot of ATNAS Corporation?
    * Emails from SuperGnome 1-4
        * SuperGnome01 Email - Email was from C to JoJo. The body discussed developing a "distributed surveillance system" and included an image in the email. The image was base64 encoded, so I needed to extract the base64 data, and then decode it. After decoding the image I found that it had a very broad and overall diagram of the Gnome plot. The image shows a "bad guy" using a SuperGnome as a Command-and-Control channel to collect still image snapshots from Gnomes.
        
        ![email attachment](/images/2015-Holiday-Hack/2015-holiday-hack-part1-SG01-email-attach.jpeg)
        
        * SuperGnome02 Email - Email was from CW to Maratha. The body discussed ordering 2 million parts. It appears each piece contributed to the Gnome's ability to snap photos. This is confirmed by the fact when we analyzed the firmware in Part 2 and found the CPU used was an ARM processor. The firmware was also based on an open source Linux OS called "OpenWrt" - which can probably be tied to the Wi-Fi adaptor as part of this kit. The Gnome kit consists of:
            + Ambarella S2Lm IP Camera Processor System-on-Chip (with an ARM Cortex A9 CPU and Linux SDK)
            + ON Semiconductor AR0330: 3 MP 1/3" CMOS Digital Image Sensor
            + Atheros AR6233X Wi-Fi adapter
            + Texas Instruments TPS65053 switching power supply
            + Samsung K4B2G16460 2GB SSDR3 SDRAM
            + Samsung K9F1G08U0D 1GB NAND Flash
            
        * SuperGnome03 Email - Email was from CLW, President and CEO of ATNAS Corporation, to burglerlackeys. The body discusses when thieving will commence! The email continues to describe that the operation will commence, after dark, on December 24, 2015. Each burgler gets a detailed itinerary of specific houses and an inventory of items to steal from each house, along with still photos of where to locate each item taken from the Gnomes. The message also includes the path to take optmized based on the hitlist. It appears the hit-list is based on the initial hit list provides in the files directory for each Gnome: present, gift, holiday, stocking, candy, tree, ribbon, amazon, walmart, newegg, thinkgeek, edbyellen, toysrus, merry, gucci, apple, laptop, money, password, combination, chimney, cookies, and santa. If caught each burgler should claim to be Santa and that they plan to repair the items they are trying to steal. All proceeds are split 50-50. We also find out why CLW named the company ATNAS - it's the reverse of Santa.
        
        * SuperGnome04 Email - Email was from Cindy Lou Who to Dr. O'Malley, her psychiatrist in Who-ville. Since we now know that every email from ```c@atnascorp.com``` was signed by C, CW, CLW, and finally Cindy Lou Who I can deduce that at least the person using the email is most likely the culprit and that the person is Cindy Lou Who. I can't confirm it though, because someone else could be using the computer without CLW's knowledge. Apparently, CLW was scared at age two when the Grinch first tried to ruin Christmas. CLW's hatred eventually grew so big that she devised the plot discussed in SG03 Email using technology to One-Up the Grinch, and ruin Christmas. This is pretty solid evidence so far.
        
    * GnomeNET Messages - The GnomeNET Messages discuss an issue with the camera feeds. It tells us that the scrambled image on SuperGnome01 was of the "boss's" office. Five images that were captured aparently garbled the image of the boss's office. Each of the five "factory_cam" images were stored on a seperate SuperGnome. Last, the GnomeNET messages tell us that the camera feed collector attempted to merge the feed by XORing them together - and this explains why the image was scrambled. Since I had 4 of the 5 images I began to used a small bash script to XOR the images.[^1]
    
        * At long last we now have the emails and a snapshot of the villan!
    
        

Bash Script used to XOR the camera feeds:
{% highlight bash linenos %}
﻿#!/bin/bash

# Start XOR with original camera feed overlap and first factory cam
# image from the first SuperGnome
echo "1st XOR on camera feed overlap error"
echo "Creating img_out1.png"
convert camera_feed_overlap_error.png factory_cam_1.png  -fx \
"(((255*u)&(255*(1-v)))|((255*(1-u))&(255*v)))/255" img_out1.png
echo

# Iterate over subsequent images recovered
for i in `seq 1 3`; 
do
    s=1
    let "s = s + i"
    echo "Creating img_out$s.png"
    convert "img_out$i.png" "factory_cam_$s.png"  -fx \
    "(((255*u)&(255*(1-v)))|((255*(1-u))&(255*v)))/255" \
    "img_out$s.png"
    echo
done
echo "Done"
echo
{% endhighlight %}
  
10. Who is the villain behind the nefarious plot.
    * Cindy Lou Who at Sixty Two was confirmed (by emails and this image of her at her desk) as the Mastermind behind the plot to steal all the presents on Christmas Eve. She built a distributed surveillance system and used the Gnomes to recon the best presents to steal.

    ![cindy lou who](/images/2015-Holiday-Hack/2015-holiday-hack-part5-cindylouwho.png)

------

------

## Endnotes

[^1]: I used a program called ImageMagick to xor the scrambled image with each factory_cam. Find more about the usage on [stackoverflow](http://stackoverflow.com/questions/8504882/searching-for-a-way-to-do-bitwise-xor-on-images).