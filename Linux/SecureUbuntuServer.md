# Setup a Secure Ubuntu Server

This is a guide on how to configure a secure ubuntu server hosted in the cloud. I will in this guide assume that the only initial user is `root`. After first step, we create a user named `admin`. I will also assume the server has the IPs `1.2.3.4` and `172.16.0.2`:


```
- Users.......: root, admin
- Public IP...: 1.2.3.4
- Local IP....: 172.16.0.2
```

## Create a New User

It is generally not a good idea to log in as root. A better idea is to create a new user, and disable root login. We want to make a new user, but still have the same rights as the root user:

```bash
adduser [USERNAME]      # creates the user
adduser [USERNAME] sudo # assignes the group
# i.e.
adduser admin      # create user named admin
adduser admin sudo # assign group sudo to user admin
```

Pro tip: If you want to disable an existing user:

```bash
passwd [USERNAME] -l
passwd [USERNAME] -u # to enable again
```

I then disconnects my session (command `exit` or simply press Ctrl + D), and connect to the server using my new user.

## Updating the System

First thing I usually do, is to make sure that `tmux` is installed on the system. That is because the next thing I usually do, is to make sure the system is fully updated, and I want to make sure that the system does not break if my SSH connection breaks or anything.

```bash
which tmux
# if nothing appears, tmux is not installed. Install with apt:
sudo apt-get install -y tmux
```

Then I start a session, and start updating:

```bash
tmux
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
```

## Generate SSH Keys

It is a very bad idea to use passwords when logging in. A beter idea is to use SSH keys. There are multiple different ways of doing it. I usually generate the SSH key on my local computer using `ssh-keygen`. On Windows, PuTTYgen can be used.

```bash
ssh-keygen -t rsa -b 4096 -C "[LOCAL USERNAME]"@"[LOCAL HOSTNAME]"
# i.e.
ssh-keygen -t rsa -b 3096 -C bryan@BryanRocks.com
```

I usually stick with the default save directory (`~/.ssh/id_rsa`). To be more secure, make sure to password protect the SSH key.

Now, copy the SSH key to the server:

```bash
ssh-copy-id admin@1.2.3.4
# or if you saved the key as ~/.ssh/mykey:
ssh-copy-id -i ~/.ssh/mykey admin@1.2.3.4
```

Then test login in from your local computer:

```bash
ssh admin@1.2.3.4
# or if you saved the key as ~/.ssh/mykey:
ssh -i ~/.ssh/mykey admin@1.2.3.4
```

## Secure SSH Configuration

The default configuration for SSH is generally not very secure. It allows for password authentication and empty passwords etc. To secure the configuration, edit `/etc/ssh/sshd_config` with your favorite editor (i.e. vim or nano). Make sure that the following settings are set:

```
PermitRootLogin no
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no
PubkeyAuthentication yes
```

A good idea is to change the SSH port. By default, SSH use port 22/tcp, but using another port will keep many bots away. Any port could be used, though a common port to use is port 2222. Simply add the following to the `sshd_config` file:

```
Port 2222
```

Then reload the configuraiton:

```bash
sudo systemctl reload sshd
```

Now do not close the session. If you made a mistake in the configuration, you cannot sign in with another session, though your existing session is not disconnected, so that can still be used to fix the mistake. Open a new session:

```bash
ssh -p 2222 admin@1.2.3.4
# or
ssh -i ~/.ssh/mykey -p 2222 admin@1.2.3.4
```

## Firewall

Last step is to enable the firewall. By default, Ubuntu has UFW installed, which is an easier interface for iptables. Let's start by checking which rules are set by default:

```bash
sudo ufw status
```

As you can see from the output, no rules have been set, and the firewall is disabled. If we enable the firewall now, we will be locked out of the server. At the moment, we have no other important services running other than SSH on port 2222. We have two choices when enabling SSH. The most secure option is to limited SSH to the public IP from our home. That way it is not possible to access the server with SSH unless you are connected at home. This can though be problematic if you do not have a static public IP address. Let's enable SSH:

```bash
# If you want to lock connection to your IP:
sudo ufw allow from [IP ADDRESS] to any port 2222
  # i.e. sudo ufw allow from 4.3.2.1 to any port 2222
# If you want to be able to SSH to the server from any IP:
sudo ufw allow 2222
```

Then enable UFW:

```bash
sudo ufw enable
```
