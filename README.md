# Backup Manager Plugin

**Demo URL:** https://october-demo.renatio.com/backend/backend/auth/signin

**Login:** backup

**Password:** backup

Backup your application with ease.

The backup is a zip file that contains all files in the directories you specify along with a dump of your database. The
backup can be stored on any of the filesystems you have configured in October. You can backup your application to
multiple filesystems at once. In addition to making the backup, the package can also clean up old backups, monitor the
health of the backups, and show an overview of all backups. The plugin can also notify you via mail, Discord or a
webhook when something goes wrong with your backups, and restore the database from any backup it created.

## Features

* A backup database and application files with mouse click
* Restore the database from any backup with a console command
* S3, FTP, SFTP, Dropbox storage support
* MySQL, PostgreSQL, SQLite and Mongo databases support
* A configurable scheduler for automatic backups
* Backups triggered over HTTP, for hosting without a system cron
* Queued backups that survive execution time limits
* A history of every backup run, with filters, bulk delete and automatic retention
* Per-action permissions, enforced on the server rather than only hidden in the interface
* Automatic cleanup of old backups
* Extensive settings options
* Encryption and password protection
* Verification of every created archive
* Monitoring the health of all backups
* Mail, Discord and webhook notifications

## Requirements

This plugin requires PHP 8.3, with the [ZIP module](http://php.net/manual/en/book.zip.php), October CMS 4.0 and Laravel
12.40 or higher. It's not compatible with Windows servers.

If you are using an older version of Laravel and October, take a look at one of the previous versions of this plugin.

The package needs free disk space where it can create backups. Ensure that you have **at least** as much free space as
the total size of the files you want to backup.

Make sure `mysqldump` is installed on your system if you want to backup MySQL databases.

Make sure `pg_dump` is installed on your system if you want to backup PostgreSQL databases.

Make sure `mongodump` is installed on your system if you want to backup Mongo databases.

## Why is this a paid plugin?

Something that is free has little or no perceived value. Users do not commit to free products and only use them until
something else looks nice and is free comes along. When I invest my time in the development of a new plugin I commit to
supporting and maintaining it. I ask my customers to do the same. I do not make money from this plugin by
advertisements, upgrades or additional services like hosting or setup.

Did you know that 30% of your purchase or donation goes to help fund the October Project?

My plugins take many hours to develop (40-120+) and even more hours to document and maintain. My paid plugins have to
pay for both this time, and the time I am spending on free plugins and less successful paid plugins. This means that it
will take even a successful plugin years to become profitable. Please consider buying an extended license if you want me
to continue to maintain these plugins for the very small fee I ask in return or hire me for adding functionality that
you feel is missing but valuable.

## Like this plugin?

If you like this plugin, give this plugin a Like or Make donation with [PayPal](https://www.paypal.me/mplodowski).

## My other plugins

Please check my other [plugins](https://octobercms.com/author/Renatio).

## Support

Please use [GitHub Issues Page](https://github.com/mplodowski/backupmanager-plugin-public/issues) to report any issues
with plugin.

> Reviews should not be used for getting support or reporting bugs, if you need support please use the Plugin support
> link.

Icon made by [Darius Dan](https://www.flaticon.com/authors/darius-dan)
from [www.flaticon.com](https://www.flaticon.com/).

# Documentation

## Usage

After installation plugin will register backend **Backups** menu position.

**Application backup** button will create backup with all project files and database dump file. To change which files
will be included see plugin **Settings**. Default settings will take project **base path** and exclude **vendor** and **node_modules** directories from it.

**Database backup** button will create only database backup.

**Clean old backups** button will clean up old backups based on the plugin **Settings**.

**Settings** button will redirect you to the plugin settings.

## Permissions

Access is granted per action, so an administrator can be allowed to run a backup without being able to delete one, or
to read the history without being able to clear it. Assign them under **Settings → Administrators** on the **Backups**
tab.

| Permission | What it unlocks |
| --- | --- |
| **Manage backups** | The backups page itself. Required for everything else on that page. |
| **Create backups** | The *Application backup* and *Database backup* buttons. |
| **Clean old backups** | The *Clean old backups* button. |
| **Delete backups** | The delete icon on each row, the checkboxes and *Delete selected*. |
| **Download backups** | The download icon, and the download route behind it. |
| **Copy restore command** | The terminal icon that copies the restore command. |
| **Manage history** | The History page itself. |
| **Delete history entries** | The delete icon on each row, the checkboxes, *Delete selected* and *Clear history*. |
| **Manage settings** | The plugin settings, and the *Settings* button in the toolbar. |

Every action is checked on the server, not only hidden in the interface, so a hand-written request cannot get around a
missing permission. Administrators marked as super users bypass all of these, as everywhere else in October.

> **Copy restore command** only hides the button. Restoring runs from the console, so anybody with shell access can
> restore regardless of what this permission says.

## Restoring a backup

Restoring replaces your database with the one stored inside a backup archive. It is available **from the console only**,
because it is destructive and cannot be undone. In the backups list, the orange terminal button copies the matching
restore command to your clipboard so you can paste it into a terminal.

```bash
php artisan backup:restore
php artisan backup:restore --path="October CMS/2026-01-15-09-20-58.zip" --disk=local
```

| Option | Description |
| --- | --- |
| `--path` | Path to the backup archive on the disk. Required when running non-interactively. |
| `--disk` | Disk the backup is stored on. Defaults to the disk selected in the backups list. |
| `--connection` | Database connection to restore into. Defaults to the application default. |
| `--password` | Password for an encrypted archive. Defaults to the one saved in the settings. |
| `--skip-safety-backup` | Do not create a safety backup first. Leaves you without a way back. |
| `--force` | Skip the confirmation prompt. Required for unattended runs. |

> **Only the database is restored.** Project files inside the archive are never touched. If you need them back, unpack
> the archive manually.

Along with the safety backup path, the command prints two ways to roll back: the `backup:restore` command and a raw
database client pipeline (`unzip -p ... | mysql ...`) for the case where the application is so broken that artisan
itself no longer runs. The pipeline never contains the database password; the client asks for it.

> **You will be logged out.** The dump contains `backend_users` and `sessions`, so accounts return to the state they
> were in when the backup was taken. Any account created afterwards will be gone.

> **Restoring is not transactional.** Schema changes in MySQL cannot be rolled back, so an import that fails halfway
> leaves the database in an inconsistent state. This is why a safety backup is taken by default - keep the path it
> prints.

Tables that are not part of the dump, such as tables excluded from the backup, are left untouched by a restore.

When the archive holds dumps of several databases, the one matching `--connection` is used. It is found by its database
name first, then by the connection name, so back up two databases at once and each can still be restored on its own. If
no single dump matches, the restore stops and lists what the archive contains instead of guessing.

### PostgreSQL and SQLite

Dumps for these engines are created without drop statements, so they can only be restored into an empty database.
Prepare one before restoring, for example by recreating the schema.

### Requirements

Restoring uses the native client of your database engine, the same way backups are created with `mysqldump`. Make sure
`mysql`, `psql` or `sqlite3` is installed and available in `PATH`, and that `proc_open` is enabled.

### If the application no longer boots

A safety backup is always stored unencrypted, so it can be imported without this application:

```bash
unzip -p "storage/app/October CMS/safety-2026-01-15-09-20-58.zip" "db-dumps/*.sql" > /tmp/recover.sql
mysql -u USER -p DATABASE < /tmp/recover.sql
```

## Maximum execution time error

When you experience **Maximum execution time of ... seconds exceeded** error than most likely application backup is too
large and PHP cannot do this process in a single request.

> What can I do?

1. Change **max_execution_time** property value in your PHP configuration.
2. Use [Scheduler](#scheduler) to perform automatic backups (recommended).
3. Use [Console commands](#console-commands) to perform backups.
4. Enable **Queue backups** in the settings, see [Queued backups](#queued-backups).

## Queued backups

Backups started from the backend normally run inside the web request, which means a large backup can hit the PHP
execution time limit. Enabling **Queue backups** on the Destination tab dispatches the work to the queue instead, so the
request returns immediately and the backup keeps running in the background.

This requires a running queue worker:

```bash
php artisan queue:work
```

With queueing enabled only a flash message is shown after clicking, because the backup runs in the background. Open
the **History** page once the job completed to see its output.

> Leave this setting off if you do not run a queue worker, otherwise backups started from the backend will never run.

Each queued backup reads the settings when it runs, so a change takes effect on the next backup without restarting the
worker.

## Backup history

The **History** page under the Backups menu records every backup and cleanup run: when it happened, how long it took,
how large the archive was and whether it succeeded. Clicking an entry shows the output of that particular run, so a
failure stays readable long after it happened.

The history is a journal of what the plugin did, not a list of the archives that exist. Backups removed straight from
the disk, or created on another server, leave no trace in it - the backups list itself always reads the disk, so it
stays accurate regardless.

Entries can be removed in three ways: the trash icon on a row removes that one entry, checking rows enables
**Delete selected** in the toolbar, and **Clear history** wipes the page clean. None of them touch the archives
themselves - removing an entry only forgets that the run happened. Entries also expire on their own, see
**Keep history entries for days** in the settings.

## Settings

This plugin ships with settings page. Go to **Settings**, and you will see a menu item **Backup Manager** listed
under **Backup** section.

### Database

| Property                   | Description                                                                                                                          |
|----------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| **Databases**              | The names of the connections to the databases that should be backed up. MySQL, PostgreSQL, SQLite and Mongo databases are supported. |
| **Exclude tables**         | Those tables will not be included in backup.                                                                                         |
| **Compress database dump** | The database dump can be compressed to decrease disk space usage.                                                                    |

### Files

| Property                          | Description                                                                                                                          |
|-----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| **Include**                       | The list of directories and files that will be included in the backup. Leave empty to backup whole October project.                  |
| **Exclude**                       | These directories and files will be excluded from the backup. Directories used by the backup process will automatically be excluded. |
| **Follow links**                  | Determines if symlinks should be followed.                                                                                           |
| **Ignore unreadable directories** | Determines if it should avoid unreadable folders.                                                                                    |
| **Verify backup**                 | Checks that the created archive can be opened, is not empty and contains the expected number of files. Adds extra time to each backup. |

### Destination

| Property            | Description                                         |
|---------------------|-----------------------------------------------------|
| **Filename prefix** | The filename prefix used for the backup zip file.   |
| **Name**            | The name of this application.                       |
| **Disks**           | The disk names on which the backups will be stored. |
| **Queue backups**   | Runs backups through the queue instead of the web request. |

### Scheduler

Configure how often plugin will run automatic tasks for database backup, application backup, clean old backups and
monitor the health of all backups actions.

> **Important note:** For scheduled tasks to operate correctly you must set up the
> scheduler: https://docs.octobercms.com/4.x/setup/scheduler.html

#### Allow external trigger

Shared hosting often has neither a system cron nor console access, which leaves the scheduler above with nothing to run
it. Switching **Allow external trigger** on publishes three addresses, one per task, that run it over HTTP instead:

```
https://example.com/backupmanager/trigger/<token>/db
https://example.com/backupmanager/trigger/<token>/app
https://example.com/backupmanager/trigger/<token>/clean
```

Point an external cron service at whichever one you need - cron-job.org, EasyCron, or any uptime monitor that fetches a
URL on a schedule. A `200` means the task ran, a `500` means it failed, so a monitoring service will tell you when
backups stop working.

> **The address is the password.** Anybody who knows it can run the task. Keep it out of tickets, commit messages,
> screenshots and anything else that gets shared. Switch the trigger off and the address stops working immediately,
> even though the token stays the same. If an address did leak, the **Generate new addresses** button below the list
> replaces the token, and every old address stops working at once.

Use the addresses over HTTPS only: the token travels in the URL, so on plain HTTP it would cross the network in
cleartext, and URLs also end up in web server, proxy and cron service logs. Treat those logs as sensitive.

The endpoint answers `404` to everything it refuses - a wrong token, an unknown task, or the trigger being switched off
all look identical from outside, so probing for it tells an attacker nothing. It accepts five requests per minute per
IP address and ten per minute across all addresses combined; anything beyond that gets `429`, which keeps whoever
finds the address from filling the disk with parallel backups.

With **Queue backups** enabled the request returns as soon as the task is queued rather than waiting for the backup to
finish, which keeps a large backup from hitting the timeout of whichever service is calling it. Either way the run is
recorded on the History page like any other.

### Security

Here you can specify password protection for backups. You will be asked to enter this password in order to unzip the
backup file.

#### Password

Remember to use long strings and to keep your password safe – **without it, you will never be able to open your backup**
.

Leave it blank if you want to keep your backup without a password.

#### Encryption

It's common to encrypt backups before storing them somewhere to prevent unauthorized access.

It's important to try this workflow and also to decrypt a backup archive, so you know that it works and you have a
working backup restore solution.

**Warning:** the default MacOS app to (un)archive ZIPs seems unable to open/extract encrypted ZIP files.
You should use an app like [The Unarchiver](https://theunarchiver.com/) or [BetterZip](https://macitbetter.com/).

### Cleanup

| Property                                                 | Description                                                                                             |
|----------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| **Keep all backups for days**                            | The number of days for which backups must be kept.                                                      |
| **Keep daily backups for days**                          | The number of days for which daily backups must be kept.                                                |
| **Keep weekly backups for weeks**                        | The number of weeks for which one weekly backup must be kept.                                           |
| **Keep monthly backups for months**                      | The number of months for which one monthly backup must be kept.                                         |
| **Keep yearly backups for years**                        | The number of years for which one yearly backup must be kept.                                           |
| **Delete oldest backups when using more megabytes than** | After cleaning up the backups remove the oldest backup until this amount of megabytes has been reached. |
| **Keep history entries for days**                        | History entries older than this number of days are removed daily by the scheduler. Set to 0 to keep the history forever. |

### Monitoring

A backup is considered unhealthy if the date of the latest backup is too far in the past to be useful or if the amount
of storage space required for all backups is not available.

| Property                                          | Description                                                                                                                                                                                                  |
|---------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Newest backups should not be older than days**  | Send a notification when newest backup will be older than given days. Default to 1 day.                                                                                                                      |
| **Storage used may not be higher than megabytes** | Send a notification when storage used for backups will be higher than given megabytes. Default to 5000 megabytes. Setting to 0 means the monitor will consider that the backup can use unlimited storage.    |

### Dumping the database

The `mysqldump`/`pg_dump` are used to dump the database. If they are not installed in a default location, you can add a
key named `dump.dump_binary_path` in October `database.php` config file. Only fill in the path to the binary. Do not
include the name of the binary itself.

If your database dump takes a long time, you might exceed the default timeout of 60 seconds. You can set a higher (or
lower) limit by providing a `dump.timeout` config key which specifies, in seconds, how long the command may run.

Here's an example for MySQL:

```
// config/database.php
'connections' => [
	'mysql' => [
		'driver' => 'mysql'
		...,
		'dump' => [
		   'dump_binary_path' => '/path/to/the/binary', // only the path, so without `mysqldump` or `pg_dump`
		   'use_single_transaction',
		   'timeout' => 60 * 5, // 5 minute timeout
		   'exclude_tables' => ['table1', 'table2'],
		   'add_extra_option' => '--optionname=optionvalue',
		]
	],
```

There is also a setting in backend area to exclude database tables from backup. By default, it will exclude logs tables.

## Configuring the backup disk

By default, the backup will be saved into the storage/app/October CMS/ directory of your October application. This
folder most probably is configured to be public. We recommend that you create a disk named backups (you can use any name
you prefer) in filesystems.php and select this disk in plugin Settings.

## Filesystems

Plugin supports following storage drivers:

* Local Storage
* S3 Cloud Storage
* FTP Storage
* SFTP Storage
* Dropbox Cloud Storage

More drivers can be added on feature requests. Just create an issue with **[Feature Request]** in title, and I will see
what can be done.

The filesystem configuration file is located at **config/filesystems.php**. Within this file you may configure all of
your "disks". Example configurations for each supported driver is included in the configuration file. So, simply modify
the configuration to reflect your storage preferences and credentials!

### S3 Driver Configuration

Before using the S3 driver, you will need to install the Flysystem S3 package via the Composer package manager:

```
composer require league/flysystem-aws-s3-v3 "^3.0"
```

The S3 driver configuration information is located in your `config/filesystems.php` configuration file.

After filesystem configuration go to plugin **Settings** and check `s3` disk in **Destination** tab.

### FTP Driver Configuration

Before using the FTP driver, you will need to install the Flysystem FTP package via the Composer package manager:

```
composer require league/flysystem-ftp "^3.0"
```

Laravel's Flysystem integrations work great with FTP; however, a sample configuration is not included with the framework's default `filesystems.php` configuration file. If you need to configure an FTP filesystem, you may use the configuration example below:

```
'ftp' => [
    'driver' => 'ftp',
    'host' => env('FTP_HOST'),
    'username' => env('FTP_USERNAME'),
    'password' => env('FTP_PASSWORD'),

    // Optional FTP Settings...
    // 'port' => env('FTP_PORT', 21),
    // 'root' => env('FTP_ROOT'),
    // 'passive' => true,
    // 'ssl' => true,
    // 'timeout' => 30,
],
```

After filesystem configuration go to plugin **Settings** and check `ftp` disk in **Destination** tab.

### SFTP Driver Configuration

Before using the SFTP driver, you will need to install the Flysystem SFTP package via the Composer package manager:

```
composer require league/flysystem-sftp-v3 "^3.0"
```

Laravel's Flysystem integrations work great with SFTP; however, a sample configuration is not included with the framework's default `filesystems.php` configuration file. If you need to configure an SFTP filesystem, you may use the configuration example below:

```
'sftp' => [
    'driver' => 'sftp',
    'host' => env('SFTP_HOST'),

    // Settings for basic authentication...
    'username' => env('SFTP_USERNAME'),
    'password' => env('SFTP_PASSWORD'),

    // Settings for SSH key based authentication with encryption password...
    'privateKey' => env('SFTP_PRIVATE_KEY'),
    'password' => env('SFTP_PASSWORD'),

    // Optional SFTP Settings...
    // 'hostFingerprint' => env('SFTP_HOST_FINGERPRINT'),
    // 'maxTries' => 4,
    // 'passphrase' => env('SFTP_PASSPHRASE'),
    // 'port' => env('SFTP_PORT', 22),
    // 'root' => env('SFTP_ROOT', ''),
    // 'timeout' => 30,
    // 'useAgent' => true,
],
```

After filesystem configuration go to plugin **Settings** and check `sftp` disk in **Destination** tab.

### Dropbox configuration

Install [Dropbox Adapter](https://octobercms.com/plugin/renatio-dropboxadapter) plugin.

Read this external plugin documentation and configure Dropbox filesystem.

After filesystem configuration go to plugin **Settings** and check `dropbox` disk in **Destination** tab.

## Console commands

Plugin will create five new artisan commands for working with a console.

**backup:run** command will run new backup process. Add **--only-db** option for backup only database.

**backup:clean** command will run clean old backups process.

**backup:list** command will display status of all monitored destination filesystems.

**backup:monitor** command will check status of all monitored destination filesystems.

**backup:restore** command will restore the database from a backup archive. See [Restoring a backup](#restoring-a-backup).

## Notification channels

Notifications can be delivered by e-mail, to Discord, or to any service that accepts a webhook. Each channel is used
only once it has an address configured, so leaving a field empty simply disables that channel. When several are
configured, every enabled notification is sent through all of them.

| Setting | Description |
| --- | --- |
| **Notification email** | E-mail address that will receive notifications. Separate multiple addresses with a comma. Every address is validated when you save, because a malformed one would stop backups from running. |
| **Discord webhook URL** | Create it in your Discord server under Integrations. |
| **Webhook URL** | Notifications are posted as JSON. Works with Slack, Mattermost, Microsoft Teams and custom integrations. |

> Slack accepts incoming webhooks, so use the **Webhook URL** field to post backup notifications to a Slack channel.

A notification is sent when:

- backup was successful,
- backup has failed,
- cleanup was successful,
- cleanup has failed,
- healthy backup was found,
- unhealthy backup was found.

To configure this feature go to plugin **Settings** and **Notifications** tab.
