```
sudo sed -i 's/^[[:space:]]*-[[:space:]]*10\.1\..*/            - 10.1.40.39\/24/; s/via: 10\.1\..*/via: 10.1.40.1/' /etc/netplan/50-cloud-init.yaml && sudo netplan apply
```
