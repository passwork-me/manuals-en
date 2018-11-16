# How to enable authorization in MongoDB

By default, the self-hosted version of Passwork does not use authorization to connect with the MongoDB. 
You can enable and configure authorization in MongoDB for increased security.

### Create a user in MongoDB
Open a command line and execute:   
```
mongo
```
 
Create a user with the commands:
```
use admin;
db.createUser(
{
	user: "adminuser",
	pwd: "password",
   roles: [ 
   	{ role: "userAdminAnyDatabase", db: "admin" },
		{ role: "dbOwner", db: "admin" }, 
		{ role: "dbOwner", db: "pwbox" },
		{ role: "dbOwner", db: "pwbox-cache" } ]
})  

```

`user`— login
`pwd` — password
Do not use the following characters in user password `@`, `:`, `#` because this can lead to application failures.

### Enable authorization mode
#### Find the MongoDB configuration file
Depending on your operating system and database version, the configuration file may be located: 
* mongod.conf  
* mongod.cfg  
* mongodb.conf  

For Linux: this file is usually located in the directory `/etc/`
Fpr Windows: check MongoDB installation folder.
You can find the path to the configuration file with the following commands.

```
mongo
use admin
db.adminCommand( { getParameter : '*' } )
```

#### Enable authorization mode
Open the configuration file and add the line:  
```
security.authorization: enabled
```
Save the file and restart MongoDB.
  
### Configure Passwork to work with the new mode
Open `<пассворк>/app/config.ini`.     
Find `connectionString`:
```
[mongo]
connectionString = mongodb://localhost:27017
dbname = pwbox
```
and change to:  
  
```
connectionString = mongodb://adminuser:password@localhost:27017
dbname = pwbox
```
Save the file and refresh the page in the browser with Passwork.
    
## Troubleshooting

•	Passwork stopped working correctly after the described actions
•	the page displays the string `{"response":false}`  
  
Check the Passwork log files that are in `<passwork>/app/logs/`.    
If you see these lines:  
```
Failed to connect to: localhost:27017: Authentication failed on database 'admin' with username adminuser: auth failed
#0 C:\inetpub\wwwroot\app\config\services.php(484): MongoClient->__construct('mongodb://usera...')
```
The most likely problem is that you have installed the old PHP extension `mongo`. 
Open the browser page `http://your_passwork/check.php` - version `mongo` should be 1.6.

Try the following.
Open the MongoDB configuration file, comment out or delete the line:
```
security.authorization: enabled
```
Save the file and reload MongoDB.
Open the command line:
```
mongo
use admin
  
var schema = db.system.version.findOne({"_id" : "authSchema"});
schema.currentVersion = 3;
db.system.version.save(schema);
```
Then return the line:
```
security.authorization: enabled
```

Save the file and reload MongoDB.

Check if Passwork works appropriately.
## Technical support
If you still see problems send the following information to <info@passwork.me>:
* screenshot of the page `http://your_passwork/check.php`
* screenshot of the config file `<passwork>/app/config.ini`.
* last log file from `<passwork>/app/logs /`
* screenshot of MongoDB config file
* PHP log file
