# Vagrant with Virtual Box

## Problem

I tried to install homestead with Parallels and VirtualBox. My issue is that when running `vagrant up`, I get the following error:

> Bringing machine 'default' up with 'virtualbox' provider... [default] Clearing any previously set forwarded ports... [default] Clearing any previously set network interfaces... There was an error while executing `VBoxManage`, a CLI used by Vagrant for controlling VirtualBox. The command and stderr is shown below.
>
> Command: ["hostonlyif", "create"]
>
> Stderr: 0%...
> Progress state: NS_ERROR_FAILURE
> VBoxManage: error: Failed to create the host-only adapter
> VBoxManage: error: VBoxNetAdpCtl: Error while adding new interface: VBoxNetAdpCtl: ioctl failed for /dev/vboxnetctl: Inappropriate ioctl for devic
> VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005), component HostNetworkInterface, interface IHostNetworkInterface
> VBoxManage: error: Context: "int handleCreate(HandlerArg*, int, int*)" at line 66 of file VBoxManageHostonly.cpp

## Solution

The first suggested solution was to restart Virtual Box

```bash
sudo /Library/StartupItems/VirtualBox/VirtualBox restart
```

This did not work as `/Library/StartupItems/VirtualBox/VirtualBox` did not exist. The actual solution was to run

```bash
sudo launchctl load /Library/LaunchDaemons/org.virtualbox.startup.plist
```

And then of course

```bash
vagrant up
```

- Solution 1: <https://stackoverflow.com/questions/21069908/vboxmanage-error-failed-to-create-the-host-only-adapter>
- Solution 2: <https://stackoverflow.com/questions/18149546/vagrant-up-failed-dev-vboxnetctl-no-such-file-or-directory>
