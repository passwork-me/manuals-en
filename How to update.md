# How to update Passwork

### Repository
The repository is located at `https://passwork.download/`

### Preparing for the update
Make a copy of the folder with the Passwork files.  
In case of any problems, you can roll back to your version.  
Files are located in the directory:  
For Windows: `С:\inetpub\wwwroot\`   
For Linux: `/var/www/` 

### Updating with Git
_This method is suitable if Passwork was installed using the command `git clone`_

Open a command prompt in the Passwork folder.  
Run the command:  
```
git pull
```
The system will automatically upload the changes.

### Re-download all files Passwork
_If you previously installed Passwork without using the git utility (downloaded the archive and unpacked it)._   
Rename the folder with Passwork, and create a new empty folder with the same name.

#### 1. Select the installation option:
#### a) using the Git utility (recommended)
Make sure you have git installed.  
Go to the created empty folder and clone the repository:   
```
git clone http://passwork.download/passwork/passwork.git .
```

#### b) through downloading a zip archive  
Authorize through the browser in the repository: <https://passwork.download/>.  
Go to the project: <https://passwork.download/passwork/passwork>  
Click on the "Download as archive" button.
The latest stable version of the project will be automatically downloaded.   
Unpack the archive into the created empty folder
  
#### 2. Transfer the license keys:    
Copy the files from the folder  `<passwork>/app/keys/` from the old version to the new one.  
Thus, in the folder `<passwork>/app/keys/` there will be files with licenses `(.lic)` and one file `reginfo.php` (or `reginfo.json`) 
  
#### 3. Transfer the file with the settings   
Copy the file `<passwork>/app/config/config.ini` from the old version to the new one.  
Compare the `<passwrok>/app/config/config.ini` with the template config `<passwork>/app/config/config.example.ini`. Perhaps the new version has additional settings.

### For Windows + IIS user
IIS can reset Rewrite rules (if you delete and load .htaccess file). Check your rewrite rules. If they are empty then repeat importing the rule like it's described in the manual https://github.com/passwork-me/manuals-en/blob/master/WindowsServer-2016.md

### Make sure that Passwork works correctly
In case of problems: return the old folder with files to roll back to the previous version and contact our technical support at <info@passwork.me>
   
