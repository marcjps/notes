
# Linux Networking Configuration

### Wireless network configuration

Create wifi connection

	nmcli dev wifi con "ssid" password "password"

Show network devices

	nmcli dev show 

Toggle a connection

	nmcli con down id "Exoplanet"
	nmcli con up id "Exoplanet"

Connections are saved in /etc/sysconfig/network-scripts/

To set the host name, edit /etc/hostname

To see IP address:
	
	ip addr


### Firewall

List open ports:

```text
sudo firewall-cmd --list-all
```

Add a port:

```text
sudo firewall-cmd --add-service=http --permanent
```

Then reload:

```text
sudo firewall-cmd --reload
```

### Using SSHFS

SSHFS lets you mount a file system on another server as a mount point, over SSH.  

Install:

	yum install sshfs

Mount:

	sshfs -o idmap=user emote@sdf.lonestar.org:/arpa/af/e/emote /mnt/sdf
	sshfs -o idmap=user emote@sdf.lonestar.org:/www/af/e/emote /mnt/www

The idmap=user option maps the local user account to the remote one, so that files owned by your remote account appear to owned by your local one.

Unmount:

	fusermount -u /mnt/sdf

If you have open permissions on the mount point, then all users are allowed to mount it.  E.g. 

	chmod 777 /mnt/sdf

SSHFS will use private key authentication.  Just ensure your private key is in ~/.ssh/id_rsa and your public key has been copied to the authorized_keys on the remote host.

### Using samba

Samba implements supports for SMB, the method Windows uses for sharing files.

Installation:

	yum install samba-client cifs-utils

List shares

	smbclient -L luna -U ms

Mount a share.  Note that the filesystem location /mnt is for mounting network drives.

	mount -t cifs //zen/C' /mnt/zen -o username=ms
	mount -t cifs //luna/HRM' /mnt/zen -o username=ms

Ensure that you have cifs-utils or this won't work.

Unmount share

	umount /mnt/luna
	umount /mnt/zen

Fix for "cannot allocate memory": http://jlcoady.net/windows/how-to-resolve-mount-error12-cannot-allocate-memory-windows-share

Smbclient interactive mode
	
	smbclient //luna/c


