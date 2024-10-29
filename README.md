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
        [Service]
        Type=oneshot
        RemainAfterExit=yes
        ExecStart=/usr/bin/git clone https://github.com/jbtrystram/test-coreos-etc /var/lib/confexts/node-config
        ExecStart=systemd-confext merge
        [Install]
        WantedBy=multi-user.target
    # These services needs to write to /etc and confexts
    # make /etc RO
    - name: coreos-ignition-write-issues.service
      mask: true
    - name: console-login-helper-messages-gensnippet-ssh-keys.service
      mask: true
```

The machine could then be updated with 

```
cd /var/lib/confexts/node-config
git pull
systemctl restart systemd-confext.service
```
