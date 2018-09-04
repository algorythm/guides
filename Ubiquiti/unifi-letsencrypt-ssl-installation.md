# Installing SSL Certificates from Lets Encrypt

Most guides online covers how to purchase a new SSL certificate, 
and installing it using `/usr/lib/unifi/lib/ace.jar`. This guide
is for generating and installing SSL certificates from LetsEncrypt
which are free, and can be automated.

SSH into the server, and download my [automation repository](https://github.com/algorythm/automation) to `/opt`:

```bash
# Log in as root user
su - 
# Download automation repository
git clone https://github.com/algorythm/automation/ /opt/automation
```

You need to make sure that port 80 and 443 are both to the world. If nginx or apache2 is running on that port, make sure to stop them first. 

```bash
ufw allow 80           # if using ufw
ufw allow 443          # if using ufw
systemctl stop nginx   # if running
systemctl stop apache2 # if running
```

Now we are ready. If you need to generate certificates for `unifi.example.com` and install them on your unifi controller:

```bash
/opt/automation/unifi-le-cert.sh -d unifi.example.com -e [YOUR EMAIL ADDRESS]
```

If you already have generated lets encrypt certificates, and only need to install them:

```bash
/opt/automation/unifi-le-cert.sh -d unifi.example.com -i
```

If you need to renew certificates, and reinstall the new certificates if they are due:

```bash
/opt/automation/unifi-le-cert.sh -d unifi.example.com -r
```

To automate renwal of SSL certificates, edit crontab:

```bash
crontab -e
```

Then insert the following line:

```
@daily /opt/automation/unifi-le-cert.sh -d unifi.example.com -r
```

Best practice is to try to renew daily. SSL Certificates from Lets Encrypt have an expiration date of three months.
