# Automated unRAID Keyfile Deployment via a Raspberry Pi FTP Server

This guide provides a secure method to automatically deploy an unRAID keyfile during boot using a Raspberry Pi FTP server. This eliminates SSH key complexities while maintaining security through user isolation and read-only access.

## Prerequisites
- Raspberry Pi (any model) on same network as unRAID
- Basic Linux command line knowledge
- unRAID USB boot drive accessible

## Step 1: Raspberry Pi FTP Server Setup

### Install vsftpd
```bash
sudo apt update
sudo apt install vsftpd -y
```

### Configure vsftpd
```bash
sudo nano /etc/vsftpd.conf
```
Modify these lines:
```ini
anonymous_enable=NO
local_enable=YES
write_enable=NO
chroot_local_user=YES
allow_writeable_chroot=YES
```

### Create Dedicated FTP User
```bash
sudo adduser --gecos "FTP" ftpkey
sudo mkdir /home/ftpkey/files
sudo chown ftpkey:ftpkey /home/ftpkey/files
```

### Place Keyfile in Secure Location
```bash
sudo cp /path/to/your/unraid.key /home/ftpkey/files/keyfile
sudo chown ftpkey:ftpkey /home/ftpkey/files/keyfile
sudo chmod 400 /home/ftpkey/files/keyfile
```

### Restart FTP Service
```bash
sudo systemctl restart vsftpd
```

## Step 2: Configure unRAID Boot Script

### Edit go File
```bash
nano /boot/config/go
```
Add this before `/usr/local/sbin/emhttp &`:
```bash
# Wait for network and Pi to be ready
until ping -c1 <Your-Pi-IP> &>/dev/null; do sleep 2; done

# Download keyfile via FTP
curl -s --netrc-file /boot/config/.netrc -o /root/keyfile "ftp://<Your-Pi-IP>/files/keyfile"

# Set permissions
chmod 600 /root/keyfile
```

### Create .netrc Credentials File
```bash
nano /boot/config/.netrc
```
Add these lines (replace with your Pi's IP and password):
```
machine <Your-Pi-IP>
login ftpkey
password YOUR_SECURE_PASSWORD
```

### Set Strict Permissions
```bash
chmod 600 /boot/config/.netrc
```

## Step 3: Verification and Testing

### Test Connectivity
From unRAID console:
```bash
ping <Your-Pi-IP>  # Verify Pi reachability
curl -v --netrc-file /boot/config/.netrc -o /tmp/testfile "ftp://<Your-Pi-IP>/files/keyfile"
```

### Test FTP Server
From any network device:
```bash
curl -v ftp://<Your-Pi-IP>/files/keyfile -u ftpkey
```

## Step 4: Reboot and Validate
1. Reboot both Raspberry Pi and unRAID server
2. unRAID should automatically:
   - Wait for Pi to become available
   - Download keyfile via FTP
   - Start array with retrieved keyfile

## Security Notes
- **User Isolation**: `ftpkey` user has no shell access and restricted to home directory
- **Read-Only FTP**: Server configured with `write_enable=NO`
- **Network Security**: All communication stays within local network
- **Credential Protection**: `.netrc` file has 600 permissions
- **Keyfile Permissions**: Keyfile is stored with 400 permissions on Pi

## Troubleshooting

### Check FTP Logs
On Raspberry Pi:
```bash
sudo tail -f /var/log/vsftpd.log
```

### Common Fixes
1. **Connection Timeouts**: Add delay in `go` file:
   ```bash
   sleep 10  # Extra delay before download
   ```
2. **Permission Issues**: Verify ownership:
   ```bash
   sudo chown -R ftpkey:ftpkey /home/ftpkey/files
   ```
3. **FTP Connection Problems**: Test basic FTP access:
   ```bash
   ftp <Your-Pi-IP>
   (login with ftpkey credentials)
   ```

### Fallback Procedure
Keep monitor/keyboard attached to unRAID for first reboot. If automatic retrieval fails:
1. Manually download keyfile:
   ```bash
   curl -s --netrc-file /boot/config/.netrc -o /root/keyfile "ftp://<Your-Pi-IP>/files/keyfile"
   ```
2. Start array manually

## Maintenance
- **Update Password**: When changing FTP password, update both:
  1. Raspberry Pi: `sudo passwd ftpkey`
  2. unRAID: `/boot/config/.netrc` file
- **Keyfile Rotation**: Replace `/home/ftpkey/files/keyfile` on Pi when updating unRAID license

This solution provides a secure, automated keyfile deployment while eliminating SSH key management complexities. The keyfile is only exposed through a restricted FTP user with no shell access, and transfers occur entirely within your local network.
