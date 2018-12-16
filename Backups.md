# MongoDB Backup

### Preparation
You can create database backups using the `mongodump` utility.
And restore - using the utility `mongorestore`.

Both utilities come with a MongoDB database.

**Windows**

Find the folder where the database is installed (usually it is `C:\Program Files\MongoDB\bin\`).

**Linux**

These utilities are available from the command line after the database is installed.

Use a command line to work with the utilities.

### Creating backup
Create an empty directory and open it in the command line.
Run:

```
mongodump
```

The utility will connect to the local database, create a `dump` folder and save dumps of all databases.
Save the `dump` folder to your backup storage.

For more fine-tuning, see the detailed information about `mongodump` - <https://docs.mongodb.com/manual/reference/program/mongodump/>

### Recovery
Open a command line in the folder with the dump folder `dump`.
Execute

```
mongorestore dump
```

The utility will connect to the local database and restore all databases from backups.

For more fine-tuning, see the detailed information on `mongodump` - <https://docs.mongodb.com/manual/reference/program/mongorestore/>

