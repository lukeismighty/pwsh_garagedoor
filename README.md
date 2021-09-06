# pwsh_garagedoor
RaspberryPi powered Powershell setup and script for opening and closing a garage door (a relay)

To start, get yourself a Raspberry Pi that can run Powershell (3 and up it seems). I made the setup here on a Raspberry Pi 3 Model B Rev 12.

Download the Raspberry Pi Imager from https://www.raspberrypi.org/software/. 

![image](https://user-images.githubusercontent.com/2230723/132256397-9346c601-46aa-43ec-89b8-bd6e70d9814a.png)

Install that and run it. When asked for the options, click Choose OS.

![image](https://user-images.githubusercontent.com/2230723/132256439-4ba7ec37-9613-4a79-b95a-0908b10a82d5.png)

Click on Raspbberry Pi OS (other).

![image](https://user-images.githubusercontent.com/2230723/132256466-b08ef8e9-dcb2-47d8-8743-a516558bf44b.png)

Select the Lite OS.

![image](https://user-images.githubusercontent.com/2230723/132256476-7dd7546f-87b5-44ee-b42f-51faacacaf5e.png)

Click on your Storage you'd like to put the OS onto for the Pi and select it from the list.

![image](https://user-images.githubusercontent.com/2230723/132256525-8a386746-5456-4f55-abb8-bbbcdaa6e979.png)

After that, click Write. When its all done, close the program. Pull the SD card from your computer and put it right back in. We need to add some stuff to it.

Download the three files in the dependencies folder and place them on the boot partition that mounts on your computer. This is writeable from Windows, Mac, and Linux.
The files are ssh, wpa_supplicant.conf, and getTemp.sh.  
The ssh file enables ssh at boot on the Raspberry Pi. Since this is a headless install (well, mine is), we want that on when we turn it on.  
Open the wpa_supplicant.conf file and change the name of your network to your Wi-Fi network name (case sensitive!) and put the Wi-Fi password in as well. You can see the marked lines where these need to be. Don't erase the quotation marks though, they are needed.  
The getTemp.sh file is a nice little script that lets you know the temperature of your Pi while its running. If you install this in a place that gets hot, you can run this to see how hot it is. This'll give you an idea if you are baking the thing and killing it with heat.  

Save your files, if you opened more than just the wpa_supplicant.conf, eject your SD card, and put the SD card into the Pi.

Take you wires you have for your relay or relays, the sensor for the door, and hook them up to these pins:

![image](https://user-images.githubusercontent.com/2230723/132257265-e086fa49-b0a8-408c-97f9-d1a9d3011d0f.png)

Pin 4 is your power for your relay  
Pin 6 is your ground for your relay  
Pin 7 is the "positive" side of your sensor (in this case, the door sensor (a reed switch))  
Pin 8 is the "ground" side of your sensor (in this case, the door sensor (a reed switch))  
Pin 12 is the pin routing to your first relay to open and close it, I called this relay 1  
Pin 14 is the pin routing to your second relay to open and close it, I called this relay 2  
The image here also has pins 11 and 13, it should read 12 and 14.  
![image](https://user-images.githubusercontent.com/2230723/132257811-fc9be30f-31b6-4bea-927c-6256f8b86cd9.png)

Power up the Raspberry Pi now if you'd like and connect your garage door button wire to the relay. I used relay 2 on accident, so everything here is for relay 2. If you don't connect this up, you'll hear the relays open and close anyway, so you'll know its working though it isn't hooked up. The wire I used, since I wanted it to be stiff, is solid core and a twisted pair. Colored, so I know which one goes where. You could probably use Ethernet cable too if you wanted and had that laying around. Bell wire works well, but also is, as of writing this, $18. https://www.lowes.com/pd/Southwire-100-ft-20-2-Twisted-Doorbell-Wire-By-the-Roll/3369292

You'll need to divine what your IP address is for your Raspberry Pi when it comes up and joins your Wi-Fi network. You can get this from your router's DHCP table, your DHCP server's leases, or be lazy and run something like Angry IP Scanner (requires Java). https://angryip.org/. 

When you have Angry Ip Scanner open, click on Tools at the top and then Fetchers. Add some into what you are looking for.

![image](https://user-images.githubusercontent.com/2230723/132258042-f504dd72-10fc-4159-ad28-35ec4f53f2ae.png)

Open the Preferences from the Tools menu and change your Pinging Method to Combined UDP and TCP

![image](https://user-images.githubusercontent.com/2230723/132258092-07892322-63f2-43f4-bda0-052d480a78e5.png)

On the Ports tab of the Preferences, change the ports to 21,22,23,53,80,443,2456,2457,2458,8080,8443,19132,25565,32400. While this is a lot its what I use to look at my network to see who is talking.

![image](https://user-images.githubusercontent.com/2230723/132258113-0e0102d5-af2b-4a4b-af54-89ac8654962d.png)

When your device shows up in the list, it should look something like this:

![image](https://user-images.githubusercontent.com/2230723/132258246-ba2ea195-eb50-4e27-b030-9bd30a5a6806.png)

Open up Powershell on your computer and connect to the Raspberry Pi with ssh (keep in mind your IP may, and probably will, be different):

```
ssh pi@192.168.1.9
raspberry
```

When logged in, run the following commands to get setup:

```
sudo adduser myusername
mypassword
mypassword
```

Hit enter a bunch

```
groups
```

Highlight and copy all the groups that are listed except for pi (the first one, usually), in the script below, when you see pasteHere, paste the selection instead of typing that out.

```
for each in pasteHere; do sudo usermod -a -G $each myusername; done
exit
```

Now ssh back in with your user account you made and lets set it up sort of like the pi user was and remove the pi user. If your account name on the Raspberry Pi is the same as your Windows computer, you can omit myusername from here on out when connecting with ssh.

```
ssh myusername@192.168.1.9
sudo deluser pi
sudo nano /etc/sudoers
```

In the sudoers file, we want to add the following lines near the bottom but before the lines you see here that match what is in the file:

```
# Allow users listed below to use sudo without a password
myusername ALL=(ALL:ALL) NOPASSWD:ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
```

Push CTRL-X to leave nano and make sure you save and exit, overwriting the file. Exit the ssh session again and rejoin it.

```
exit
ssh myusername@192.168.1.9
```

Now that we no longer have the pi user account and our own, with our own password, we're going to setup the Raspberry Pi OS for Powershell. We need to do some updating though before it'll actually work correctly. And we don't have to type in our password when we sudo, but we still have to type sudo. Nice lazy feature. The semi-colons also let us chain commands. Another nice lazy feature.

```
sudo apt update; sudo apt upgrade -y; sudo rpi-update
```

This will update your repositories' info and then upgrade the programs that need upgrading. After those are all done, rpi-update will run and prompt you to push y or N. This updates the firmware on the Raspberry Pi so we can use the GPIO ports with everything we want. Hit y and wait a little bit. After it is done:

```
sudo reboot now; exit
```

When the Raspberry Pi has restarted, ssh back in and run the Raspberry Pi config.

```
ssh myusername@192.168.1.9
sudo raspi-config
```

You'll use the arrow keys, spacebar, tab, and enter to move, select, navigate, and progess through the screens.  
In 1. System Options, select S4 Hostname and set your hostname for your Pi to the name you want it to have, unless you like raspberrypi for some reason.
Select Back.  
In 3. Interface Options, select P4 SPI and P5 I2C, Enable them both.  
In 5. Localisation Options, select L1 Locale, unmark the en_GB.UTF-8 UTF-8, mark the en_US.UTF-8 UTF-8, and select Ok. On the next screen Select the en_US.UTF-8 UTF-8 again.  
In 5. Localisation Options, select L2 Timezone, set it to your timezone.  
After thats set, you'll be back at the main menu, select Finish. Reboot the Raspberry Pi again.  

```
sudo reboot now; exit
```

When it is back up, get back into it and lets install Powershell. At the time of writing this, 7.1.4 is the latest. You can find newer versions by checking here, https://github.com/PowerShell/PowerShell/releases/, and substituting the number in the address within the code block below.

```
ssh myusername@192.168.1.9
sudo apt install libunwind8 -y
wget https://github.com/PowerShell/PowerShell/releases/download/v7.1.4/powershell-7.1.4-linux-arm32.tar.gz
mkdir ~/powershell
tar -zxvf ./powershell-7.1.4-linux-arm32.tar.gz -C ~/powershell
sudo ln -s ~/powershell/pwsh /usr/bin/pwsh
sudo ln -s ~/powershell/pwsh /usr/local/bin/powershell
sudo WIRINGPI_CODES=1 pwsh
Install-Module Microsoft.PowerShell.IoT
Import-Module Microsoft.PowerShell.IoT
Get-Command -Module Microsoft.PowerShell.IoT
exit

pwsh
Install-Module Microsoft.PowerShell.IoT
Import-Module Microsoft.PowerShell.IoT
Get-Command -Module Microsoft.PowerShell.IoT
exit
```

You should have seen it show that the module is installed and cmdlets like Get-GpioPin exist now. This setup installed it for both pwsh and super users pwsh. When you want to control the GPIO pins, use sudo WIRINGPI_CODES=1 pwsh, when you want to just use Powershell, you can just use pwsh.

There are comments throughout the opendoor.ps1 script that will explain what each thing is and does, but the gist of it is that the physical pins do not match the pins we are going to be using, numberwise, so if you hooked everything up like it should be, it should just sort of work. The file is on this repository, you can download it on the Raspberry Pi or nano and copy and paste everything into it. One thing, for simplicity sake, that you'll want to do is change the file to be executable.

```
sudo chmod +x opendoor.ps1
```

That'll make it so you can ssh into the Raspberry Pi and run:

```
sudo ./opendoor.ps1; exit
```

And it'll act as if it were someone pushing the garage door button.
