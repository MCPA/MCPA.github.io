---
layout: post
title: "2015 SANS Holiday Hack Part 2"
category: squashfs firmware analysis mongodb Node.js Holiday Hack
author: "Leander"
---
![login](/images/2015-Holiday-Hack/2015-holiday-hack-part2.png)

<strong>Description:</strong> The second part of the holiday hack required you to [return to the Dosis neighborhood](https://quest.holidayhackchallenge.com/), find Jessica, and unwrap the secrets of the Gnome's firmware. For this challenge you will need to download the firmware-mod-kit ([FMK](https://firmware-mod-kit.googlecode.com/files/fmk_099.tar.gz)) so that you can analyze the firmware.
<!--break-->

------  

# Introduction

![intro](/images/2015-Holiday-Hack/2015-holiday-hack-part2-intro.png)

The specific questions we need answered involve the analyzing a firmware image that we downloaded. After speaking with Jessica we can download the firmware from [here](﻿﻿https://www.holidayhackchallenge.com/2015/giyh-firmware-dump.bin). After downloading it i checked the file type with ```binwalk firmware-dump.bin```, and found out that it was a squashfs piece of firmware. 

# Extracting Firmware

I learned from Jeff that  "firmware files often consist of header records and binary code". I also found out that binwalk is a handy tool to use when analyzing firmware. Running binwalk on the downloaded firmware yields a little information: a PEM cert, a 32-bit  Linux Standard Base[^1] object, and a squashfs filesystem.

![binwalk hint](/images/2015-Holiday-Hack/2015-holiday-hack-part2-binwalk.png)

<figure class="highlight"><pre><code class="language-text" data-lang="text">DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PEM certificate
1809          0x711           ELF 32-bit LSB shared object, ARM, version 1 (SYSV)
168803        0x29363         Squashfs filesystem, little endian, version 4.0, compression:gzip, size: 17376149 bytes,  4866 inodes, blocksize: 131072 bytes, created: Tue Dec  8 13:47:32 2015</code></pre></figure>

![squashfs hint](/images/2015-Holiday-Hack/2015-holiday-hack-part2-squashfs.png)

Now let's extract all the files with ```﻿binwalk -D "elf 32-bit lsb sbject":.so giyh-firmware-dump.bin -e```. Next we need to unsquash the filesystem that was extracted with ```/opt/fmk/./unsquashfs_all.sh 29363.squashfs```

![extract squash](/images/2015-Holiday-Hack/2015-holiday-hack-part2-extract.png)

# Solution

Now for our searching. Our first clue on the CPU lies in the file information for the shared object we extracted. So now we know the file cpu is based of a 32-bit ARM CPU. We can confirm that by checking any binary in the filesystem as well. For example, ```file /bin/cat``` displays:

```﻿bin/cat: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-armhf.so.1, stripped```

We can also find operating system information in a variety of places.[^2] Since the filesystem is not mounted the best place to search is the ```/etc``` directory. After looking through I find <strong>openwrt_version</strong> and <strong>openwrt_release</strong>. A quick ```cat openwrt*``` command read out the lines to provide me the operating system information.[^3]

<figure class="highlight"><pre><code class="language-text" data-lang="text">﻿DISTRIB_ID='OpenWrt'
DISTRIB_RELEASE='Bleeding Edge'
DISTRIB_REVISION='r47650'
DISTRIB_CODENAME='designated_driver'
DISTRIB_TARGET='realview/generic'
DISTRIB_DESCRIPTION='OpenWrt Designated Driver r47650'
DISTRIB_TAINTS=''
r47650</code></pre></figure>

Next, we need to find out information about the web interface, database, and find the plaintext password stored in the database. The ```/opt``` directory usually holds information for binaries installed on a system. Which shows us that mongodb is installed on the gnome device. Inside the mongodb directory there are 4 files and 2 directories. Running strings on them should display any plaintext pieces of info. Fortunately, the first attempt on gnome.0 yielded the plaintext username and password.

<figure class="highlight"><pre><code class="language-text" data-lang="text">--snip--
﻿gnome.users
username
user
password
user
user_level
username
admin
password
SittingOnAShelf
-snip-</code></pre></figure>

Last, we need to findout the web framework. Web stuff is generally held in the ```/www``` folder. Searching through a few directories yields a couple clues: index.js, *.jade, app.js, and node_modules. After a few, google searches we find that the web framework is Node.js (and Jessica actually confirms this as well).

With that [return to the Dosis neighborhood](https://quest.holidayhackchallenge.com/), find Jessica, and  confirm our password to move on to the next challenge!

![solution](/images/2015-Holiday-Hack/2015-holiday-hack-part2-solved.png)

# Analysis

3. What operating system an CPU type are used in the Gnome? What type of web framework is the Gnome web interface built in?  
  * The Gnome operating system is the "Bleeding Edge Release" for OpenWrt r47650 and the CPU type is 32-bit ARM.
  * The web framework is Node.js
4. What kind of a database engine is used to support the Gnome web interface? What is the plaintext password stored in the Gnome database?  
  * The database engine is ```Mongo DB```.
  * The plaintext password is ```SittingOnAShelf```.

------

------

# Endnotes

[^1]: A Linux Standard Base (LSB) binary is used when a developer wants to create an application that works across more than one Linux distribution. It includes a standard set of APIs a given distribution of Linux must support in order to make LSB binaries portable across Linux Operations Systems . [Click here to learn more about LSB](http://www.linuxfoundation.org/collaborate/workgroups/lsb)
[^2]: Linux has several ways to find out information about the host operating system. Specifically, ```uname``` is a handy command used to print system information. Type ```man uname``` to find out more about uname. We can also typically find it under the ```/etc``` folder under a file with <strong>"release"</strong> or <strong>"version"</strong>.
[^3]: OpenWrt is a GNU/Linux firmware distribution that is typically used for wireless routers. Learn more about OpenWrt [here](https://wiki.openwrt.org/doc/howto/user.beginner).