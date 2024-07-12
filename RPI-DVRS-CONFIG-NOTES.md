# DVMHost install for RXP25 on RPi

NOTE: tested by jsb on 2024-06-10 on a fresh RPi 3B image

- Install fresh Raspberry Pi OS LITE 32-bit (debian bookworm) image.
- Log in to device, disable bluetooth and getty on serial:
```
sudo systemctl disable bluetooth.service serial-getty@ttyAMA0.service
sudo systemctl mask serial-getty@ttyAMA0.service
grep '^dtoverlay=disable-bt' /boot/firmware/config.txt || echo 'dtoverlay=disable-bt' | sudo tee -a /boot/firmware/config.txt
sudo sed -i 's/^console=serial0,115200 *//' /boot/firmware/cmdline.txt
```
- reboot

- Update/upgrade and install packages:
```
sudo apt -y update
sudo apt -y upgrade
sudo apt -y install cmake nano git screen build-essential libasio-dev libncurses-dev libssl-dev stm32flash libssl1.1
```

- installing dvmhost software: build softwar from source for first install.
```
       cd ~
       git clone https://github.com/DVMProject/dvmhost
       cd dvmhost
       mkdir build; cd build
       cmake -DENABLE_TCP_SSL ..
       make
       sudo make old_install
       sudo make old_install.service
```

Files will be located in /opt/dvm, with the binaries in /opt/dvm/bin.  Also, a systemd service will be added so you can automatically get it started upon boot.

- customize ```config.yml``` with the right parameters (templates will be provided to you by the admins to help with this):
  - ensure ALL file paths are fully qualified paths:
     - ```filePath```, ```activityFilePath```, ```file:``` entries under ```iden_table:```, ```radio_id:``` and ```talkgroup_id:``` sections
  - set the following parameters in ```config.yml``` (only partial options shown, please edit your existing templated file) for the network section:
```
network:
# enter your node ID as provided by the RXP25 admins, usually your RadioID + 01, 02, 03... unless you have a 6-digit repeater ID.  THIS MUST BE UNIQUE
  id: XXXXXX 

# RXP25 primary FNE IP address
  address: XXX.XXX.XXX.XXX

# Do not change this unless an admin tells you to.
  port: XXXXX

# RXP25 FNE access password.
  password: "XXX"

# link between FNE and DVMHost is encrypted.
  encrypted: true

# to be provided by admins
  presharedKey: "XXX" 

# set both of these to true unless you know what you're doing - FNE provides the ACLs and rules
  updateLookups: true  
  saveLookups: true

# optional but this really helps with troubleshooting - set both to true
  allowActivityTransfer: true
  allowDiagnosticTransfer: true

# [...]
```
   - in ```system:```, please set your ```identity:``` to the following format: ```VE0ABC-1``` (replacing 1 with the hotspot number).  If this is a repeater, feel free to use the repeater callsign (VE0XYZ) - max 8 chars
        - Fill location info under ```location:```
        - Under ```config:```
            - ensure your channel configs line up with the ```iden_table.dat```, you know how to do this.  If not, reach out to an admin
            - Ensure ```netId:``` is set to ```0CA25``` and sysId to ```C25```
            - ```rfssId``` and ```siteId``` are unique to your site and must be obtained by an admin
        - under ```cwid:``` define whatever cwid you want/need
        - under ```modem: / protocol:```, set ```type:``` to ```uart``` and set the port to the right TTY port (usually ttyAMA0)
- check rest of config options and save

- IMPORTANT STEPS: FLASH FIRMWARE IF NOT DONE YET, AND CALIBRATE YOUR MODEM!!! (not in this document, out of scope)

- when ready, enter the following commands to start and activate the dvmhost service:
```sudo systemctl enable --now dvmhost.service```
