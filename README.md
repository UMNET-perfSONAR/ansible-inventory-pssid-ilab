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
This password may be needed when running ansible commands

# Elastic setup
follow instructions at [ansible-playbook-elastic](https://github.com/UMNET-perfSONAR/ansible-playbook-elastic)


**pSSID Bootstrap**:

1. Connect to VPN and ssh into bastion

From bastion:
2. Clone the playbook:

```
git clone https://github.com/UMNET-perfSONAR/ansible-playbook-pSSID.git
cd ansible-playbook-pSSID
```
All commands can be run from the ansible-playbook-pSSID directory. Ansible commands must be run from this directory

3. Clone this inventory in the playbook dir:
```
git clone https://github.com/UMNET-perfSONAR/ansible-inventory-pssid-ilab.git
```

4. Get the required roles (ignore errors so we can run this multiple times):

```
ansible-galaxy install -r requirements.yml --ignore-errors
```

5. On bastion, clean out ~/.ssh/known_hosts
6. ssh to [ip address]:
  IP address of the RPi where ubuntu was installed in the previous steps
  IP address can retrieved from consose with `ip a` or `ip a|grep eth0`
  inet address of eth0 is the ip address we need

7. Edit `ansible-inventory-pssid-ilab/inventory/hosts` and under `[pssid-testpoints]` add the ip address from previous step and other ip addresses from hosts need to be provisioned

**provision user accounts as ubuntu**
8. Check if public key exists
  run `ls ~/.ssh/`
  if there is a file that ends in .pub, the content is your own unique public key
  else create your key by running `ssh-keygen -t ed25519 -C "your_email@example.com"` or `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`

9. Edit `ansible-inventory-pssid-ilab/playbooks/bootstrap.yml` and add username(s) unders the `vars` field

10. a: Edit `ansible-inventory-pssid-ilab/inventory/group_vars/all/bootstrap_vars.yml`. Under users, add username(s) that were added in step 9 and add their public keys. 

10. b: some variables are already encrypted using ansible vault, you will need the vault password from the admin

11. Provision accounts on the Rpi with ssh keys:
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
After this step, users listed in bootstrap.yml should be able to ssh to [ip address] without needing a password

**Regular provisioning:**

12. Use Ansible ping to verify connectivity to targets:

ansible ping as default user (without -u option from previous command)
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
All hosts listed in inventory should be successfull. In cases of error, check the correct user with the correct public has been assigned an account with step 11.

13. disable ubuntu account after we verify we can use our own account to become root:
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
This disables ssh as ubuntu. Step 12 should still be successfull.

13. Install pSSID, perfsonar packages, rabbitmq:

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
This playbook installs and sets dependencies necessary to run pSSID


14. for each pi listed in hosts, create a directory named ip address of each host in `ansible-inventory-pssid-ilab/inventory/host_vars`. Define any necessary variable specific to the host within their host_vars directory. If SSID authentication using macaddress is necessary, encrypt the macaddresses using ansible vault(use the same password used for other vaulted vars) and define them within specific host directory.


15. Create a config file for pSSID to run with for each host
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

16. Add scripts necessary to start pSSID and retrieve ip address:
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

17. set up networking using ifupdown: install ifupdown and bring up the interfaces
ip addresses could change after this step, but if elk is set up, the machine should send the ip address to elk
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

18. disable networkd and create wpa_supplicant:
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

19. Extract identifiers from pi and save it locally:
```
cd ansible-inventory-pssid-ilab/playbooks
git clone git@github.com:UMNET-perfSONAR/umich-pssid-info.git
cd ../..
ansible-playbook \
  --become  \
  --become-method su \
  --become-user root \
  --ask-become-pass \
  --ask-vault-pass \
  --extra-vars 'overwrite=False' \
  -i ansible-inventory-pssid-ilab/inventory \
  ansible-inventory-pssid-ilab/playbooks/extract_identifiers.yml
```

Make sure push the umich-pssid-info.git contents

pSSID should start everytime the probe is restarted
