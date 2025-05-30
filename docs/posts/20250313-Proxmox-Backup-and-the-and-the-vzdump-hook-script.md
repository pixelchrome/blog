---
title: "Proxmox Backup and the and the vzdump hook script"
date:
  created: 2025-03-13

linktitle: "Proxmox Backup and the and the vzdump hook script"
slug: "proxmox-backup-and-the-and-the-vzdump-hook-script"

tags:
- Linux
- Proxmox
- Backup

authors:
- harry
---
## My setup

I use the Proxmox Backup Server to create backups of my VMs. The Proxmox Backup Server runs as a VM on a different (TrueNAS) server. The server is connected to a PDU. Therefore the PDU Outlets must be switched on and off...
<!-- more -->
Here is my (preliminary) vzdump hook script

```sh
#!/bin/bash

#set -x

remote_user="<remote-user>"
remote_host="<remote-host>"
backup_host="<proxmox-backup-server>"
remote_storage="vaultpbs"
replication_process_name="zettarepl:"

check_backup_task="ssh ${remote_user}@${backup_host} proxmox-backup-manager task list"
check_replication_task="ssh ${remote_user}@${remote_host} pgrep -f ${replication_process_name}"
shutdown_backup_host="ssh ${remote_user}@${backup_host} shutdown -P now"
shutdown_remote_host="ssh ${remote_user}@${remote_host} shutdown -P now"

if [ "$1" == "job-init" ]; then
    logger "Backup starts. Wake ${remote_host}"
    # PDU Power on server vault and switch
    snmpset -v2c -c private 10.0.10.60 .1.3.6.1.4.1.1718.3.2.3.1.11.1.1.2 i 1
    snmpset -v2c -c private 10.0.10.60 .1.3.6.1.4.1.1718.3.2.3.1.11.1.1.3 i 1
    echo "waiting..."
    sleep 600
    echo "pinging..."
    ping $remote_host -c 10
    /usr/sbin/pvesm set $remote_storage -disable false
fi
if [ "$1" == "job-end" ]; then
    logger "Backup finished"
    while true; do
        output_backup_task=$(eval ${check_backup_task})
        output_replication_task=$(eval ${check_replication_task})

        # Check if backup or replication is still running
        if [ -n "$output_backup_task" ] || [ -n "$output_replication_task" ]; then
            logger "Backup Jobs still running on ${remote_host}"
        else
            logger "Backup finished. Shutdown ${backup_host}"
            $shutdown_backup_host
            sleep 300
            logger "Backup finished. Shutdown ${remote_host}"
            $shutdown_remote_host
            break  # Exit the loop if the process is not running
        fi
        sleep 120  # Wait for 120 seconds before checking again
    done
    /usr/sbin/pvesm set $remote_storage -disable true
    sleep 600

    # PDU Power off server vault and switch

    snmpset -v2c -c private 10.0.10.60 .1.3.6.1.4.1.1718.3.2.3.1.11.1.1.2 i 2
    snmpset -v2c -c private 10.0.10.60 .1.3.6.1.4.1.1718.3.2.3.1.11.1.1.3 i 2

fi
exit 0
```

### The purpose of this script.

#### Before backing up the VMs

1. Switching on the server
2. Wait until the server is running
3. Activate the remote_storage - backup destination

#### After the backup of the VMs

1. Check whether other processes are running on the backup server
	- other Proxmox backup processes
	- or (TrueNAS) replication processes
2. Shutdown of the PBS VM
3. Shutdown of the server
4. Deactivate the remote_storage - backup destination
5. Switch off the PDU outlets

## Add the script to the backup job

After creating the script, it needs to be activated so that it is used during a backup job.

Search for the backup ID in the `/etc/pve/jobs.cfg` file.

`/etc/pve/jobs.cfg`

```cfg
vzdump: backup-07cdf241-8b56
	schedule 2:00
	all 1
	enabled 1
	fleecing 0
	mode snapshot
	notes-template {{guestname}}
	notification-mode notification-system
	repeat-missed 0
	storage vaultpbs
```

Add with the script with `pvesh set /cluster/backup/backup-07cdf241-8b56 --script /usr/local/bin/vzdump-hook-script`.

`/etc/pve/jobs.cfg`
```cfg
vzdump: backup-07cdf241-8b56
	schedule 2:00
	all 1
	enabled 1
	fleecing 0
	mode snapshot
	notes-template {{guestname}}
	notification-mode notification-system
	repeat-missed 0
	script /usr/local/bin/vzdump-hook-script
	storage vaultpbs
```
