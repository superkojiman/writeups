
About two months ago I decided I was tired of my Blackberry and wanted something a lot more fun, so I started hunting around for an Android phone. I purchased the [Samsung Galaxy Spica](http://www.samsung.com/ie/consumer/mobile-phones/mobile-phones/touch-screen/GT-I5700UWAXEU/index.idx?pagetype=prd_detail) off of eBay for about USD $300. The device came with Android 1.5 (Cupcake). It wasn't long before I started looking for ways to upgrade it to 2.1 (Eclair). 

<!--more-->

Samsung had announced that it was going to release the official updates "soon". Not soon enough for me though. I found numerous guides that detailed unofficial ways to flash Eclair onto the device. Each one was slightly different, using different options, procedures, and so on. After much reading, a long nap, and a shot of gin, I decided to give it a go.

I managed to get this working on my first try. Then I tried it again several times over the course of the week and wrote these steps down so that I could record them here. What follows is a combination of steps from various guides put together that worked for me, and hopefully they'll work for you too.

### Disclaimer

>This article is provided AS IS and I make no guarantees that it will work for you. If you choose to follow the steps outlined in this article, you agree that I will not be held responsible for any direct or indirect damages of any sort that may be caused by following the instructions on this article. If you don't want to take risks, wait for the official Samsung update or take your device to a Samsung service center to be upgraded.

### Onwards

First some information about my phone and the PC I used to flash the new firmware:

* Original firmware: I5700XXIJ5
* PC: Windows Vista 32-bit

I've heard that 64-bit versions of Windows have problems recognizing the phone's drivers, so see if you can find a 32-bit version of Windows. I used Windows Vista, but Windows XP should work just as well. 

You'll need a few things before we begin:

* New PC Studio 1.3.0.IJ1 (abbreviated to NPS). This should have come with the phone. You can also download it from [Samsung's website](http://www.samsungmobile.co.uk/support/softwaremanuals/software.do?phone_model=GT-I5700&sw_type=SW).

* Odin 4.03. This is the application that's used to flash the firmware to the phone. Get it [here](http://www.techorganic.com/software/spica_21/Odin4_03-Spica_ops.zip).

* I5700EXXJCE.zip Get it [here](http://www.techorganic.com/software/spica_21/I570EXXJCE.zip). 

* spica_jc3.rar. Get it [here](http://www.techorganic.com/software/spica_21/spica_jc3.rar).

* dvr5700.zip. These are the drivers for the phone. I never had to use them since the drivers were installed when I installed NPS, but just in case you can get them [here](http://www.techorganic.com/software/spica_21/dvr5700.zip).

* WinRAR. You'll need this to extract the spica_jc3.rar file. Get it [here](http://www.rarlab.com/). If you already have a tool that uncompresses rar files, you can use that instead of WinRAR.

If you're paranoid like me, you'll probably want to check the MD5 hashes for the firmware files and Odin. For Windows, you can use the [md5sums](http://www.pc-tools.net/win32/md5sums/) tool, or any other application that can generate MD5 hashes. These are the hashes on my end:

```
MD5 (Odin4_03-Spica_ops.zip) = 9f40bc20690508b625ec2798ff4d0792
MD5 (I570EXXJCE.zip) = 1cd87530f483fcfa95ef6b2bf08ff6f3
MD5 (spica_jc3.rar) = 5af65f4bbc66402044deca9e5d46f6ec
MD5 (dvr5700.zip) = ccecd4d798dafc7396b2fb67652ff423
```

Alright, let's begin.


### Preparation

1. Make sure the phone is fully charged. If you're going to be working on a laptop without a power supply, make sure the laptop is fully charged as well. You don't want the phone or laptop to turn off halfway through the flashing.

1. Backup whatever you need to on your phone, such as contacts, SMS messages, applications, etc. If you don't know how to do this, search on Google there are applications available in the Market that can save this data in the SD card.

1. Install New PC Studio 1.3.0.IJ1. This is relatively straight forward. After you install it, connect your phone to the PC with the USB cable that came with it. NPS should recognize it and load up the phone. If it does, you've installed it correctly. Unplug the phone and reboot the PC.

1. Install WinRAR if you don't have a tool that can unpack rar files. Again, this is relatively straight forward.

1. Create a new folder on your Desktop called Work. You can call it anything really, but this is where we'll extract the contents of the zip and rar files. For the purpose of this tutorial, I'll refer to this folder as Work.

1. Extract the contents of I570EXXJCE.zip into the Work folder.

1. Extract the contents of Odin_4_03-Spica_ops.zip into the Work folder.

1. Extract the contents of spica_jc3.rar into the Work folder.

1. At this point you'll have a bunch of files in the Work folder. Check to make sure that these are present (the rest can be deleted):

    * I570EXXJCE.tar
    * Odin Multi Downloader 4.03
    * spica_jc3.ops
    * SS_DL.dll


### Preliminary Test

The purpose of this step is to see if Odin will even recognize your phone.

1. Shutdown your phone and remove the SIM card, SD card, and battery.

1. Leave the battery out for a minute and then re-insert it.

1. Put the phone into download mode by pressing volume down + power button + camera. You will see a picture of a blue disk and the text Downloading… as well as a message in a blue rectangle that says DO NOT TURN OFF TARGET!!!

1. Plug the phone into the PC and launch Device Manager. If you don't know how to launch Device Manager, click here. Under Modems check for Samsung Mobile Modem and under USB check for Samsung USB Device. If either one of these are missing, unplug the phone, reinstall NPS and try it again. Do not proceed until you see the Samsung modem and USB device!

1. Shut off any anti-virus scanners.

1. Shut off NPS completely. Check the Windows task bar, if it's running there, right click on the icon and turn it off.

1. Launch Odin. You should see the COM Port Mapping turn into a yellow box and display the COM port. The Message box should say <1> Added!!! and <1>Detected!!!. If you do not see it, reboot the phone and the PC and try it again. If that doesn't work, re-install NPS and try again. 

**Note:** You may get a warning popup in Korean that says something like "IMAGE PATH ...". This is normal, just press OK to continue.
1. If it worked, unplug the phone.

At this point, we can assume that the drivers are properly installed and Odin recognizes the phone. There's a good chance that flashing will be smooth from here on, although you may still not be quite so lucky.

### Flashing

This section will deal with flashing the phone, thereby deleting all the data on the phone. Make sure you've backed up everything that you need. 

We need to reset the phone to factory settings. This will delete **everything** on the phone. Dial the following: 2767*3855#. The phone will return to factory settings immediately without prompting you with an "OK/Cancel" dialog.

Shut off the phone and remove the battery for about a minute.

Put the phone into download mode by pressing volume down + power button + camera. You will see a picture of a blue disk and the text Downloading… as well as a message in a blue rectangle that says DO NOT TURN OFF TARGET!!!

Plug it into the computer with the USB cable.

Start Odin. As before you should see the COM Port Mapping box turn yellow and display the COM port. Again, just ignore the Korean warning messages that may pop up.

Set the following options in Odin:

* OPS field: spica_jc3.ops
* One Package field: I570XXJCE.tar
* One package: check
* Reboot: check
* Protect OPS: check
* Debug Only: **UNCHECK!!!**
* Reset time: 30 seconds

Leave the fields for Debug Only, Boot, Phone, PDA, and CSC blank.

Double check everything. When you're sure you're ready, click the Start button. Odin should display a green progress bar, and the phone should display a blue progress bar. In less than 20 seconds it should start flashing. Here's the procedure as I've seen it. 

   First the Odin message box will start to print out the following and you'll see the green progress bar slowly filling up:

```
<1> zImage download..
<1> 1/5 Finished.
<1> datafs.rfs download..
<1> 2/5 Finished.
<1> factoryfs.rfs download..
```
Next the blue bar on the phone's screen will start to fill up and you'll see this printed in Odin's message box:

```
<1> 3/5 Finished.
<1> cache.rfs download..
<1> 4/5 Finished.
```
The blue bar on the phone's screen will fill up and the phone will reboot. Odin's message box should display:

```
<1> modem.bin download..
<1> 5/5 Finished.
<1> reset pda..
<0> Started Timer
<1> Close serial port and wait until rebooting.
```

The phone display the same picture on the screen as you did when you did a hard reset. Be patient and wait for it to reboot. Eventually you'll see the status box on Odin turn blue with the words PASS and the message box will display:

```
<1> PASS!!!
<0> Destroy instant..
<0> Killed timer
<1> Detected!!!
<1> Disconnected
```
At this point, your phone has been successfully flashed. Total time should be approximately **2** minutes.

Click any of the buttons on the phone and you should now be on the lock screen... in French. Unplug the phone.

The firmware is in French, so your phone will be in French now. To fix this, you'll need to change the locale setting on the phone by pressing the Menu button > Parametres > Parametres de langue > Langue et region and select your language.

Ok, so what can go wrong during the process? I've tried flashing the phone several times, and I ran into a couple of hiccups whereby over 5 minutes pass and the progress bar on Odin or the phone don't move. When this happens I unplug the phone, remove the battery for a minute, and put it back into download mode, and flash it again. It might also help to restart the computer while waiting for the minute to pass. As long as your phone can enter download mode, there's the hope of flashing some firmware into it and saving it. 

Good luck!
