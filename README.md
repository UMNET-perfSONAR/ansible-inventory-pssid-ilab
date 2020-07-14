# ansible-inventory-pssid-ilab

**Quick Start**:

Fron Console:

Install Ubuntu 18 64 bit version:

https://ubuntu.com/download/raspberry-pi

Install on SD Card as per Ubuntu's instructions.

log on as ubuntu
change ubuntu's password

```
sudo -s
passwd
```

From bastion:

Clone the playbook:

```
git clone https://github.com/UMNET-perfSONAR/ansible-playbook-pSSID.git
cd ansible-playbook-pSSID
```

Clone this inventory in the playbook dir:

```
git clone https://github.com/UMNET-perfSONAR/ansible-inventory-pssid-ilab.git
```

Get the required roles (ignore errors so we can run this multiple times):

```
ansible-galaxy install -r requirements.yml --ignore-errors
ansible-galaxy install \
  -r ansible-inventory-netbasilisk-perfsonar/requirements.yml\
  --ignore-errors
```

clean out .ssh/known_hosts
ssh to 154

```
ansible all \
  -u ubuntu --ask-pass --become-method sudo \
  -i ansible-inventory-pssid-ilab/inventory \
  -m ping
```

become root
clean out .ssh/known_hosts
ssh to 154

```
ansible-playbook \
  -u ubuntu --ask-pass --become-method sudo \
  -i ansible-inventory-pssid-ilab/inventory \
  ansible-inventory-pssid-ilab/playbooks/bootstrap.yml
```

```
ansible-playbook \
  --user epcjr \
  --ask-pass \
  --ask-become-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  ansible-inventory-pssid-ilab/playbooks/bootstrap.yml
```

Regular provisioning:

Use Ansible ping to verify connectivity to targets:

```
ansible all \
  --ask-become-pass \
  --become-method su \
  -i ansible-inventory-pssid-ilab/inventory \
  -m ping
```

Run the playbook:

```
ansible-playbook \
  --ask-pass \
  --ask-become-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  elastic.yml
```

# Management Commands:

Display auth interfaces on Archivers:

```
ansible ps-archives \
  --ask-pass --ask-become-pass \
  --become-method su \
  -i ansible-inventory-netbasilisk-perfsonar/inventory \
  -a "/usr/sbin/esmond_manage list_user_ip_address"
```

Delete an auth interface on Archivers:

```
ansible ps-archives \
  --ask-pass --ask-become-pass \
  --become-method su \
  -i ansible-inventory-netbasilisk-perfsonar/inventory \
  -a "/usr/sbin/esmond_manage delete_user_ip_address USERNAME IPADDR"
```

---
