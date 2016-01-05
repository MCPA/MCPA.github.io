---
layout: post
title: "2015 SANS Holiday Hack Part 1"
category: web pcap Holiday Hack
author: "Leander"
---
<strong>Description:</strong> The first part of the holiday hack required you to log into the [Holiday Hack Quest Site](https://quest.holidayhackchallenge.com/). 

![login](/images/2015-Holiday-Hack/2015-holiday-hack-part1-login.png)

After logging into the game server find Josh Dosis and download the packet capture. 
Based on your analysis you should be able to see which commands are sent across the Gnome command-and-control channel and extract an image. For this challenge I used wireshark to analyze the traffic and a scapy python script to extract the information I needed from the packet capture.
<!--break-->

![find-josh-dosis](/images/2015-Holiday-Hack/2015-holiday-hack-part1-find-josh.png)

------  

# Introduction

![intro](/images/2015-Holiday-Hack/2015-holiday-hack-part1-intro.png)

First things first, let's review the questions. The specific questions we need involve the C2 channel and extracting an image from the pcap. I wasn't sure where to start first so my initial thought it to load the pcap into wireshark and profile the packet capture. The first thing I noticed was that the packet capture contained a relatively small number (1,568) wireless traffic and DNS packets. Next, I checked the packet capture conversation statistics. This showed me <strong><em>1x TCP</em></strong> converstaion and <strong><em>21x UDP conversations</em></strong>. 

# Packet Inspection

The unusual part was that the TCP conversations were DNS query and response packets - which are normally sent over UDP.[^1] The second unusual thing is that the responses have text responses which is generally an unused field. Additionally, the text appears to be base64 encoded for example packet #432 has ```﻿Tk9ORTo=``` as the text in the response. Which decodes to ```NONE:``` when using the command ```echo Tk9ORTo= | base64 -d```.

At this point I realized I had uncovered the way messages were being passed inside the packet capture. Next, I filtered the for the largest sized stream and selected a packet so that I could follow the UDP stream. It's a fairly long filter:

```(ip.addr eq 10.42.0.18 and ip.addr eq 52.2.229.189) and (udp.port eq 53 and udp.port eq 26214)```

![intro](/images/2015-Holiday-Hack/2015-holiday-hack-part1-evidence.png)

After I was able to view the conversations I knew I had found the right packets. The next step was to begin writing a script to extract all the data I needed. Otherwise it takes a really long time to copy and paste the values from each individual packet - and then subsequently base64 decode them. I used a interactive packet manipulation program called scapy, which can be directely used with Python.[^2] 

# Solution

The script I developed reads a packet capture using Scapy. Then it checks for a specific id code associated with the packet captures using the Scapy function called ```haslayer()```. Initially, I placed the entire set of packet IDs and DNS responses into a list and printed the entire base64 decoded list to see the contents. 

The C2 channel appears to use tags such as ```EXEC``` and ```FILE```. I took advantage of these C2 tags to find where the image started so that I could extract it. The specific indicator I found while analyzing the traffic was ```FILE:START_STATE,NAME=/root/Pictures/snapshot_CURRENT.jpg```.

After that I started extracting and writing the decoded file parts to disk by looking for the C2 tag ```FILE:``` and the C2 messages were printed to the screen. Through this method I was able to seperate two specific commands sent over this packet capture and I was also able to piece together the jpeg that was transferred. My Python script can be found at the bottom in the Endnotes.

<figure class="highlight"><pre><code class="language-text" data-lang="text">﻿Text:  NONE:   
Text:  NONE:   
Text:  EXEC:iwconfig  
Text:  EXEC:          Power Management:off  
Text:  EXEC:lo        no wireless extensions.  
Text:  EXEC:eth0      no wireless extensions.  
Text:  NONE:   
Text:  NONE: 
Text:  NONE: 
Text:  EXEC:cat /tmp/iwlistscan.txt</code></pre></figure>

![extracted image](/images/2015-Holiday-Hack/2015-holiday-hack-pcap-extract-IMG.png)

# Analysis

1. Which commands are sent across the Gnome’s command-and-control channel?  
  * Two commands are sent that I found. ```iwconfig``` and ```cat /tmp/iwlistscan.txt```  
  * Two C2 tags are used by the Command and Control server. ```EXEC:``` and ```FILE:```.  
2. What image appears in the photo the Gnome sent across the channel from the Dosis home?  
  * The image displays a bedroom, with a bunkbed, and elf on the shelf feet.  
  * The image also shows some text ```GnomeNET-NorthAmerica```  

------

------

# Endnotes

[^1]: DNS queries and responses are usually sent as UDP packets. The DNS message format is generally used to query and resolve a web address to an IP address instead of humans needing to remember unique IPs. [Click here to learn more DNS Queries and Responses](http://www.firewall.cx/networking-topics/protocols/domain-name-system-dns/160-protocols-dns-query.html)

[^2]: Scapy is a packet manipulation tool capable of decoding, encoding, and other types of networking packet crafting and analysis. One of the better benefits is that you can use it directely with Python without much effort - other than learning how to use Scapy. [Click here to learn more about Scapy](http://www.secdev.org/projects/scapy/)

### Python script used to extract image and view C2 channel:  

{% highlight python linenos %}

from scapy.all import *
from pwn import *
from colorama import *
    # Colorama constants are:
    #     Colors are Black, Red, Green, Yellow, Blue, Magenta, Cyan, White, and Reset
    #     Can be Fore or Back
    #     Can Style with Dim, Normal, Bright, Reset_All
    # Example: Fore.RED + Style.BRIGHT + "Hello World"

if __name__ == "__main__":
    if len(sys.argv) < 2 or len(sys.argv) > 2:
        print "usage: %s file.pcap" % (sys.argv[0],)
        sys.exit(1)

def test_decode(array):
    for x in range(1, 25):
        try:
            data = base64.standard_b64decode(array[x:])
            return data
        except:
	    continue

def main():
    filename = sys.argv[1] 
    pkts = rdpcap(filename)
    
    myList = []
    myErrors = []
    
    for p in pkts:
        if p.haslayer(DNS):
            #packet 242 begins the transfer of a large file
            if p.id == 242:
                try:
                    myList.append(base64.standard_b64decode(p.an.rdata))
                except:
                    myErrors.append(p.an.rdata)
            else:
                try:
                    myErrors.append(p.an.rdata)
                except:
                    continue

    newfile = open(filename+"-output", 'w')
    newfile.truncate()

    for i in myList:
    # Expect to see "FILE:" whenever a file is transfered, carve it out
        if (i[:5] == "FILE:"):
            newfile.write(i[5:].rstrip())
    # Prints lines that did not write based on C2 channel formatting
    #   else:
    #       print i

    newfile.close()

    print Fore.RED + Style.DIM + "\nPossible C2 Channel Text:\n" + Style.RESET_ALL
    for i in myErrors:
        print Fore.GREEN, "Text:", Fore.CYAN, test_decode(i), Style.RESET_ALL

    print Fore.MAGENTA + Style.BRIGHT + "\nCarved out the image successfully and stored the output in a file called %s\n" % (filename+"-output")

main()

{% endhighlight %}