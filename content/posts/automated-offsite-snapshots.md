---
title: "Automated Offsite Snapshots"
date: 2019-02-09T18:39:04-08:00
draft: false
tags: [ "home-assistant", "backups", "automations" ]
---

# Adding Home Assistant to My Synology/Dropbox Backup Routine

There was a time when I felt comfortable using GitHub as my main backup solution for [Home Assistant](https://www.home-assistant.io). I would backup my secrets.yaml via my workstation backup routine (on the rare times that it changed) and I considered the database to be expendable. As Add-Ons have started becoming commonplace and various components have started moving from yaml to integrations I no longer consider this sufficient.

This solution utilizes the [Remote Backup](https://github.com/carstenschroeder/hassio-addons/tree/master/remote-backup) addon, my home [Synology Diskstation NAS](https://www.synology.com) and [Dropbox](https://www.dropbox.com) via encrypted [Hyper Backup](https://www.synology.com/en-us/dsm/feature/hyper_backup) tasks.

## Requirements

### Durability

We've grown fond enough of the system that I consider it worth at least 3-2-1 for its backups. The chances of meaningful loss with this configuration is very low.

+ 3+ copies
  + Running in HASS
  + Snapshots
    + Previous 3 daily snapshots in Home Assistant
    + Previous 14 daily snapshots (above included) in NAS
  + Backed up to Dropbox
    + Encrypted, versioned, and non-public via Hyperbackup
+ 2+ Media
  + Micro-SD (in my Pi)
  + SATA Hard Drive (in my NAS)
  + Dropbox
+ 1 copy offsite
  + *See also:* Dropbox

Obviously my configs are also still in github, further minimizing risk.

### I want the HASS part to be self contained

If we need to restore from backup, that restore should be everything needed to be up and running again - including the backup routine. To this end I am going to use Hass&#46;io snapshots, the Hass&#46;io remote backup plugin, and vanilla Home Assistant automations.

### Database Too

It's not expendable anymore but also not... mission critical (yet). I am not doing 3-2-1 here but I am doing a daily offsite backup.

## Prerequisites

+ The Home Assistant device and the Synology Diskstation have network routes and access controls that allow SSH traffic.
+ Hypervisor is set up on the Synology Diskstation and configured to use Dropbox.
+ There should be a backup folder in the main volume of the Synology with a home assistant subfolder.
  + `/volume1/Backups/HomeAssistant/`

## Set up

### Generate A Key Pair

A public/private keypair is needed for the remote backup addon (it's also just a good idea). This is best done (saved and securely backed up) somewhere other than the synology.
```bash
ssh-keygen -t rsa -b 4096 -C "hass_backup"
```
I used my workstation; saving them in a backed up directory and in my password manager.

### Synology User

1. From The Users section of the DSM Control Panel select Create <br>
![User creation screen from the Synology DSM](/images/automated-offsite-snapshots_1-user-creation.png)
1. Make sure the user is in the admin group. As of the time of this writing only admin users are allowed SSH. <br>
![Group Management screen from the Synology DSM](/images/automated-offsite-snapshots_2-groups.png)
1. The user should only be given access to two folders: the one where backups are being sent (`volume1/Backups/HomeAssistant`) and the `homes` directory. If the homes directory doesn't exist it will be created in a later step then it can be given read/write access here. <br>
![User permissions screen from the Synology DSM](/images/automated-offsite-snapshots_3-folder-permissions.png)
1. Deny all access to applications <br>
![User application access screen from the Synology DSM](/images/automated-offsite-snapshots_4-applications.png)

### Synology SSH/SCP

1. If SSH isn't otherwise on it needs to be enabled. <br>
![SSH enabling](/images/automated-offsite-snapshots_5-ssh.png)
   1. Make note of the port.
1. In the advanced tab of the Users section of the control panel, scroll to the bottom and enable home directories. <br>
![Enabling Home Directories](/images/automated-offsite-snapshots_6-home-directory.png) 
    1. If the user home directories weren't previously enabled remember to give the `hass_backup` user read/write permissions in the Synology DSM User Control Panel.
1. Change the home directory in `/etc/passwd` to be the `homes` directory that the Synology created when home directories were enabled. SSHing in using the synology user is now possible, but there will be an error about not being able chdir into your home directory. This error won't prevent fixing of the problem.
    ```bash
    sudo vim /etc/passwd
    ```
   1. Change `/var/services/homes/hass_backup` to `/volume1/homes/hass_backup`
   2. I suspect this would be unnecessary if I had enabled home directories before enabling SSH or creating the hass_backup user but I was disinclined to go back and unwind everything to find out.
2. `chmod 0750 /volume1/homes/hass_backup`
3. At this point it should be possible to ssh into the synology using the hass_backup user's password (without any errors).
    ```bash
    ssh hass_backup@<synology ip address>:2222
    ```

#### Enable Public Key Authentication

While logged into the Synology via SSH:

1. `sudo vim /etc/ssh/sshd_config`
1. Uncomment these 3 lines
    ```bash
    #RSAAuthentication yes
    #PubkeyAuthentication yes
    #AuthorizedKeysFile .ssh/authorized_keys
    ```
1. Restart the ssh service
    ```bash
    sudo synoservicectl --reload sshd
    ```
1. `mkdir ~/.ssh`
1. `chmod 0750 ~/.ssh`
1. `vim ~/.ssh/authorized_keys`
   1. Paste the contents of the public key generate above into a new line of this file.
1. At this point it should be possible to ssh into the synology without entering a password.
    ```bash
    ssh -i <location of private key> hass_backup@<synology ip address>:2222
    ```

### Hass&#46;io remote backup plugin

1. Add the repository and install the add-on [per its instructions](https://github.com/carstenschroeder/hassio-addons/tree/master/remote-backup).
1. Configure it with the everything that was just set up. <br>
![Remote Backup Creation Screen](/images/automated-offsite-snapshots_7-remote-backup-config.png)

At this point it should be able to test that backups are being sent to the Synology via the Developer Tools Service panel. It will take a few minutes to perform the backup after the service call is made. <br>
![Service Test Example](/images/automated-offsite-snapshots_8-service-test.png)

### Automation

This automation is taken directly from the remote backup add-on docs. The time was changed and the id was added.

```yaml
- id: daily-backup
  alias: Daily Backup at 00:30
  trigger:
    platform: time
    at: '00:30:00'
  action:
  - service: hassio.addon_start
    data:
      addon: 954f2f4e_remote_backup
```

### Hyper Backup

The hyper backup job is pretty simple. Backup the `HomeAssistant` folder and the MariaDB application (which at the time of this writing is only used by Home Assistant).

1. Create a new "data backup task" with Dropbox as its destination. <br>
![Backup Destination Window](/images/automated-offsite-snapshots_9-create-backup-task.png)
1. A new browser window will open to grant Hyper Backup dropbox permissions. Allow them.
2. Select the Dropbox folder to send to (`SynoBackups`) and what the and what to call within that folder (`HomeAssistant`). <br>
![Backup Destination Screen](/images/automated-offsite-snapshots_10-dropbox-setup.png)
1. Select the Synology folder to back up from the "Data Backup" screen. `Backups > HomeAssistant`
2. Select the MariaDB 10 from the "Application Backup" screen.
3. Set the backup task settings. <br>
![Backup Setting Screen](/images/automated-offsite-snapshots_11-task-settings.png)
    1. The password for this backup is stored in my password manager.
    2. At the end of the wizard you will be prompted to have the backup's encryption key. The backup can be accessed using either the password or the key. If both are kept safe then at lest one should be available when needed.
    3. I chose a Hyper Backup time sufficiently after my Home Assistant backup automation.
1. Choose a rotation schedule. I keep a smart rotation with 128 versions.
2. At the end of the wizard, after saving the backup's encryption key, there is an option to back up now.
   1. This will be the last thing to test.
   2. Note: Backing up Maria DB will make it briefly unavailable.

## Clean Up

My snapshots are only about 50M, but I still don't have a reason to keep them all. I am going to prune all local backups older than 14 days.

1. Create a User Defined Script under Scheduled Tasks <br>
![Scheduled Task Creation Screen](/images/automated-offsite-snapshots_12-scheduled-task-creation.png)
1. The task should run as the backup user. Name it descriptively. I set a scheduled task using the administer account but run the task as the hass backup user. <br>
![Task Creation Screen - General Tab](/images/automated-offsite-snapshots_13-task-settings.png)
1. In the schedule tab choose daily at an time.
2. Under task settings set notification settings for abnormal terminations. The script to run is a linux find command with delete flag <br>
![Task Creation Screen - Task Settings Tab](/images/automated-offsite-snapshots_14-task-creation-general.png) <br>
```
cd ~
find /volume1/Backups/HomeAssistant/ -type f -mtime +14 -delete
```
That change to the home directory at the beginning of this script bypasses a bug currently in syology with non-admin user scripts. Without that the script will throw a permission error.