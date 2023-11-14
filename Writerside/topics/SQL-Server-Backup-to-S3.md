# SQL Server Backup to S3


## Overview

Protecting the data in the MySql Database by taking a backup and storing it in S3.

## Using a Timer Service to Backup the Database

This service works using systemd timers. currently there's a timer service in: /etc/systemd/system/backup.timer

```ini
[Unit]
Description=Backup MySql Timer

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target#
```

This service triggers at 03:00 every day. It then runs the backup.service.

```ini
[Unit]
Description=Backup The K3S Master Database to S3.

[Service]
ExecStart=/bin/bash /opt/backup/backup.sh

[Install]
WantedBy=default.target
```

This service file also in /etc/systemd/system/ will execute the backup.sh script.

## Backup Script

backup.sh



```bash
#!/bin/bash


file="kine-$today.sql"
mysqldump --defaults-file=/etc/mysql/debian.cnf -u debian-sys-maint homelab > $file

tar_file="kine-$today.tar.gz"
tar -czvf $tar_file $file
rm $file

export AWS_ACCESS_KEY_ID=<Access Key>
export AWS_SECRET_ACCESS_KEY=<Access Secret>
export AWS_DEFAULT_REGION=eu-west-1
export AWS_DEFAULT_OUTPUT=json
aws s3 cp $tar_file s3://k8s-foulkes-sql-backup

rm $tar_file
```

## Configuration of S3 Bucket

The S3 bucket is configured to expire files after 30 days.

## Lifecycle Policy

Delete Objects that are older than 30 days

<img src="aws_s3_lifecycle_policy_sql_ bucket.png" alt="Lifecycle policy"/>