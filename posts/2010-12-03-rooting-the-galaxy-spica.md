
Finally got around to doing this. I'm writing this guide as a future reference in case I need to do this again. First you need to upgrade to Android 2.1, so follow the instructions in: [Samsung Galaxy Spica: Upgrading Android 1.5 to 2.1](/2010/06/28/samsung-galaxy-spica-upgrading-android). 

<!--more-->

### Disclaimer
>This article is provided AS IS and I make no guarantees that it will work for you. If you choose to follow the steps outlined in this article, you agree that I will not be held responsible for any direct or indirect damages of any sort that may be caused by following the instructions on this article. If you don't want to take risks, wait for the official Samsung update or take your device to a Samsung service center to be upgraded.

### Required Files
* spica_jc3.ops. Same one you used when you upgraded to 2.1.
* i5700_LK2-08_PDA.7z. Needed to root the phone. Grab it from [here](http://www.techorganic.com/software/spica_21/i5700_LK2-08_PDA.7z). The MD5 checksum is 305093241f8cfb70343461556629aef6. This 7-zip archive contains i5700_LK2-08_PDA.tar. Extract it and put it in the same location as spica_jc3.

### Rooting
1. Turn your phone off, remove the battery, SIM card, and SD card. Wait for a minute and then re-insert the battery

1. Boot into download mode (Volume down + Camera + Power on)

1. Turn off any anti-virus you might have running on your PC.

1. Plug the phone into the PC and make sure Odin recognizes the COM port.

1. Leave all options unchecked and fields blank except for the following:
    * PDA field: i5700_LK2-08_PDA.tar
    * OPS field: spica_jc3.ops
    * Reboot option: checked
    * Protect OPS option: checked
    * Reset time:: 30 seconds
1. Click on Start and wait until the phone stops rebooting and Odin displays the blue box with the words PASS in it. The procedure should take under 60 seconds.

1. Wait for the phone to finish rebooting before unplugging it. Your phone is now rooted.

### Recovery

Your phone now has the ability to boot into recovery mode. To do this switch off the phone, then hold down the volume down + answer call + end call buttons at the same time. Your phone will boot into recovery. In the recovery menu you can navigate using the directional pad on the phone and using the OK button to select a menu. The most useful feature here is the ability to do a full image backup of your phone so you can quickly restore it in case you need to.
