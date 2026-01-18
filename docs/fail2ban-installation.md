# Prereqs

### Verify OS Version
`cat /etc/os-release`
Ubuntu 22.04.5 LTS

### Check disk space
`df -h`
*-h specifies human readable formats for the sizes like KB, MB, or GB rather than everything in KB*
![Disk Space Verification](../assets/disk-space.png)


# Log Setup

1.  Share the log folder on Windows
	![Windows Share Setup](../assets/windows-share-setup.png)
	- Make sure permissions are only set to Read, we don't want this folder being modified at all. Click OK.
	- Go to the security tab next. Select edit and add, then type "everyone" and check names. Make sure only read is selected in permissions for everyone. I had to uncheck "read and execute" and "modify"
	- Note your IP address: if you don't know, use Win+R, cmd, ipconfig
2.  Mount it on the Pi
	Install CIFS utilities (for mounting Windows shares)
	make mount point for the logs
	change ownership of the directory
	```bash
	sudo apt update
	sudo apt install cifs-utils -y
	sudo mkdir -p /mnt/jellyfin-logs
	sudo chown <username>:<group> /mnt/jellyfin-logs
	```
	mount the drive:
	`sudo mount -t cifs //<WINDOWS_SERVER_IP>/jellyfin-logs /mnt/jellyfin-logs -o credentials=/etc/jellyfin-creds.txt,ro,uid=1000,gid=1000`
	- -t specifies type (cifs)
	- -o tells it to use a credentials file with the windows credentials
	- ro is read only, and uid and gid can be found by running `id "username"`
	I test and we can see the folder from the Pi:
	![Log Folder Verification](../assets/log-folder-mount.png)
	Added the drive mapping/mounting to file system table (fstab) so that it would be permanent: `//<WINDOWS_SERVER_IP>/jellyfin-logs /mnt/jellyfin-logs cifs credentials=/etc/jellyfin-creds.txt,ro,uid=1000,gid=1000,_netdev,x-systemd.automount 0 0`
3. Find an actual failed login line from the log
	generated a failed login attempt by signing out and using the incorrect password to sign into my account:
	`[2026-01-17 11:30:21.496 -06:00] [INF] [50] Jellyfin.Server.Implementations.Users.UserManager: Authentication request for "<username>" has been denied (IP: "<client-ip>").`

**Issue:** Fail2Ban rule showed 0 packets matched despite successful ban log. **Discovery:** Validated that active, established sessions bypass `INPUT` chain rules in stateful firewalls. **Validation:** Tested via Private/Incognito mode to ensure a `NEW` connection state was triggered. **Result:** Confirmed 100% mitigation for unauthorized external attempts.
