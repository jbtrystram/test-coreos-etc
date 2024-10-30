The idea is to deploy a coreOS VM with the following

```yaml
variant: fcos
version: 1.5.0
systemd:
  units:
    - name: my-confext-setup.service
      enabled: true
      contents: |
        [Unit]
        Description=System configuration import
        Documentation=https://github.com/jbtrystram/test-coreos-etc
        After=network-online.target
        Wants=network-online.target
        # This only needs to run once
        ConditionPathExists=!/var/lib/confexts/node-config
        # These services needs to write to /etc so order after as confext makes /etc RO
        After=coreos-ignition-write-issues.service console-login-helper-messages-gensnippet-ssh-keys.service
        Wants=coreos-ignition-write-issues.service console-login-helper-messages-gensnippet-ssh-keys.service
        [Service]
        Type=oneshot
        RemainAfterExit=yes
        ExecStart=/usr/bin/git clone https://github.com/jbtrystram/test-coreos-etc /var/lib/confexts/node-config
        ExecStart=systemd-confext merge
        [Install]
        WantedBy=multi-user.target
```

The machine can then be updated with 

```
cd /var/lib/confexts/node-config
git pull
systemd-confext refresh
```

The update could be done automatically with a systemd timer and servuce like these : 

```
[Unit]
Description=Update nnode configuration
Requires=network-online.target

[Service]
Type=oneshot
WorkingDirectory=/var/lib/confexts/node-config
# update the git repository
ExecStart=git pull
ExecStart=systemd-confext refresh
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```
And the accompagnying timer : 
```
[Unit]
Description=Trigger confext update

[Timer]
# Daily
OnCalendar=daily
# At each boot, after 2 minutes
OnBootSec=2min

[Install]
WantedBy=timers.target
```
