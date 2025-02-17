<!--
N.B.: This README was automatically generated by https://github.com/YunoHost/apps/tree/master/tools/README-generator
It shall NOT be edited by hand.
-->

# Restic for YunoHost

[![Integration level](https://dash.yunohost.org/integration/restic.svg)](https://dash.yunohost.org/appci/app/restic) ![](https://ci-apps.yunohost.org/ci/badges/restic.status.svg) ![](https://ci-apps.yunohost.org/ci/badges/restic.maintain.svg)  
[![Install Restic with YunoHost](https://install-app.yunohost.org/install-with-yunohost.svg)](https://install-app.yunohost.org/?app=restic)

*[Lire ce readme en français.](./README_fr.md)*

> *This package allows you to install Restic quickly and simply on a YunoHost server.
If you don't have YunoHost, please consult [the guide](https://yunohost.org/#/install) to learn how to install it.*

## Overview

## Overview

A [Restic](https://restic.net/) package for YunoHost (heavily inspired by [the Borg package](https://github.com/YunoHost-Apps/borg_ynh/)).

Restic is a backup tool that can make local and remote backups.
This package uses restic to make backups to a sftp server.
The package does not handle local backups yet but you can work around that by using the local sftp server as target server (see my comment [here](https://forum.yunohost.org/t/sauvegarde-yunohost-avec-restic/10275/33)).


**Shipped version:** 0.12.0~ynh9



## Disclaimers / important information

## Usage

If you want to backup your server A onto the server B.

## Setup Restic app on Server A

Firstly set up this app on the server A you want to backup:

```
$ yunohost app install https://github.com/YunoHost-Apps/restic_ynh
Indicate the server where you want put your backups: serverb.domain.tld
sftp port of your server (default: 22): 2222
The directory where you want your backup repositories to be created in (default: ./): ./servera.domain.tld
Indicate the ssh user to use to connect on this server: servera
You are now about to define a new user password. The password should be at least 8 characters - though it is good practice to use longer password (i.e. a passphrase) and/or to use various kind of characters (uppercase, lowercase, digits and special characters).
Indicate a strong passphrase, that you will keep preciously if you want to be able to use your backups:
Would you like to backup your YunoHost configuration ? [yes | no] (default: yes):
Would you like to backup mails and user home directory ? [yes | no] (default: yes):
Which apps would you backup (list separated by comma or 'all') ? (default: all): gitlab,blogotext,sogo
Allow backup method to temporarily use more space? [yes | no] (default: yes):
Indicate the backup frequency (see systemd OnCalendar format) (default: *-*-* 0:15:00): *-*-* 0:05
Indicate the backup check frequency (see systemd OnCalendar format) (default: Sat *-*-8..31 3:15:00):
Indicate the complete backup check frequency (see systemd OnCalendar format) (default: Sun *-*-1..7 3:15:00):
```

You can schedule your backup by choosing an other frequency. Some example:

Monthly :

Weekly :

Daily : Daily at midnight

Hourly : Hourly o Clock

Sat *-*-1..7 18:00:00 : The first saturday of every month at 18:00

4:00 : Every day at 4 AM

5,17:00 : Every day at 5 AM and at 5 PM

See here for more info : https://wiki.archlinux.org/index.php/Systemd/Timers#Realtime_timer

After each invocation an e-mail will be sent to root@yourdomain.tld with the execution log.

Restic can check backups consistency and verify the actual backed up data has not been modified.
If you use the default values for the backup checks frequencies, a full check will be made on the first day of each month and a simple check will be made on each one of the three remaining weeks of the month.

At the end of the installation, the app displays the public_key and the user to give to the person who has access to the server B.

You should now authorize the public key for user `servera` on server B by logging into server B with user `servera` and running:

```
mkdir ~/.ssh -p 2>/dev/null
touch ~/.ssh/authorized_keys
chmod u=rw,go= ~/.ssh/authorized_keys
cat << EOPKEY >> ~/.ssh/authorized_keys
<paste here the privakey displayed at the end of installation>
EOPKEY
```
If you don't find the mail and you don't see the message in the log bar you can find the public_key with this command:
```
cat /root/.ssh/id_restic_ed25519.pub
```

## (Optional) set sftp jail on server B

To improve security, make sure user `servera` can only do sftp and can only access his home directory on server B.
This is how you would do it on Debian/Ubuntu, otherwise refer to your distribution manual (don't forget to replace `servera` with the real username)

```
cat << EOCONFIG >> /etc/ssh/sshd_config
Match User servera
   ChrootDirectory %h
   ForceCommand internal-sftp
   AllowTcpForwarding no
   X11Forwarding no
EOCONFIG
service ssh restart
```

## Test
At this step your backup should schedule.

If you want to be sure, you can test it by running on server A:
```
systemctl start restic.service
```

Next you can verify the backup contents by running on server A
```
restic -r sftp:serverb.domain.tld:servera.domain.tld/auto_conf snapshots
```

Replace `auto_conf` with `auto_<app>` if you did not choose to backup configuration but only applications.

If you want to check the backups consistency:
```
systemctl start restic_check.service
```

If you want to make a complete check of the backups - keep in mind that this reads all the backed up data, it can take some time depending on your target server upload speed (more on this topic in [the Restic documentation](https://restic.readthedocs.io/en/latest/045_working_with_repos.html#checking-integrity-and-consistency)):
```
systemctl start restic_check_read_data.service
```

## Display the apps list to backup

```
yunohost app setting restic apps
```

## Edit the apps list to backup

```
yunohost app setting restic apps -v "nextcloud,wordpress"
```

## Launch a backup

```
systemctl start restic
```

## Launch a backups check

```
systemctl start restic_check.service
```

## Launch a complete backups check

WARNING: this will read data from your backups destination server.
It may take a quite long time depending on the target server's internet upload speed and hardware performance.

```
systemctl start restic_check_read_data.service
```

## Backup on different server, and apply distinct schedule for apps

You can setup the Restic app several times on the same server so you can backup on several server or manage your frequency backup differently for specific part of your server.


## Documentation and resources

* Official app website: https://restic.net
* Official admin documentation: https://restic.readthedocs.io/en/latest/
* Upstream app code repository: https://github.com/restic/restic
* YunoHost documentation for this app: https://yunohost.org/app_restic
* Report a bug: https://github.com/YunoHost-Apps/restic_ynh/issues

## Developer info

Please send your pull request to the [testing branch](https://github.com/YunoHost-Apps/restic_ynh/tree/testing).

To try the testing branch, please proceed like that.
```
sudo yunohost app install https://github.com/YunoHost-Apps/restic_ynh/tree/testing --debug
or
sudo yunohost app upgrade restic -u https://github.com/YunoHost-Apps/restic_ynh/tree/testing --debug
```

**More info regarding app packaging:** https://yunohost.org/packaging_apps