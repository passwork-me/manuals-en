# Passwork Update Manual

The new version of Passwork works on PHP 7.x version (version 5 is no longer supported) along with the new PHP extension `mongodb`.

General update scheme:

1. Install the new version of Passwork on a separate server
2. Transfer settings and database

This will allow you not to disable your current working version of Passwork.

## Preparation
Open http://your-passwork/check.php page and check the PHP version of the PhalconPHP extension.
If your version is lower than 3.0.0, then contact our technical support for details of the update <info@passwork.me>.

## Installing a new version of Passwork
Install Passwork with [instructions](../README.md).

## Data transfer
Create a database backup and transfer it to the new server.
Creating and deploying backups is described in [MongoDB Backup](../Backups.md).

Find the file `<passwork>/app/config/config.ini` and transfer it to the new server.

Make sure everything works correctly.
