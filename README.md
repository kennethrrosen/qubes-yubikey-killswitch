# qubes-yubikey-killswitch
A simple, functional killswitch for use with QubesOS and Yubikey

Here are some great resources for using Yubikey with QubesOS:
https://www.qubes-os.org/doc/yubikey/
https://forum.qubes-os.org/t/u2f-only-w-yubikey/12304
https://github.com/QubesOS/qubes-app-yubikey

However, I wanted to use my Yubikey in order to access 2FA apps (specifically my vault and KeePassXC directory) while still using the Yubikey to shutdown the computer on removal (the guides above provide for 2FA with the Yubikey, or a lock-screen option).

Here's what I did. First, follow the guide here: https://www.qubes-os.org/doc/yubikey/
Then make the following changes:

### In Dom0 
In `/etc/qubes-rpc/custom.LockScreen`
```
sudo shutdown now
```

### In sys-usb
In `/rw/config/yubikey.rules`
```
ACTION=="remove", SUBSYSTEM=="usb", ENV{ID_SECURITY_TOKEN}=="1", RUN+="/rw/config/yubikey-detach.sh"
```
In '/rw/config/yubikey-detach.sh'
```
#!/bin/bash

# Introduce a delay to give time for YubiKey to get assigned to VM, if that's the case
sleep 5

# Check if YubiKey is still connected
if lsusb | grep -iq yubico; then
    exit 0 # Exit quietly if YubiKey is still connected
else
    /usr/bin/qrexec-client-vm dom0 custom.LockScreen # Trigger RPC if YubiKey is not connected
fi
```
Then `sudo chmod +x /rw/config/yubikey-detach.sh && sudo udevadm control --reload-rules && sudo udevadm trigger` 


