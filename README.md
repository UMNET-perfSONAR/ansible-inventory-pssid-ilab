# ansible-inventory-pssid-ilab

**Quick Start**:

**RPi4 OS install**:
Install Ubuntu 18 64 bit version:
https://ubuntu.com/download/raspberry-pi
Install on SD Card as per Ubuntu's instructions.

from console:
log on as ubuntu
change ubuntu password by hand


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

clean out .ssh/known_hosts
ssh to 154

# NOTE: add root password change to bootstrap.yml
# use a vault variable
```
ansible pSSID-testpoints \
  -u ubuntu --ask-pass --become-method sudo \
  -i ansible-inventory-pssid-ilab/inventory \
  -m ping
```

become root
clean out .ssh/known_hosts
ssh to 154

# proviion as ubuntu:
# -root password
# - ssh keys & accounts
```
ansible-playbook \
  -u ubuntu --ask-pass --become-method sudo \
  --ask-vault-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  ansible-inventory-pssid-ilab/playbooks/bootstrap.yml
```

Regular provisioning:

Use Ansible ping to verify connectivity to targets:

# ansible ping as default user instead of ubuntu
```
ansible all \
  --ask-become-pass \
  --ask-vault-pass \
  --become-method su \
  -i ansible-inventory-pssid-ilab/inventory \
  -m ping
```

# disable ubuntu account after we verify we can use our own account to become root
```
ansible-playbook \
  -u ubuntu --ask-become-pass --become-method sudo \
  --ask-vault-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  ansible-inventory-pssid-ilab/playbooks/security.yml
```

# intall perfSONAR 
# install pSSID
#FINAL: run network.yml to set dest network

Run the playbook:

```
ansible-playbook \
  --ask-pass \
  --ask-become-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  --limit pssid-elk.miserver.it.umich.edu \
  elastic.yml
```

```
ansible-playbook \
  --ask-pass \
  --ask-become-pass \
  -i ansible-inventory-pssid-ilab/inventory \
  elastic.yml
```

# Management Commands:

**Elastic Development**:

---

Bootstrap

```
git clone https://github.com/UMNET-perfSONAR/ansible-playbook-pSSID.git
cd ansible-playbook-pSSID
cd roles
git clone git@github.com:UMNET-perfSONAR/ansible-role-elastic.git
cd ..
git clone https://github.com/UMNET-perfSONAR/ansible-inventory-pssid-ilab.git
ansible-galaxy install -r requirements.yml --ignore-errors
```

Provision Elastic

```
ansible elastic \
  --ask-pass \
  --ask-become-pass \
  --become-method sudo \
  --inventory ansible-inventory-pssid-ilab/inventory \
  -m ping

ansible-playbook \
  --ask-pass \
  --ask-become-pass \
  --become-method sudo \
  --limit elastic, \
  --inventory ansible-inventory-pssid-ilab/inventory \
  pSSID.yml
```

Provision pSSID

```
ansible pSSID-testpoints \
  --ask-vault-pass \
  --ask-become-pass \
  --become-method su \
  --inventory ansible-inventory-pssid-ilab/inventory \
  -m ping

ansible-playbook \
  --ask-become-pass \
  --ask-vault-pass \
  --become-method su \
  --limit ansible pSSID-testpoints, \
  --inventory ansible-inventory-pssid-ilab/inventory \
  pSSID.yml
```
