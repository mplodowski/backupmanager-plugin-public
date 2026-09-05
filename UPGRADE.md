# Upgrade guide

Versions not listed here need no action. Back up the database before upgrading.

## Upgrading To 2.0.0

Plugin requires October CMS build 420+ with Laravel 5.5 and PHP >=7.0. Settings were reset to defaults, so review
them.

## Upgrading To 2.1.0

Dropbox support moved to the [Dropbox Adapter](https://octobercms.com/plugin/renatio-dropboxadapter) plugin.

## Upgrading To 2.1.4

The database port must be set in the database config.

## Upgrading To 4.0.0

Plugin requires October CMS build 467+ with Laravel 6.x and PHP >=7.2.

## Upgrading To 4.1.0

Plugin requires October CMS 1.x with Laravel 6.x and PHP >=7.3.

## Upgrading To 4.2.0

Plugin requires October CMS 2.x.

## Upgrading To 5.0.0

Plugin requires October CMS 3.0 or higher, Laravel 9.0 or higher and PHP >=8.0.

## Upgrading To 5.1.0

Mail notification settings changed. Configure them again on the Notifications tab if you use them.

## Upgrading To 6.1.0

Plugin requires PHP 8.3 or higher. `spatie/laravel-backup` was updated to 10.x, so run `composer update`.

## Upgrading To 6.6.0

Run `php artisan october:migrate`.

## Upgrading To 6.6.1

A malformed notification e-mail address is now refused when saving the settings.

## Upgrading To 6.7.0

Run `php artisan october:migrate`. The **Backup log** popup was replaced by the History page; its retention is set on
the Cleanup tab.

## Upgrading To 7.0.0

**The plugin permissions were replaced and nothing is migrated. Every role loses access to Backup Manager until you
grant the new permissions by hand.** Go to **Settings → Role Permissions** and tick what each role needs on the
**Backups** tab; tick everything to restore what a role could do before. Super users are unaffected.

## Upgrading To 7.2.0

The external trigger token is now created when the settings are saved. If you enabled **Allow external trigger** in
7.1.0, open the plugin settings, save them once and point your external cron service at the new addresses.

## Upgrading To 7.3.0

The safety backup taken by `backup:restore` is encrypted only with `--password` (or `BACKUP_RESTORE_PASSWORD`);
  without it the safety backup is stored unencrypted.

The backup lock and queue deduplication need a cache store shared across processes. If `CACHE_STORE` is `array`,
  switch to `file`, `database` or `redis`.

The backups page warns when the file backup includes `.env` without an archive password. Set a password in
  **Settings > Security** or exclude `.env`.
