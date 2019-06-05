# Emergency mode
If you can't remember or restore an authorization password of the owner user then you can reset owner's password in the emergency mode.
Find `<passwork>/app/config/config.ini` file and add the first line:

```
emergency = on
```

Save the file and navigate to https://your-passwork-host/emergency  
Click "Reset owner password". The new password will be displayed.  
Remove `emergency = on` from the config.ini file.  
