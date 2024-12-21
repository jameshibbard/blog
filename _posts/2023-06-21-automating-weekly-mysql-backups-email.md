---
title: Automating Weekly MySQL Backups via Email
layout: post
permalink: /automating-weekly-mysql-backups-email/
excerpt_separator: <!--more-->
tags:
  - cron
  - mysql
  - bash
  - servers
  - email
  - backup
  - linux
  - shell scripting
image_shadow: true
---

Manually backing up databases can be a frustrating and time-consuming task.  It often involves logging into servers, managing credentials, transferring files, and keeping track of schedules to avoid missed backups.

In this post, I'll walk you through the steps I took to create an automated solution for weekly MySQL backups. It involves setting up a cron job to run the backup at regular intervals, then emailing it to myself for easy access.

<!--more-->

> In this guide I will be using Ubuntu Server 22.04 LTS, however it shouldn't be too hard to adapt these instructions for other operating systems.

## Configuring mysqldump for Password-Free Access

The first step is to programatically generate a backup of the MySQL database. We can use [mysqldump](https://dev.mysql.com/doc/refman/8.4/en/mysqldump.html) for this purpose.

To give it a try, log into your server and run:

```bash
mysqldump -u <db-user> -p <db-name> > test_export.sql
```

This will prompt you to enter your password, then will create a `test_export.sql` file containing the specified database, in the same directory you ran the command.

Obviously we are not going to enter the password every time, meaning we need to save it somewhere our script will be able to access it.

We could hardcode the password into the script we are about to create, but a more flexible solution is to use a `.my.cnf` file in our home directory. This file is a user-specific configuration file that allows MySQL tools to securely access credentials without requiring them to be entered manually or exposed in scripts.

You can check if this file exists on your system like so:

```bash
test -f ~/.my.cnf && echo "File exists" || echo "File does not exist"
```

Next, open the file in your editor. If the file doesn't exist, this will create it:

```bash
cd
nano .my.cnf
```

> The `cd` command navigates to the user's home directory. For the rest of this guide, I will assume we are working from there.

Now enter:

```text
[client]
user=your_username
password=your_password
```

Then press <kbd>Ctrl</kbd>+<kbd>O</kbd> to save the file, followed by <kbd>Enter</kbd>, and <kbd>Ctrl</kbd>+<kbd>X</kbd> to exit the editor.

Once created, ensure the file is secure by setting its permissions to allow only the owner to read it:

```bash
chmod 600 .my.cnf
```

MySQL tools will now automatically use these credentials when executed.

You can try it out to make sure it works. Enter:

```
mysql
```

This should land you in the MySQL command-line client without prompting for a username or password. If it doesn't work, double-check the `[client]` section in your `.my.cnf` file and ensure the file permissions are set correctly.

## Creating a MySQL Backup Script

Next we need to write a shell script to run the `mysqldump` command.

```bash
nano db-backup.sh
```

Then, adjusting the first two variables to suit your needs, enter:

```bash
#!/bin/bash

# Variables
DB_NAME="your_database_name"
BACKUP_DIR="/path/to/backup"
BACKUP_FILE="$BACKUP_DIR/backup_$(date +%Y%m%d%H%M%S).sql"

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Run mysqldump
mysqldump --defaults-file=~/.my.cnf "$DB_NAME" > "$BACKUP_FILE"

# Check if backup was successful
if [ $? -eq 0 ]; then
  echo "Backup successful: $BACKUP_FILE"
else
  echo "Backup failed"
  exit 1
fi
```

I have added comments to the script to show what each part does. Worthy of note however, is the check to determine whether the `mysqldump` command was successful by evaluating its exit status (`$?`). This ensures that we'll be notified if something goes wrong.

The backup is saved in the directory specified by `BACKUP_DIR`, which is created automatically if it doesn't exist. The script generates a timestamped filename for each backup, keeping them organized and easy to manage.

Save the script as `db-backup.sh` and make it executable:

```bash
chmod +x db-backup.sh
```

Then run it to make sure that it is working as expected:

```bash
./db-backup.sh
```

If everything has gone according to plan, navigate to where you are storing the files and run the `ls` command. You should see something like:

```text
jim@server:~/db$ ll
backup_20241221075046.sql
```
## Sending the Backup via Email

Once the backup is created, you can email it to yourself using mailx. With a little luck, this may already be installed and configured on your server.

You can test this by sending a simple email (swap out the email address with your own):

```bash
echo "This is a test email" | mailx -s "Test Email" your_email@example.com
```

If you receive the email, mailx is ready to use. If not, you'll need to install and configure it first.


<div class="summary">
  <button class="summary-toggle">Show me how…</button>
  <div class="summary-content" markdown="1">

## Setting Up Postfix to Work with Gmail and mailx

The following steps will guide you through installing and configuring [Postfix](https://www.postfix.org/) (a Mail Transfer Agent) to work seamlessly with mailx and Gmail. You will need a Gmail account to follow along.

### 1. Install Postfix and mailx
Begin by installing Postfix and mailutils (which includes mailx):

```bash
sudo apt update && sudo apt install mailutils
```

### 2. Select the Configuration Type
During installation, choose _Internet Site_ when prompted to set the mail server configuration type. When asked for a domain name, you can enter _localhost_ unless you have a proper domain you wish to use.

### 3. Edit the Postfix Configuration
Open the Postfix configuration file:

```bash
sudo nano /etc/postfix/main.cf
```

Add or modify the following lines:

```text
relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
```

Save and close the file.

### 4. Obtain an App Password from Gmail
To use Gmail with Postfix, you must generate an _app-specific password_. This requires enabling two-factor authentication on your Gmail account.

- Go to the [_Security_ section of your Google account](https://myaccount.google.com/security).
- Under _How you sign in to Google_, select _2-Step Verification_.
- At the bottom of the page, select _App Passwords_.
- Create a new password and note it down for the next step.

### 5. Create the SASL Password File
Create the file that will store your Gmail credentials:

```bash
sudo nano /etc/postfix/sasl_passwd
```

Add the following line, replacing the email address and password with your Gmail email and app password:

```plaintext
[smtp.gmail.com]:587 your_email@gmail.com:your_app_password
```

Secure the file and postmap it to create a hashed version:

```bash
sudo chmod 600 /etc/postfix/sasl_passwd
sudo postmap /etc/postfix/sasl_passwd
```

This converts the plain text file into a secure format that Postfix can use for lookups.

### 6. Restart Postfix
Restart the Postfix service to apply the changes:

```bash
sudo systemctl restart postfix
```

### 7. Test Email Sending
To test your configuration, send a test email. Replace the email address with the address you wish to send to:

```bash
echo "This is a test email" | mailx -s "Test Email" your_email@gmail.com
```

If everything is configured correctly, you should receive the email. If not, check the logs for debugging:

```bash
sudo tail -f /var/log/mail.log
```

This is a quick overview of setting up Postfix with Gmail. For a more detailed guide, refer to [this extensive tutorial on Dev.to](https://dev.to/chigozieco/configure-postfix-to-send-email-with-gmails-smtp-from-the-terminal-4cco).

  </div>
</div>

With mailx now working, we can update the backup script to include email functionality.

First, let's install [sharutils](https://www.gnu.org/software/sharutils/), which includes the `uuencode` utility used for encoding files for email attachments.

```bash
sudo apt update && sudo apt install sharutils
```

Here is the updated script:

```bash
#!/bin/bash

# Variables
DB_NAME="your_database_name"
BACKUP_DIR="/path/to/backup"
BACKUP_FILE="$BACKUP_DIR/backup_$(date +%Y%m%d%H%M%S).sql"
COMPRESSED_FILE="$BACKUP_FILE.gz"

EMAIL="your_email@example.com"
SUBJECT="Database Backup: $(date +%Y-%m-%d)"
BODY="Attached is the database backup."

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Run mysqldump
mysqldump --defaults-file=~/.my.cnf "$DB_NAME" > "$BACKUP_FILE"

# Check if backup was successful
if [ $? -eq 0 ]; then
  echo "Backup successful: $BACKUP_FILE"

  # Compress the backup
  gzip "$BACKUP_FILE"
  echo "Backup compressed: $COMPRESSED_FILE"

  # Send email with compressed backup attached
  echo "$BODY" | uuencode "$COMPRESSED_FILE" "$(basename "$COMPRESSED_FILE")" | mailx -s "$SUBJECT" "$EMAIL"
  echo "Email sent to $EMAIL"
else
  echo "Backup failed"
  exit 1
fi
```

As you can see we have made a couple of changes:

- The backup file is now compressed using `gzip` before being emailed. This reduces the file size, making it faster to send and easier to store.
- The script uses `uuencode` to encode the compressed backup and attach it to the email.
- Timestamped filenames are used for both the uncompressed and compressed backups.

If you run the script at this point, you should receive an email with a compressed backup attached.

## Automating with Cron

Finally, we want to automate the script to run once a week.

Open the crontab editor by running:

```bash
crontab -e
```

If it your first time running this, you might be asked to select an editor. Choose nano.

Then add the following line to schedule the script to run every Monday at midnight:

```text
0 0 * * 1 /path/to/db-backup.sh
```

Be sure to replace `/path/to/db-backup.sh` with the actual path, before saving the file and exiting the editor.

You can confirm the job is scheduled correctly by listing the crontab:

```bash
crontab -l
```

> If you want to run the conjob at a different time, or different frequency, you can consult this [cron expression generator](https://crontab.cronhub.io/).

## Conclusion

And that's a wrap!

In this guide, I've demonstrated how to automate MySQL backups by running a backup at a regular interval, and emailing the resultant backup files to an aribtary address.

By implementing these steps and scheduling the process with cron, you can maintain regular backups effortlessly and safeguard against data loss.

If you have any questions, or are stuck on a particular part of the tutorial, hit me up in the comments below and I'll do my best to help.
