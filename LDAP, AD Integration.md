## LDAP and Active Directory Integration.md

## Description
When the integration is enabled, Passwork verifies the login and password of a user in LDAP / AD. If LDAP / AD allows you to connect, then Passwork authorizes the user.
A local account will also be created in Passwork.

When you log in for the first time, Passwork will notice that there is no local account yet and will redirect the user to the registration page.
On the registration page, you must enter your login and password from your LDAP / AD account.

*Important*

LDAP / AD does not apply to the owner of the organization. This user is always logged in with his local account.
This is necessary in order to always have the opportunity to log in as an administrator in case of a problem.

### 1. Find LDAP properties in a config file.
Find `<passwork folder>/app/config/config.ini` file and section [ldap].

### 2. Configure mask
Passwork uses LDAP to sign in users. When user enters his login and password and click `Sign in`, Passwork tries to connect (bind) to LDAP using his credentials. If LDAP accepts the request then the user gets signed in.

By default LDAP accepts quite complicated «login» string which is called DN (Distinguished Names). This login string is inconvenient to use as a regular user login. To solve this problem you can use `mask` field of the config file. Mask is a template for DN with a special token `<login>` which is replaced with real user login during signin process.

### 3. Mask for Windows Active Directory
Let's say your domain has a name `MyCompany` and you probably has a built-in Windows user `Administrator`. This way to can log in to Windows using account `MyCompany\Administrator` and its password.
So, set the mask:

```
mask = "MyCompany\<login>"
```

Then try to sign up under `Administrator` (not `administrator@mycompany` or `MyCompany\Administrator` — the domain is already specified in the mask)

### 4. Mask for Linux LDAP
LDAP DN looks like `"uid=login,ou=group,dc=domain"` but it's depend on how our LDAP is configured. 
Try to use the following mask:

```
mask = "uid=<login>,ou=users,dc=crowd"
```

Where users is a name of group with users and crowd is a name of your domain.

### 5. Final preparation 
When you enable LDAP most of Passwork users will be unable to log in because their logins probably don't match logins from LDAP.

Log in to Passwork under administrator role. Navigate to company settings page. Enable signup and «New users automatically get administrator role».

Then enable LDAP in config file.
And try to sign up under `administrator` (or any another LDAP login). 
If LDAP is properly configured then a new user will be created (with administrator role).

Later you can disable signup and `New users automatically get administrator role`.

