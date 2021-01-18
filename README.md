# ansible-inventory-pssid-ilab

**Quick Start**: 

**RPi4 OS install**:
Install Ubuntu 18 64 bit version:
https://ubuntu.com/download/raspberry-pi
Install on SD Card as per Ubuntu's instructions.

from console:
log on as ubuntu
change ubuntu password by hand

```
sudo -s
passwd
```


**pSSID Bootstrap**:

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
  -r ansible-inventory-pssid-ilab/requirements.yml\
  --ignore-errors
```


become root
clean out .ssh/known_hosts
ssh to [ip address]

**provision user accounts as ubuntu**
ssh keys & accounts:
```
ansible-playbook \
  -u ubuntu \
  --become \
  --become-method sudo \
  --become-user root \
  --ask-pass \
  --ask-become-pass \
  --ask-vault-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  ansible-inventory-pssid-ilab/playbooks/bootstrap.yml
```

**Regular provisioning:**

Use Ansible ping to verify connectivity to targets:

ansible ping as default user instead of ubuntu
```
ansible all \
  --become \
  --become-method su \
  --become-user root \
  --ask-become-pass \
  --ask-vault-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  -m ping
```

disable ubuntu account after we verify we can use our own account to become root:
```
ansible-playbook \
  --become \
  --become-method su \
  --become-user root \
  --ask-become-pass \
  --ask-vault-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  ansible-inventory-pssid-ilab/playbooks/security.yml
```

set up ifupdown: install ifupdown and bring up the interfaces
ip addresses could change after this step:
```
ansible-playbook \
  --become  \
  --become-method su \
  --become-user root \
  --ask-become-pass \
  --ask-vault-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  ansible-inventory-pssid-ilab/playbooks/ifupdown_setup.yml
```

disable networkd and create wpa_supplicant:
```
ansible-playbook \
  --become \
  --become-method su \
  --become-user root \
  --ask-become-pass \
  --ask-vault-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  ansible-inventory-pssid-ilab/playbooks/dhcp_network.yml
```

Install pSSID, pscheduler, rabbitmq:
intall perfSONAR 
install pSSID
```
ansible-playbook \
  --become \
  --become-method su \
  --become-user root \
  --ask-become-pass \
  --ask-vault-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  pSSID.yml
```

Create pssid_conf.json for each pi
for each pi, add directory in inventory/host_vars that holds config vars:
```
ansible-playbook \
  --become  \
  --become-method su \
  --become-user root \
  --ask-become-pass \
  --ask-vault-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  ansible-inventory-pssid-ilab/playbooks/create_pssid_conf.yml
```

boot script:
```
ansible-playbook \
  --become  \
  --become-method su \
  --become-user root \
  --ask-become-pass \
  --ask-vault-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  ansible-inventory-pssid-ilab/playbooks/boot_setup.yml
```

Extract identifiers from pi and save it locally:
```
ansible-playbook \
  --become  \
  --become-method su \
  --become-user root \
  --ask-become-pass \
  --ask-vault-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  ansible-inventory-pssid-ilab/playbooks/extract_identifiers.yml
```


# Elastic setup
follow instructions at (ansible-playbook-elastic)[https://github.com/UMNET-perfSONAR/ansible-playbook-elastic]
