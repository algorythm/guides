# Ubiquiti Unifi AP Adoption and Reset

This guide will show how to adopt and reset Ubiquiti Unifi APs.

## Physical Reset

1. Press and hold the **reset** button for 10 seconds.
2. Release the button (the LEDs on the **UAP** will stop glowing).
3. Do not disconnect the **UAP** from its power source (the ethernet cable) during the **reboot** process.
4. The **UAP** will restore factory settings.

## SSH Reset

Access the UAP via SSH, and once in, isse the commands:

```bash
syswrapper.sh restore-default
```

The UAP should quickly reboot with factory default settings. Remember to not disconnect UAP from power source during this process.

## Adoption over SSH

Start by finding the IP address for the access point. For this example, let's say the IP is `172.16.0.2`. The default username is `ubnt`, default password is `ubnt`. Then SSH to the access point.

For this example, I want to connect my UAP to a hosted Unifi controller, let's say the FQDN is `unifi.example.com`.

1. Make sure the AP is running updated firmware. If it is not, see this guide: [UniFi - Changing the Firmware of a UniFi Device](https://help.ubnt.com/hc/en-us/articles/204910064-UniFi-Upgrading-firmware-image-via-SSH).
2. Make sure the AP is in factory default state. If it's not, do:

```
sudo syswrapper.sh restore-default
```
3. SSH into the device and type the following and hit enter:
```
set-inform http://unifi.example.com:8080/inform
```
4. After issuing the set-inform, the UniFi device will show up for adoption on the Unifi controller. Once you click **adopt**, the device will appear to go offline.
5. Once the device goes offline, issue the command from step 3 again, and the device will start provisioning:
```
set-inform http://unifi.example.com:8080/inform
```
