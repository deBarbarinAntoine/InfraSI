# Backup Server

This guide walks you through configuring BorgBackup for automated backups on your Ubuntu server.

## Prerequisites

- An SSH key pair (if you don't have one, follow the instructions in the "Generate SSH Key Pair" section below)
- Access to a remote server for storing backups (optional, but highly recommended)

---

## 1. Installation

1. Open a terminal session on your Ubuntu backup server via SSH.

2. Update package lists:

   ```Bash
   sudo apt update
   ```
   
3. Install BorgBackup:
   ```Bash
   sudo apt install borgbackup borgbackup-doc
   ```
   
---

## 2. Configure BorgBackup

### Repository

Decide where to store your backups. We're using a remote server accessible via SSH.

### Steps

1. `borg init ssh://username@server_address:/path/to/repository` (replace with your details)
2. enter a strong passphrase when prompted. This is crucial for accessing and decrypting backups. Store it securely.

---

## 3. Setting Up Automatic Backups

### Script Creation

1. Create a script named backup.sh (or any preferred name) to automate backups using `borg create`.

	Example:
	```Bash
	#!/bin/bash
		
	# Set backup source and destination
	BACKUP_SOURCE="/home"
	BACKUP_DEST="::$(hostname)-$(date +%Y-%m-%d)" # Creates a unique archive name with date
		
	# Exclude specific directories (optional)
	EXCLUDE_DIRS="/home/*/.cache /home/*/.ccache"
		
	# Run BorgBackup with options
	borg create -v --stats --exclude $EXCLUDE_DIRS $BACKUP_DEST $BACKUP_SOURCE
		
	# Optional: Prune old backups (explained later)
	# borg prune --keep-daily=7 --keep-weekly=4 --keep-monthly=6 >> /var/log/backup.log 2>&1
	```
 
2. Save the script in a suitable location (something like /home/user/backup_scripts)

### Make the Script Executable

   ```Bash
   chmod +x backup.sh
   ```

---

## 4. Scheduling Backups

User cron to schedule automatic backups:

1. Edit the crontab:
	```Bash
	crontab -e
	```
 
2. Add a line like this to run the script daily at midnight:

	```Bash
	0 0 * * * /home/user/backup_scripts/backup.sh >> /var/log/backup.log 2>&1
	```

### Adjust the Schedule:
   - Modify the cron expression for hourly, weekly, etc. backups.
   - Update the path to your script as needed.

---

## 5. Manual Restores

### Listing Backups:

1. List available backups:

	```Bash
	borg list ssh://username@server_address:/path/to/repository

	```
This will display a list of archive names with timestamps.

### Restoring a Specific Backup:

1. To restore a specific backup (e.g. your-hostname-2024-05-20)
	```Bash
	borg extract ssh://username@server_address:/path/to/repository::your-hostname-2024-05-20 /destination/folder
	```

2. Replace `/destination/folder` with the location for restored files.

---

## 6. Optional: Prune Old Backups

BorgBackup allows keeping a specific number of backups ( daily, weekly, monthly), uncomment the borg prune line in the script given earlier.

Adjust the option (`--keep-daily=7`, etc.) to define max age of a backup.

### Notice:

- Replace placeholders with your specific details.
- Securely store your BorgBackup passphrase.
- Regularly test backups to ensure functionality.

### Generate SSH Key Pair

```Bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Follow the prompts to save the key (default location is `~/.ssh/id_rsa`) and optionally set a passphrase.

### Copy Public Key to Remote Server:

```Bash
ssh-copy-id user@remote-server
```

This command will prompt for your remote user password and copy your public key to the remote server.

### Verify Passwordless SSH:

```Bash
ssh user@remote-server
```

If successful, your script can now connect to the remote server without requiring a password.





