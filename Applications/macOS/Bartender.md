# Bartender 3

Bartender unclutters the status bar:

![macOS Statusbar with Bartender](statusbar.png)

## Issue: Not starting on login:

It is a known issue with Bartender that it might not be able to start with macOS on login. Luckily the [solution](https://www.macbartender.com/b3_knowledge/not-starting-at-login/) is quite simple. Open a terminal and enter:

```bash
xattr -d -r -s com.apple.quarantine '/Applications/Bartender 3.app'
```
