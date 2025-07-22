# UASEA
Unraid Autostart Encrypted Array

Step 1: Set Up FTP Server on Raspberry Pi

    Install vsftpd:
    bash

sudo apt update
sudo apt install vsftpd

Configure vsftpd:
bash

sudo nano /etc/vsftpd.conf

Modify these lines:
ini

anonymous_enable=NO
local_enable=YES
write_enable=NO
chroot_local_user=YES
allow_writeable_chroot=YES

Create Dedicated User:
bash

sudo adduser --gecos "" --disabled-password ftpkey
sudo mkdir /home/ftpkey/files
sudo chown ftpkey:ftpkey /home/ftpkey/files

Place Keyfile:
bash

sudo cp /path/to/your/unraid.key /home/ftpkey/files/keyfile
sudo chown ftpkey:ftpkey /home/ftpkey/files/keyfile
sudo chmod 400 /home/ftpkey/files/keyfile

Restart FTP Service:
bash

sudo systemctl restart vsftpd
