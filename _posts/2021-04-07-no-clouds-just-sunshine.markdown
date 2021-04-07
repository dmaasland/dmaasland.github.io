---
title: "No clouds, just sunshine. Disconnecting Somfy Connexoon from the cloud."
---

## Introduction
I've bought some new shutters recently and got the Connexoon with my purchase for free. After connecting the device I was disappoinited to learn that it requires both an internet connection and an account to function properly. So for the last few days I've been working on a way to do away with that!

![96767ece-3eb9-40d4-9615-ae1a19d1c65a](/assets/img/somfy/96767ece-3eb9-40d4-9615-ae1a19d1c65a.jpg) 

## Goal
The goal for this project is to be able to control my Connexoon devices through Home Assistant without the need for the device to be connected to a cloud service and without the use of an account.

## First steps
First thing I did was a little network discovery. Unfortunately, port scanning the device with NMAP resulted in 0 open ports. I started a packet capture on my router and saw the device connecting to several domains, but all of them used TLS and even required client certificate authentication.

So, next thing I did was open it up and take a look at the board. The device has no screws and can be pryed open with a flathead screwdriver or another thin piece of metal. The board itself isn't anything special. One side has an ARM9G25 and some RAM:

![995baae1-1476-44dc-aa3d-dc1ffcbcc7fe](/assets/img/somfy/995baae1-1476-44dc-aa3d-dc1ffcbcc7fe.jpg) 

while the other side holds the NAND:

![19e2efec-033b-466e-b587-691251f94d2c](/assets/img/somfy/19e2efec-033b-466e-b587-691251f94d2c.jpg) 

It also has the word "Overkiz" on the board, as well as "Minibox 868". Throwing those into Google yielded two brochures about the "Overkiz Minibox":

- [https://www.overkiz.com/templates/front/pdf/fiche_minibox_web.pdf](https://www.overkiz.com/templates/front/pdf/fiche_minibox_web.pdf)
- [https://www.overkiz.com/wp-content/uploads/2018/12/FICHE_PRODUIT_MINIBOX_2019-EN.pdf](https://www.overkiz.com/wp-content/uploads/2018/12/FICHE_PRODUIT_MINIBOX_2019-EN.pdf)

Turns out the same product is sold under different names:

- Connexoon by Somfy
- Cozytouch by Atlantic
- I-Vista by Cotherm
- HI-KUMO by Hitachi

Also, one of the brochures mentions the device can also operate in "offline" mode. Interesting..

## Dumping the firmware
This gave me some leads to go on. I discovered that multiple people had already done some work on the thing:

- [https://github.com/Aldohrs/tahoma-jailbreak](https://github.com/Aldohrs/tahoma-jailbreak) (This was for the Tahoma, but it's similar)
- [https://www.lafois.com/2020/12/20/rooting-the-cozytouch-aka-kizbox-mini-part-5/](https://www.lafois.com/2020/12/20/rooting-the-cozytouch-aka-kizbox-mini-part-5/)

Though both guides weren't exactly tutorials for what I was trying to do, they did give me some useful pointers. The most important pointer came from the "tahoma-jailbreak" github repository. Here, he found a way to force the ARM SOC into some sort of recovery mode. Long story short, if you provide 3.3v to pin 9 of the flash chip during boot, it'll go into this mode.

Turns out this is also true for the Connexoon. I spent some time looking around the board with a multimeter and found both a 3.3v source as well as a test pad connected directly to pin 9 on the NAND chip:

![Naamloos](/assets/img/somfy/Naamloos.png) 

All you have to do is bridge these points while you plug in the USB power and then let go. Obviously, plug the other end into your computer. I used a paperclip which worked very well. If you've done it correctly the power LED will not turn on and you'll have a new device connected to your machine:

![6b1429cd-699a-4a31-b7dd-715ec3520533](/assets/img/somfy/6b1429cd-699a-4a31-b7dd-715ec3520533.jpg)

Now download the following software: [https://www.microchip.com/DevelopmentTools/ProductDetails/PartNO/SAM-BA%20In-system%20Programmer](https://www.microchip.com/DevelopmentTools/ProductDetails/PartNO/SAM-BA%20In-system%20Programmer)

and run these commands:

```shell
./sam-ba -p usb -b sam9xx5-ek -a lowlevel
./sam-ba -p usb -b sam9xx5-ek -a extram
./sam-ba -p serial -d sam9xx5 -a nandflash:1:8:0xc0902405 -c read:bootstrap.bin:0x000000:0x20000 -c read:ubi-volume.bin:0x20000
```

This will write the boatloader and the actual firmware to seperate files.

## Getting access
This is where things started to get messy. Although there were some scripts available to do certain things with the firmware, they didn't work out of the box and some instructions were missing. If you want I can go into details, but globally what you have to to to enable SSH is:

- Mount the root filesystem
- Set a password hash for the **`root`** user
- Create a symlink for dropbear in **`etc/rc5.d`** by running ```ln -s ../init.d/dropbear S06dropbear```
- Re-pack the firmware and write it

For writing, you use the command:
```shell
/sam-ba -p usb -b sam9xx5-ek -a nandflash:::0xc0902405 -c erase:0x20000 -c write:ubi-volume.bin:0x20000 -c verify:ubi-volume.bin:0x20000
```

Then, reboot the device. If all went well you should now be able to SSH into the device:

```shell
user@ubuntu2004:~/projects/somfy/work$ ssh root@192.168.150.7
root@192.168.150.7's password: 
at91-kizboxmini version 2021.1.4-12
built on 20210315:204444
debug:   false
release: production
user:    jenkins
0000-1111-2222:~$ 
```

(hostname censored, as that is your device PIN)

## Enabling the local api
As it turns out, enabling the local API is rather simple. Just four steps:

1. Remount the filesystem as rw: ```mount -o remount,rw /```
2. Remove the SSL config: ```rm /etc/lighttpd.d/ssl.conf```
3. Start the webserver: ```/etc/init.d/webserver start```
4. Start the local API: ```/etc/init.d/local start```

You should now be able to acces the local API with a browser:

![api](/assets/img/somfy/api.png) 

Now, this is where I got stuck for a while. The only two things you can visit without authentication are:

- /enduser-mobile-web/1/enduserAPI/apiVersion
- /enduser-mobile-web/1/enduserAPI/register/random_username

The first URL simply prints the version and the second one will always give the following message:

```json
{"error":"Missing authorization token.","errorCode":"RESOURCE_ACCESS_DENIED"}
```

No matter what I did, I could not find a way around this. I've pressed every combination of buttons on the device I could think of, I've spent hours searching through the filesystem, nothing. So I tried creating a user locally. Turns out you can generate a "token" on the device by running:

```shell
XXXX-XXXX-XXXX:~$ dbus-send --print-reply --system --type=method_call --dest=com.overkiz.utilities.local /com/overkiz/utilities/local/tokensService com.overkiz.utilities.local.tokensService.createWithRandomKey string:"test" string:"local" int32:2147483648
method return time=1615851235.721217 sender=:1.4 -> destination=:1.11 serial=59 reply_serial=2
   string "G5HNDlxlKlQUprLRlpqn"
XXXX-XXXX-XXXX:~$ 
```

There's three arguments you can give it:

1. A label
2. A scope
3. Expiration in seconds

What you get in return is a token you can use in all requests to the API by placing it in the **`X-AUTH-TOKEN`** header of your request:

![api_2](/assets/img/somfy/api_2.png) 

And now you can locally interact with the device :)

## Now what?
Well, this is as far as I've gotten. Not because I'm stuck, but because I don't have any devices to connect yet. I thought this was a good time to post this blog, just to see if:

- This project is interesting to anyone and if I should contintue to make it production ready
- If there is anyone who can give me some tips and / or pointes about the "pairing mode"
- There is anyone who can offer me some help in general.

I noticed that the API is pretty similar to the cloud version, but not all features are implemented. My endgame would be to make the current Home Assistant Tahoma integration compatible, but I have no clue on how to do that. I'm familier with python, just nog with HA integrations.

Anyway, that's it. I'm open to your thoughts and / or suggestions! To be continued :).

