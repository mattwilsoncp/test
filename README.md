This is my continuously updated (as of 24/04/2018, UK date format) beginner's guide for setting up the BeagleBone Blue with Mirko Denecke's port of ArduPilot (https://github.com/mirkix/ardupilotblue). This guide is based upon a similar one at Patrick Poirier's PocketPilot project (https://github.com/PocketPilot/PocketPilot). Many thanks to both these guys.

Only necessary steps are shown, excepting that I install Git for the sake of convenience. Information found elsewhere may indicate steps that are no longer necessary due to the software having been updated, or steps that only apply to other platforms (BBBMINI or PocketPilot, etc).

0) Before I begin, I want to stress that supplying adequate power to the BeageBone Blue is a must. Typically, we'll be attaching quite a few peripherals to it, and they'll not behave correctly without enough juice.

1) Go to https://rcn-ee.net/rootfs/bb.org/testing/ and select the directory named with the latest date. Then click on the stretch-console directory. You'll see a number of files here. Download the file named something like bone-debian-V.V-console-armhf-20YY-MM-DD-1gb.img.xz. This is what's known as the 'console' image. It's a very minimal image of Debian with only the bare essentials. An alternative is the 'IoT' image (IoT = Internet of Things) which comes with additional software and can make for a more comfortable experience if you are very new to Linux. It's available from the same site.

    At the time of writing, I'm using bone-debian-9.4-console-armhf-2018-04-22-1gb.img.xz.

2) Now you'll need to copy the image to a microSD card. Whether you are using Linux or Windows, I highly recommend a program called Etcher for this task (https://etcher.io/).

3) It should now be possible to boot up the BeagleBone Blue from the microSD card. It is beyond the scope of this document to detail ways of interacting with the BBBlue, but often it's accomplished by plugging in a Micro-USB cable and either using SSH (to 'debian@192.168.7.2', password 'temppwd') or establishing a serial link over a COM port (user 'debian', password 'temppwd)'. More information can be found here: https://beagleboard.org/blue.

4) Hopefully, you now find yourself logged into the debian user account and at the command prompt. The next task is to update and install some software using an available internet connection.

    If you are SSHing to 192.168.7.2, you can your share your computer's internet connection with the BBBlue very simply by typing (at the BBBlue's command prompt): `sudo /sbin/route add default gw 192.168.7.1`
    
    If you have established a serial link (in a terminal program like Minicom), the process is more difficult, especially as the console image does not include Connman (for WiFi). In the end, I chose to use a USB-to-Ethernet dongle I had lying around (http://accessories.ap.dell.com/sna/productdetail.aspx?c=sg&l=en&s=bsd&cs=sgbsd1&sku=470-ABNL) as I could plug this into the BBBlue and then connect it directly to my router. Note that I had to supply extra power to the BBBlue via its 2s LiPo connector for the dongle to work. The BBBlue enumerates the dongle as device 'usb2', and sets it up automatically (well, sometimes it's necessary to reboot in order for the command `ip link` to show that the device is up).
    
    Whichever way you go, you can verify you have internet by typing: `ping -c 3 www.google.com`

5) Update and install all required supporting software:

        sudo apt-get -y update
        sudo apt-get -y dist-upgrade
        sudo apt-get install -y cpufrequtils connman git
6) Update Git: `cd /opt/scripts && git pull`
7) Specify Ti real-time kernel 4_9: `sudo /opt/scripts/tools/update_kernel.sh --ti-rt-channel --lts-4_9`
8) Specify device tree binary to be used at startup: `sudo sed -i 's/#dtb=/dtb=am335x-boneblue.dtb/g' /boot/uEnv.txt`
9) Set clock frequency: `sudo sed -i 's/GOVERNOR="ondemand"/GOVERNOR="performance"/g' /etc/init.d/cpufrequtils`
10) Disable Bluetooth (optional): `sudo systemctl disable bb-wl18xx-bluetooth.service`
11) Maximize the microSD card's existing partition (which is /dev/mmcblk0p1): `sudo /opt/scripts/tools/grow_partition.sh`
12) Reboot now: `sudo reboot`
13) When you log back in, you'll see something like 'Linux beaglebone 4.9.88-ti-rt-r111 #1 SMP PREEMPT RT Sun Apr 22 10:34:09 UTC 2018 armv7l'. Good. Now it's time to set up Connman for WiFi. My method makes for easier automation in a script later on as it does not rely on you knowing the Connman hash beforehand. You will need to supply your SSID and your WiFi password. Do not use quotes:

        mkdir -p /var/lib/connman
        
        cat >/var/lib/connman/wifi.config
        [service_\$(connmanctl services | grep '<your SSID>' | grep -Po 'wifi_[^ ]+')]
        Type = wifi
        Security = wpa2
        Name = <your SSID>
        Passphrase = <your WiFi password>
        
        systemctl enable connman
14) 
