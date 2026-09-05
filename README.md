# Backup Manager Plugin

Automate database and file backups for [October CMS](https://octobercms.com) to off-site storage — encrypted and
restorable.

**Demo URL:** https://october-demo.renatio.com/backend/backend/auth/signin  
**Login:** backup  
**Password:** backup

The backup is a zip file with the directories you specify and a dump of your database, stored on any of the
filesystems configured in October. The plugin cleans up old backups, monitors their health, notifies you when
something goes wrong and restores the database from any backup it created.

## Features

* A backup database and application files with mouse click
* Restore the database from any backup with a console command
* S3, FTP, SFTP, Dropbox storage support
* MySQL, PostgreSQL, SQLite and Mongo databases support
* A configurable scheduler for automatic backups
* Backups triggered over HTTP, for hosting without a system cron
* Queued backups that survive execution time limits
* A history of every backup run, with filters, bulk delete and automatic retention
* Per-action permissions
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

`mysqldump`, `pg_dump` or `mongodump` must be installed for the database engine you back up.

Backup runs are serialized under a cache lock, which needs a cache store shared across processes: `file`, `database`
or `redis`. With the `array` store backups still run, but may overlap.

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

The plugin registers a **Backups** menu in the backend.

* **Application backup** creates a backup of all project files and the database. By default the project **base path**
  is backed up without **vendor** and **node_modules**; change that in the plugin **Settings**.
* **Database backup** creates a database-only backup.
* **Clean old backups** removes old backups according to the Cleanup settings.

## Permissions

Access is granted per action under **Settings → Administrators**, on the **Backups** tab. Super users bypass all of
them.

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

> **Copy restore command** only hides the button. Restoring runs from the console, so anybody with shell access can
> restore regardless of this permission.

## Restoring a backup

Restoring replaces your database with the one stored inside a backup archive. It is available **from the console
only**, because it cannot be undone. The terminal button in the backups list copies the matching command to your
clipboard.

```bash
php artisan backup:restore
php artisan backup:restore --path="October CMS/2026-01-15-09-20-58.zip" --disk=local
```

| Option | Description |
| --- | --- |
| `--path` | Path to the backup archive on the disk. Required when running non-interactively. |
| `--disk` | Disk the backup is stored on. Defaults to the disk selected in the backups list. |
| `--connection` | Database connection to restore into. Defaults to the application default. |
| `--password` | Password for an encrypted archive. Falls back to `BACKUP_RESTORE_PASSWORD`, then to the password saved in the settings; an interactive run asks when none fits. Also encrypts the safety backup. |
| `--skip-safety-backup` | Do not create a safety backup first. Leaves you without a way back. |
| `--force` | Skip the confirmation prompt. Required for unattended runs. |

> **Only the database is restored.** Project files inside the archive are never touched.

> **You will be logged out.** The dump contains `backend_users` and `sessions`, so accounts return to the state they
> were in when the backup was taken.

> **Restoring is not transactional.** An import that fails halfway leaves the database inconsistent, which is why a
> safety backup is taken first - keep the path it prints.

A few things to know:

* Anything passed as `--password` shows up in `ps` and the shell history; on a shared server use
  `BACKUP_RESTORE_PASSWORD` or let the command ask.
* The safety backup is encrypted only with `--password`, never with the password from the settings, because that one
  lives in the database being replaced. Without `--password` it is stored in plain form - delete it once you have
  verified the restore.
* The restore waits for no other backup: if one is running, it stops before touching the database.
* Tables not present in the dump are left untouched.
* When the archive holds several dumps, the one matching `--connection` is used, by database name first and connection
  name second. If none matches, the restore stops and lists what the archive contains.
* PostgreSQL and SQLite dumps contain no drop statements, so they can only be restored into an empty database.
* The native client - `mysql`, `psql` or `sqlite3` - must be in `PATH`, and `proc_open` enabled.

### If the application no longer boots

A safety backup taken without `--password` can be imported without this application:

```bash
unzip -p "storage/app/October CMS/safety-2026-01-15-09-20-58.zip" "db-dumps/*.sql" > /tmp/recover.sql
mysql -u USER -p DATABASE < /tmp/recover.sql
```

## Maximum execution time error

A **Maximum execution time of ... seconds exceeded** error means the backup is too large to finish in a single web
request. You can:

1. Raise **max_execution_time** in your PHP configuration.
2. Use the [Scheduler](#scheduler) (recommended).
3. Use [Console commands](#console-commands).
4. Enable [Queued backups](#queued-backups).

## Queued backups

Enabling **Queue backups** on the Destination tab runs backend backups through the queue, so the request returns
immediately and the backup continues in the background. This requires a running queue worker:

```bash
php artisan queue:work
```

Only a flash message is shown after clicking; open the **History** page to see the output once the job completed.

> Leave this setting off if you do not run a queue worker, otherwise backups started from the backend will never run.

## Backup history

The **History** page records every backup and cleanup run: when it happened, how long it took, how large the archive
was and whether it succeeded. Clicking an entry shows the output of that run.

The history is a journal of what the plugin did, not a list of the archives that exist - the backups list always reads
the disk. Removing an entry, selected entries or the whole history never touches the archives. Entries expire according
to **Keep history entries for days** in the settings.

## Settings

Go to **Settings** and open **Backup Manager** under the **Backup** section.

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

Configure how often the plugin runs the database backup, application backup, cleanup and health monitor.

> **Important note:** For scheduled tasks to operate correctly you must set up the
> scheduler: https://docs.octobercms.com/4.x/setup/scheduler.html

#### Allow external trigger

For hosting without a system cron, **Allow external trigger** publishes three addresses that run the tasks over HTTP:

```
https://example.com/backupmanager/trigger/<token>/db
https://example.com/backupmanager/trigger/<token>/app
https://example.com/backupmanager/trigger/<token>/clean
```

Point an external cron service (cron-job.org, EasyCron, an uptime monitor) at the one you need. A `200` means the
task ran, a `500` means it failed.

> **The address is the password.** Anybody who knows it can run the task, so keep it out of tickets, screenshots and
> anything else that gets shared. If it leaked, **Generate new addresses** replaces the token and every old address
> stops working at once. Switching the trigger off disables the addresses immediately.

Use HTTPS only - the token travels in the URL and ends up in server, proxy and cron service logs. When the calling
service can send headers, call `https://example.com/backupmanager/trigger/db` (or `app`, `clean`) with an
`X-Backup-Token` header instead.

The endpoint answers `404` to a wrong token, an unknown task and a disabled trigger alike, and limits requests to five
per minute per IP address and ten per minute across all addresses combined. With **Queue backups** enabled it returns
as soon as the task is queued, and only one job per task waits in the queue at a time.

### Security

#### Password

Password-protects the archive. Use a long string and keep it safe - **without it, you will never be able to open your
backup**. Leave it blank for an unprotected archive.

#### Secrets in the file backup

With the default settings the file backup contains the whole project, including `.env` with `APP_KEY`, database
credentials and storage keys. Without an archive password anyone who can read the backup can read the secrets.

Either set an archive password, or add `.env` to the **Exclude** list if your hosting recreates it. The backups page
warns as long as `.env` is backed up without a password.

#### Encryption

Test decrypting an archive once, so you know the restore path works before you need it.

**Warning:** the default macOS archive utility cannot open encrypted ZIP files. Use
[The Unarchiver](https://theunarchiver.com/) or [BetterZip](https://macitbetter.com/).

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

A backup is considered unhealthy when the latest backup is too old or the backups use more storage than allowed.

| Property                                          | Description                                                                                                                                                                                                  |
|---------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Newest backups should not be older than days**  | Send a notification when newest backup will be older than given days. Default to 1 day.                                                                                                                      |
| **Storage used may not be higher than megabytes** | Send a notification when storage used for backups will be higher than given megabytes. Default to 5000 megabytes. Setting to 0 means the monitor will consider that the backup can use unlimited storage.    |

### Dumping the database

If `mysqldump` / `pg_dump` is not in a default location, or the dump exceeds the default 60 second timeout, configure
the connection in `config/database.php`:

```
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

## Configuring the backup disk

By default backups are saved to `storage/app/October CMS/`. Create a dedicated disk (for example `backups`) in
`config/filesystems.php` and select it in the plugin **Settings**.

## Filesystems

Supported storage drivers: local, S3, FTP, SFTP and Dropbox. Configure the disk in `config/filesystems.php`, then
select it on the **Destination** tab of the plugin **Settings**.

### S3

```
composer require league/flysystem-aws-s3-v3 "^3.0"
```

A sample `s3` disk is included in `config/filesystems.php`.

### FTP

```
composer require league/flysystem-ftp "^3.0"
```

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

### SFTP

```
composer require league/flysystem-sftp-v3 "^3.0"
```

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

### Dropbox

Install the [Dropbox Adapter](https://octobercms.com/plugin/renatio-dropboxadapter) plugin and configure the disk as
described in its documentation.

## Console commands

* `backup:run` - run a backup; `--only-db` backs up only the database
* `backup:clean` - clean old backups
* `backup:list` - display the status of all monitored destination filesystems
* `backup:monitor` - check the health of all monitored destination filesystems
* `backup:restore` - restore the database from a backup archive, see [Restoring a backup](#restoring-a-backup)

## Notification channels

Notifications go out by e-mail, to Discord and to any service accepting a webhook. A channel is used once it has an
address; leave a field empty to disable it. Configure them on the **Notifications** tab of the plugin **Settings**.

| Setting | Description |
| --- | --- |
| **Notification email** | Addresses that receive notifications, separated with a comma. |
| **Discord webhook URL** | Create it in your Discord server under Integrations. |
| **Webhook URL** | Notifications are posted as JSON. Works with Slack, Mattermost, Microsoft Teams and custom integrations. |

A notification is sent when a backup or cleanup succeeds or fails, and when the monitor finds a healthy or unhealthy
backup.
