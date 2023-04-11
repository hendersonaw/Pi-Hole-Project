# Pi-Hole-Project

This project is to create a Pi-Hole that blocks adult content via DNS on your home network. It includes various blocklists and instructions on how to set everything up. 

## Hardware
The best hardware to use for this project is a [Raspberry Pi 4](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/), but they can be pretty hard to find right now due to a massive chip shortage. If you aren't able to get access to one, a good alternative to use is the [Orange Pi Zero 2](https://www.amazon.com/Orange-Pi-Allwinner-Open-Source-Android10/dp/B0B973214J). You'll also want to make sure you have a [microSD card](https://www.amazon.com/dp/B0B7NVMBPL) and an [Ethernet cable](https://www.amazon.com/dp/B00C2CBBAM). You can complete this project without an Ethernet cable, but it will make it a lot easier. 

## Installing and Setting Up Pi-Hole

### Adding the Blocklists via Adlists Feature

### Changing Default DNS Servers to Cloudflare (1.1.1.1)

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
Make sure to replace <your Healthchecks.io URL> with the URL listed on Healthchecks.io under the "How To Ping" section with the "uuid" button selected (not the "slug" button). 

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
