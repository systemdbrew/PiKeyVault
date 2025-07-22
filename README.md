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
