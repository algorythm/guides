# Mount a Network Share

This guide shows how to mount a network share in linux. In this guide I will connect to a share called `files` on the server located at `172.16.0.2`, with the username `admin`.

Make sure to create a folder for the share:

```bash
sudo mkdir -p /mnt/shares/files
```

Then connect:

```bash
sudo mount -t cifs -o username=admin //172.16.0.2/files /mnt/shares/files
```

Note: If you receive the error `wrong fs type, bad option, bad superblock on...`, make sure you have cifs-utils installed, it may not be installed on your distro by default.

```bash
sudo apt-get install cifs-utils
```
