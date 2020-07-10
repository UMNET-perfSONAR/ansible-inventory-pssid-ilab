# ansible-inventory-pssid-ilab

**Quick Start**:

Install Ubuntu 18 64 bit version:

https://ubuntu.com/download/raspberry-pi

Install on SD Card as per Ubuntu's instructions.

log on as ubuntu

'''
sudo -s
passwd
'''

Clone the playbook:

```
git clone https://github.com/perfsonar/ansible-playbook-perfsonar.git
cd ansible-playbook-perfsonar
```

Clone this inventory in the playbook dir:

```
git clone https://github.com/NetBASILISK/ansible-inventory-netbasilisk-perfsonar.git
```

Get the required roles (ignore errors so we can run this multiple times):

```
ansible-galaxy install -r requirements.yml --ignore-errors
ansible-galaxy install \
  -r ansible-inventory-netbasilisk-perfsonar/requirements.yml\
  --ignore-errors
```

Use Ansible ping to verify connectivity to targets:

```
ansible all \
  --ask-become-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  -m ping
```

Run the playbook:

```
ansible-playbook \
  --ask-become-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  pSSID.yml
```

Run the account provisioning playbook for the netbasilisk-perfsonar group:

```
ansible-playbook \
  --ask-pass --ask-become-pass \
  -i ansible-inventory-netbasilisk-perfsonar/inventory \
  ansible-inventory-netbasilisk-perfsonar/playbooks/miserver.yml
```

# Management Commands:

Display auth interfaces on Archivers:

```
ansible ps-archives \
  --ask-pass --ask-become-pass \
  -i ansible-inventory-netbasilisk-perfsonar/inventory \
  -a "/usr/sbin/esmond_manage list_user_ip_address"
```

Delete an auth interface on Archivers:

```
ansible ps-archives \
  --ask-pass --ask-become-pass \
  -i ansible-inventory-netbasilisk-perfsonar/inventory \
  -a "/usr/sbin/esmond_manage delete_user_ip_address USERNAME IPADDR"
```

---
