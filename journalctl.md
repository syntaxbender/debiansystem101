# journalctl retention configuration

- rules
  - logs will store in disk not memory
  - compressed logs for better space management
  - disabled log signing for avoid cpu overhead
  - free system space will be always under 1gb
  - set retention 15days
  - enabled audit logs for file/folder access/permission errors

```
sudo nano /etc/systemd/journald.conf
```

```
[Journal]
Storage=persistet 
Compress=yes
Seal=no 
SystemKeepFree=1G  
MaxRetentionSec=15day
Audit=yes
```
