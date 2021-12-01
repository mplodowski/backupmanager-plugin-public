# Backup Manager Plugin

**Demo URL:** https://october-demo.renatio.com/backend/backend/auth/signin  
**Login:** backup  
**Password:** backup  

Backup your whole October application with ease.

> This plugin is fully compatible with the latest version of OctoberCMS 2.x from version 4.2.0.

> The latest version that supports OctoberCMS 1.x is version 4.1.0.

The backup is a zip file that contains all files in the directories you specify along with a dump of your database. The backup can be stored on any of the filesystems you have configured in October. You can backup your application to multiple filesystems at once. In addition to making the backup, the plugin can also clean up old backups and monitor health of all backups.

![OctoberCMS Backup Manager](https://octobercms.com/storage/app/uploads/public/615/041/654/615041654ed37598938006.png)

## Features
* A backup database and application files with mouse click
* FTP, Amazon S3, Rackspace, Dropbox Cloud storage support
* MySQL, PostgreSQL, SQLite and Mongo databases support
* A configurable scheduler for automatic backups
* Automatic cleanup old backups
* Extensive settings options
* Encryption and password protection
* Monitoring the health of all backups
* Mail notifications

## Requirements

This plugin requires PHP 7.3, with the [ZIP module](http://php.net/manual/en/book.zip.php) and Laravel 6.x. It's not compatible with Windows servers.

The package needs free disk space where it can create backups. Ensure that you have **at least** as much free space as the total size of the files you want to backup.

Make sure `mysqldump` is installed on your system if you want to backup MySQL databases.

Make sure `pg_dump` is installed on your system if you want to backup PostgreSQL databases.

Make sure `mongodump` is installed on your system if you want to backup Mongo databases.

## Why is this a paid plugin?

Something that is free has little or no perceived value. Users do not commit to free products and only use them until something else looks nice and is free comes along. When I invest my time in the development of a new plugin I commit to supporting and maintaining it. I ask my customers to do the same. I do not make money from this plugin by advertisements, upgrades or additional services like hosting or setup.

Did you know that 30% of your purchase or donation goes to help fund the October Project?

My plugins take many hours to develop (40-120+) and even more hours to document and maintain. My paid plugins have to pay for both this time, and the time I am spending on free plugins and less successful paid plugins. This means that it will take even a successful plugin years to become profitable. Please consider buying an extended license if you want me to continue to maintain these plugins for the very small fee I ask in return or hire me for adding functionality that you feel is missing but valuable.

## Like this plugin?

If you like this plugin, give this plugin a Like or Make donation with [PayPal](https://www.paypal.me/mplodowski).

## My other plugins

Please check my other [plugins](https://octobercms.com/author/Renatio).

## Support

Please use [GitHub Issues Page](https://github.com/mplodowski/backupmanager-plugin-public/issues) to report any issues with plugin.

> Reviews should not be used for getting support or reporting bugs, if you need support please use the Plugin support link.

Icon made by [Darius Dan](https://www.flaticon.com/authors/darius-dan) from [www.flaticon.com](https://www.flaticon.com/).
