# How to send mails via SMTP

By default, the self-hosted version of Passwork sends mails using the php mail()-function.
In case your host system has no mailserver configured, this will not work.  
Another use-case would be, if you are using docker containers.

### Enable SMTP
Open `app/config.ini`.     
Add the smtp configuration:
```
[smtp]
enable = On;
server = smtp.domain.com
username = smtp-username@domain.com
password = your-secret-smtp-password
encryption = tls
port = 587
```
The following values for `encryprion` are possible:

| Value | Description | 
| ---- | ---- |
|  | No encryption |
| tls |  Use tls encryption |
| ssl | Use ssl encryption |
