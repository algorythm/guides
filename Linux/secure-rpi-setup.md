# Configuring a secure Raspberry Pi

This guide will configure a simple secured linux server. In this example, I will configure a raspberry pi.

## Prepare SD Card from Mac OS X

This section is for preparing an SD card with Raspbian from a Mac. Start by opening a terminal. Start by downloading the latest version of raspbian:

```bash
curl -O -J -L https://downloads.raspberrypi.org/raspbian_lite_latest
```

This might take a while since the file a fairly large. When the dowload is complete, unzip the zip-file:

```bash
unzip *-raspbian*-lite.zip
```

When that has finished, there should be a file named `[date]-raspbian-[release_name]-lite.zip`. At this point in time, the latest release for me is called `2018-04-18-raspbian-stretch-lite.zip`.

Now plug in your SD card. Find out the right diskt using diskutil:

```bash
diskutil list
```

I can usually recognize the correct device by looking at the size, however since the next few steps are extremely dangerous, it is recommended if you are even the slightest in doubt to plug out the SD card, run the command, then plug in back in and run the command again. Then compare the output to make sure it is indeed the correct device.

For me the device is `/dev/disk3`. Now, unmount the device:

```bash
diskutil unmountDisk /dev/disk3
```

Now the dangerous part comes. We will be using DiskDestroyer (`dd`) to copy the image file to the SD card, again **make sure you are using the correct device**. If your device is `/dev/disk3` like mine, you need to use `/dev/rdisk3` for this next command (notice the `r` in font of `disk`):

```bash
sudo dd bs=1m if=2018-04-18-raspbian-stretch-lite.img of=/dev/rdisk3 conv=sync status=progress
```

`if` is the input file while `of` if the output file. So it is important to make sure these are correct. `dd` command will not backup anything, so if you fucked up somewhere, you are screwed. Double check everything!

When that command has finished, make sure the SD card is still mounted. Then to enable SSH (pretty much needed for a headless server), make a file called `ssh`:

```bash
touch /Volumes/boot/ssh
```

To make the raspberry pi automatically connect to a wifi access point:

```bash
vim /Volumes/boot/wpa_supplicant.conf
```

And add this to it:

```conf
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
	ssid="[SSID]"
	psk="[password]"
}
```

Now eject the SD card

```bash
diskutil eject /dev/disk3
```

plug it into the Raspberry Pi and turn it on.

One easy way to connect to it is by powering it using USB from the computer, however it is possible to find it fairly easy on the network as well. If you connect it to the computer, connect to it using dns:

```bash
ssh pi@raspberrypi.local
```

Or if you need to find it first, use `nmap` (`brew install nmap` if you don't have it):

```bash
nmap 10.20.0.0/24 -PS22
```

Replace the ip-range to how your network is configured. The raspberry pi should pop up there if everything worked out. Now proceed to the next steps in this guide.

## Access the server

To find the server, let's scan the network for devices:

```bash
nmap 10.20.0.0/24 -PS22
```

I use `-PS22` to only see hosts with port 22 (SSH) open, which the raspberry pi has.

In my case, `10.20.0.103` is my raspberry pi:

```
Nmap scan report for 10.20.0.103
Host is up (0.00091s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
```

If you are using my dotfiles (https://github.com/algorythm/dotfiles), simply run `n22` to get a list of IPs where port 22 is open.

Now SSH into the device. Default username and password for Raspberry Pi's are:

**Username**: pi, **Password**: raspberry

## Update

Make sure the device is up to date with updates:

```bash
sudo apt-get update
```

Before upgrading the system, I usually like to install vim and tmux. Tmux mostly so I can update in a window

```bash
sudo apt-get install -y vim tmux git
sudo update-alternatives --config editor    # Update default text editor to vim
# or to be explicit:
sudo update-alternatives --set editor /usr/bin/vim.basic
```

Then update the system:

```bash
tmux new -s update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
# ctrl - b -> d to deattach from tmux window
tmux a -t update # attach to tmux window again
```

The raspberry pi has a bit slow CPU, so in this case it might take a while.

### Error when creating a new tmux screen

If you get the following error after trying to start tmux:

```
tmux: invalid LC_ALL, LC_CTYPE or LANG
```

You probably have an issue with the configured locale. One solution may be to install and reconfigure the locale used. By default, only `en_US_UTF8` is installed. For me, raspbian were configured to use `en_GB_UTF8`. You might be missing the used locale. Start by installing `locales` if it is not installed already

```bash
sudo apt-get install locale
```

The `locale` command can be used to display information about the current locale

```bash
locale
```

To configure a new locale

```bash
sudo dpkg-reconfigure
```

Select the chosen locale, mark it by pressing `space`, then continue by pressing `enter`.

## Raspi-config

**NOTE**: This is only needed for Raspberry Pis!

```bash
sudo raspi-config
```

Do following:

1. **Change the Pi password**
2. **Disable "Boot to Desktop"** - it's simply not needed for most cases
3. **Update locale settings**
4. **Set the hostname** (Network > Hostname)
5. **Set the Memory Split** (Advanced > Momery Split) - set it to **16** since the desktop is not going to be used
6. **Ensure SSH is enabled** (Advanced > SSH) - if you completed the step above, then SSH is enabled
7. **Commit the changes and reboot**

```bash
sudo reboot
```

## Creating a new user

See a list of groups:

```bash
groups
```

In my case, I get the following output:

```
pi adm dialout cdrom sudo audio video plugdev games users netdev input
```

~Issue following command to add a user with correct permissions (at least for Raspberry Pis):~

Best way to add a user is to issue the following command:

```diff
- sudo useradd -m -G adm,dialout,cdrom,sudo,audio,video,plugdev,games,users,netdev,input USERNAME
+ sudo adduser USERNAME
+ sudo adduser USERNAME sudo # to add the user to the group sudo
```

Replace **USERNAME** with your desired username. Then, if the user needs sudo:

```bash
sudo visudo
```

~Find a line that says `root ALL=(ALL:ALL) ALL` and add a new line with your newly added user:~

```diff
root  ALL=(ALL:ALL) ALL
- awo   ALL=(ALL:ALL) ALL
```

To change a password:

```bash
sudo passwd USERNAME
exit
```

Now sign in again, though this time as your new user. Then, ~delete~disable the `pi` user:

```diff
- sudo deluser --remove-all-files pi
+ sudo passwd pi -l
  # Unlock "pi" user: sudo passwd pi -u
```

## Generate SSH-Key

SSH keys are generally a lot more secure than logging in with SSH. To generate a new SSH key:

```bash
ssh-keygen -t rsa -b 4096 -C "USERNAME@HOSTNAME"
```

I usually stick with the default save location of `~/.ssh/id_rsa`. To be more secure, make sure to password protect the SSH key.

Then, disable root login as well as login with password using SSH. Edit `/etc/ssh/sshd_config`, make sure that the following settings are set:

```bash
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords yes
PubkeyAuthentication yes
```


## Connecting USB Drives

```bash
sudo fdisk -l

# Look for the USB drive (i.e. /dev/sda1
sudo mkdir /my-folder
sudo mount /dev/sda1 /my-folder
sudo chgrp -R users /my-folder
sudo chmod -R g+w /my-folder
sudo vim /etc/fstab
```

Press `o` to start a new line and write:

```
/dev/sda1 [tab] /my-folder [tab] ext4 [tab] defaults [tab] 0 [tab] 2
```

Press `[ESCAPE] :wq [ENTER]` to save and exit.

The group called "users" now has read and write access to the usb drive located at `/dev/sda1`, mounted at `/my-folder`.

**NOTE**: For testing purposes, `sudo mount -a` mounts everything in the fstab file.

