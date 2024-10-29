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
        [Service]
        Type=oneshot
        RemainAfterExit=yes
        ExecStart=/usr/bin/git clone https://github.com/jbtrystram/test-coreos-etc /var/lib/confexts/node-config
        [Install]
        WantedBy=multi-user.target

```

The machine could then be updated with 

```
cd /var/lib/confexts/node-config
git pull
systemctl restart systemd-confext.service
```
