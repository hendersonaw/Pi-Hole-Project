# Pi-Hole-Project

This project is to create a Pi-Hole that blocks adult content via DNS on your home network. It includes various blocklists and instructions on how to set everything up. 

## Table of Contents
1. [Hardware](#hardware)
2. [Installing and Setting Up Pi-Hole](#installing-and-setting-up-pi-hole)
    - [Networking...](#networking-)
    - [Adding the Blocklists via Adlists Feature](#adding-the-blocklists-via-adlists-feature)
    - [Force SafeSearch on Search Engines and YouTube](#force-safesearch-on-search-engines-and-youtube)
    - [Changing Default DNS Servers to Cloudflare (1.1.1.1)](#changing-default-dns-servers-to-cloudflare-1111)
3. [Creating Cronjobs for Heartbeat via Healthchecks.io](#creating-cronjobs-for-heartbeat-via-healthchecksio)
    - [Healthchecks.io Setup](#healthchecksio-setup)
    - [Pi-Hole Setup](#pi-hole-setup)
4. [Conclusion](#conclusion)

## Hardware
The best hardware to use for this project is a [Raspberry Pi 4](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/), but they can be pretty hard to find right now due to a massive chip shortage. If you aren't able to get access to one, a good alternative to use is the [Orange Pi Zero 2](https://www.amazon.com/Orange-Pi-Allwinner-Open-Source-Android10/dp/B0B973214J). You'll also want to make sure you have a [microSD card](https://www.amazon.com/dp/B0B7NVMBPL) and an [Ethernet cable](https://www.amazon.com/dp/B00C2CBBAM). You can complete this project without an Ethernet cable, but it will make it a lot easier. The rest of the documentation will assume you are using an Ethernet cable. 

## Installing and Setting Up Pi-Hole
Here are the general steps you need to follow to install Pi-Hole: 
1. Install an Operating System on your microSD card via [Balena Etcher](https://www.balena.io/etcher). 
2. Once installed, plug it into your Pi and turn it on. 
3. Once logged into the command prompt, run the following command in a command line to install Pi-Hole: 
```
curl -sSL https://install.pi-hole.net | bash
```
More information on how to install Pi-Hole can be found on the [Pi-Hole website](https://pi-hole.net/). 

### Networking... :(
Because networking can be so different for everyone, I can't provide an exact step-by-step list of instructions for you. However, I'll do my best to provide the information for you to learn how to find the information needed for your specific infrastructure. For the following instructions, I will mostly assume that you are working on a Windows PC, but will try to assist any Linux users along the way (no promises...). 

Connect your Pi to your PC via Ethernet cable. On your PC, open a command prompt (on Windows, press the Windows key and type "cmd"), then type "ipconfig" (Linux users, use "ifconfig") in the command prompt to find your device's IP address. Find the section that says "Wireless LAN adapter Wi-Fi" and note the IP address for the "Default Gateway". This should be something similar to 192.168.1.1 or 10.0.0.1. Open a web browser and type that IP address into it. 

This IP address is for your router and will look different based on which router you have. Here are the general instructions for what you're looking for: 
1. Log into your router with your admin credentials (if they are similar to Username - "admin" Password - "admin", you should change this immediately). 
2. Find the list of devices connected to your network and look for your Pi's name (e.g., "raspberrypi", "orangepizero2", etc.). 
3. Take note of the IP address assigned to your Pi. 

Now, type your Pi's IP address into the browser, followed by "/admin" (e.g., "192.168.1.231/admin"). This should bring you to the Pi-Hole admin web login. If it does, congrats! The hardest part of the project is over for you! If it didn't, make sure that you successfully installed Pi-Hole on your device. 

Log into the web portal using the password given to you at the end of Pi-Hole's installation. 

### Adding the Blocklists via Adlists Feature
To add massive amounts of domains to the blocklist, click on the Adlists button in the navigation menu on the left side. Then add the following lists to block NSFW domains: 
```
https://nsfw.oisd.nl/
https://raw.githubusercontent.com/chadmayfield/my-pihole-blocklists/master/lists/pi_blocklist_porn_all.list
```
For additional domains, I have created a few more lists to include grey sites that aren't blocked by either of the above lists: 
```
https://raw.githubusercontent.com/hendersonaw/Pi-Hole-Project/main/lists/Alternate%20Search%20Engine%20List
https://raw.githubusercontent.com/hendersonaw/Pi-Hole-Project/main/lists/Libreddit%20Public%20Sites
https://raw.githubusercontent.com/hendersonaw/Pi-Hole-Project/main/lists/Picture%20Sharing%20Sites
https://raw.githubusercontent.com/hendersonaw/Pi-Hole-Project/main/lists/Proxy%20Sites
https://raw.githubusercontent.com/hendersonaw/Pi-Hole-Project/main/lists/URL%20Screenshot%20Sites
```

Once you have added the lists you want to block, expand the "Tools" section in the left navigation bar and click "Update Gravity". Click on the big blue bar that says "Update" and wait for it to complete. You'll know it's done when you see a big green bar that says "Success!"

### Force SafeSearch on Search Engines and YouTube
If you want to enforce SafeSearch on Search Engines and restrictions on YouTube on all devices on your network, expand the "Local DNS" section and click on "DNS Records". Then add the following local DNS domains: 
```
Domain                          IP Address
----------------------------    --------------
forcesafesearch.google.com      216.239.38.120
restrict.youtube.com            216.239.38.120
restrictmoderate.youtube.com    216.239.38.119
safe.duckduckgo.com              52.250.41.2
strict.bing.com                 204.79.197.220
```

Then in the left navigation menu, click on "CNAME Records" and add the following local CNAME records: 
```
Domain                          Target Domain
------------------------        -----------------------------
duckduckgo.com                  safe.duckduckgo.com
m.youtube.com                   restrictmoderate.youtube.com
www.bing.com                    strict.bing.com
www.duckduckgo.com              safe.duckduckgo.com
www.google.com                  forcesafesearch.google.com
www.youtube-nocookie.com        restrictmoderate.youtube.com
www.youtube.com                 restrictmoderate.youtube.com
youtube.googleapis.com          restrictmoderate.youtube.com
youtubei.googleapis.com         restrictmoderate.youtube.com
```
There are two levels of restricting Youtube, "Restrict Moderate" and "Restrict". If you find that the moderate YouTube restriction is not enough for you, you can make it even stricter by changing "restrictmoderate.youtube.com" to "restrict.youtube.com" in the Target Domain. 

### Changing Default DNS Servers to Cloudflare (1.1.1.1)
Now we'll change our default DNS servers from Google to Cloudflare's Family DNS server. In the navigation menu, click on "Settings". At the top of the screen, click on the "DNS" tab. Unselect all checkboxes on the "Upstream DNS Servers" on the left side, then check all 4 Custom "Upstream DNS Servers" on the right side. Add the following IP addresses in the Custom IP fields: 
```
Custom 1 (IPv4) - 1.1.1.3
Custom 2 (IPv4) - 1.0.0.3
Custom 3 (IPv6) - 2606:4700:4700::1113
Custom 4 (IPv6) - 2606:4700:4700::1003
```
This will provide additional security and use Cloudflare's blocklists in combination with your Adlists. 

## Creating Cronjobs for Heartbeat via Healthchecks.io

If you want to set up a heartbeat for your Pi-Hole to alert when it goes down, you can use [Healthchecks.io](https://healthchecks.io/). Using a heartbeat will instruct the Pi-Hole to reach out to the website periodically to make sure that the Pi-Hole is still running. If the Pi-Hole fails to ping the website due to being unplugged/turned off, Healthchecks.io will send an email to whoever you'd like saying your blocker is down. This adds an additional layer of accountability so you won't be tempted to unplug the device. 

### Healthchecks.io Setup
To set this up, create an account at [Healthchecks.io](https://healthchecks.io/) and log in. Then create a new project labeled "Pi-Hole Blocker" (or any other name you'd like) and click the "Add Check" button. Give it a human-friendly name and select how often you want the website to expect a ping from the Pi-Hole (I recommend a Period of 5 minutes with a Grace Time of 5 minutes), then click "Save". 

### Pi-Hole Setup
On your Pi, run the following command to create a script labeled "heartbeat.sh" in your home directory: 
```
nano ~/heartbeat.sh
```
Then add the following code inside that file: 
```
#!/bin/bash

# using curl (10 second timeout, retry up to 5 times):
curl -m 10 --retry 5 <your Healthchecks.io URL>
```
Make sure to replace _\<your Healthchecks.io URL\>_ with the URL listed on Healthchecks.io under the "How To Ping" section with the "uuid" button selected (not the "slug" button). 

Press CTRL+S to save the file, then CTRL+X to exit. Then in the command line of your Pi, run the following command: 
```
crontab -e
```
If it is the first time you are running the crontab command, you will be required to pick an editor as shown in the next piece of code: 
```
$ crontab -e
no crontab for user - using an empty one

Select an editor. To change later, run 'select-editor'.
1. /bin/nano <---- easiest
2. /usr/bin/vim.basic
3. /bin/ed

Choose 1-3 [1]:
```
Select which editor to use (if you're not sure, pick 1). At the bottom of the file, write your crontab job with the command you want to run. If you followed my recommendation on the Healthchecks.io setup and set the Period to 5 minutes, use the following code: 
```
*/5 * * * * bash ~/heartbeat.sh
```

For more information on scheduling a cronjob, see this [link](https://crontab.guru/). 

## Conclusion
You should be all set! I hope you found this project to be useful. If there's any details I left out, feel free to reach out and I'll update the instructions accordingly. Thanks for reading!
