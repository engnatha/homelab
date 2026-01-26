## Server Setup Procedure
This doc covers the high level details that each server was taken through from unboxing to network configuration.

### ISO Creation
For simplicity, I went with creating a bootable USB media. In a more advanced setup in the future, it would be desirable to bring
servers online using a netboot option. There are many guides on doing this and the steps are dependent on the OS you're using to
make the bootable drive. There are many tools like [ventoy](https://www.ventoy.net/) for doing this, so it's left as an exercise to
the reader.

### Boot from the USB
Every machine is different in the startup phase. Spamming `ESC`, `F9`, `F12`, or other common keys _immediately_ after powering on
is essential. Once you hit the BIOS menu, it should hopefully be obvious to walk through selecting the USB as the boot drive. For
the HP EliteDesk, this was actually annoying to set up. I had to go through the Boot Options menu and change the boot order to prefer
the USB first over the Windows installation the vendor provided.

Once the server boots, you can go through the installation of your OS how you'd like. Since I was installing Ubuntu and confident that
I wanted it, I went with the interactive installer, chose the default options, and erased all data on the drive during the installation
process.

This is also the fun part to choose server hostnames. I'm naming all my servers after Italian cities. The first three are ROME, SALERNO,
and FLORENCE.

### Configure passwordless SUDO
Totally optional, but this is really convenient if you're going to regularly mess with root settings. If your `/etc/sudoers` file ends with
`@includedir /etc/sudoers.d`, then you can make a file like `/etc/sudoers.d/passwordless-user` with the following content.
```
<username> ALL=(ALL) NOPASSWD:ALL
```

### Configure SSH Access
Since this is the first boot up after installation, it's a good option to run `sudo apt-get update && sudo apt-get -y update`. Configuring ssh
is easy from there.
1. `sudo apt -y install ssh`
2. `sudo systemctl enable ssh`

From a separate machine, you should be able to ssh into the device using the user/pass combo from initial setup. If you'd like to add ssh keys,
this is a good time to add those to `~/.ssh/authorized_keys`.

### Configure Networking
For my application, I want a static IP address. This change can be done in a few simple steps.
1. `sudo nano /etc/netplan/01-static-ip.yaml`
2. Apply the contents from [template](../src/netplan_templates/01-static-ip.tpl.yaml) filling in the details appropriately
3. `sudo chmod 0600 /etc/netplan/01-static-ip.yaml`
4. `sudo netplan apply`

If you're doing this over ssh, your session will terminate. You should be able to `ping` the device by the new IP address and `ssh` in again.

### Disable Wireless Modules
Since my servers are hardwired on my home network, I do not need Wifi access. To be security conscious, I also opted to disable the bluetooth integration
on each server. Here's the procedure for doing that.
1. `sudo nano /etc/modprobe.d/disable-wireless.conf` to modify kernel module behavior
2. Add the following exclusion directives
```
# Disable Wi-Fi
blacklist iwlwifi
blacklist cfg80211
blacklist mac80211

# Disable Bluetooth
blacklist bluetooth
blacklist btusb
```
3. `sudo update-initramds -u` to update kernel settings
4. `sudo systemctl disable --now bluetooth.service` to disable bluetooth systemd service
5. `sudo nmcli radio wifi off` to disable wifi devices
6. `sudo nano /etc/NetworkManager/conf.d/disable-wifi.conf` and add the following contents to make this persistent
```
[keyfile]
unmanaged-devices=type:wifi
```
7. `sudo systemctl restart NetworkManager` to apply
8. `sudo reboot` to apply all the changes

After a successful reboot, there a few quick ways to verify the changes were applied coorectly. `ip link` shouldn't show any wireless interfaces. These will 
typically start with `wl` if present. `hciconfig` should show no devices. `rfkill list` should also show no devices.
