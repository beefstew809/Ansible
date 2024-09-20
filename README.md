# Ansible

## Install Ansible Server on Rocky 9

In the terminal of your system, run the following:

```bash
sudo dnf install epel-release
sudo dnf install python3-devel
sudo dnf install python3-pip
pip3 install --upgrade pip
pip3 install ansible
ssh-keygen -t ed25519 -C "Ansible Server" -f .ssh/ansibleserver
ssh-copy-id -i ~/.ssh/ansibleserver.pub user@ip-address
```

## Copy code to system
Now you can do a git clone of this repo in your home folder. Copy the SSH key to github and setup the .ssh/config file with:
```
Host github.com
        User git
        Hostname github.com
        PreferredAuthentications publickey
        IdentityFile /home/ansible/.ssh/<your private ansible ssh key>
```
Clone the repo

## Define Systems
Edit inventory.ini and add systems.

Edit group_vars and host_vars as needed.

## Install Roles
- https://github.com/diodonfrost/ansible-role-terraform
- https://github.com/artis3n/ansible-role-tailscale
- https://github.com/GROG/ansible-role-package
- https://github.com/GROG/ansible-role-fqdn
- https://github.com/geerlingguy/ansible-role-docker
```bash
cd AnsibleServer
ansible-galaxy install diodonfrost.terraform -p roles/
ansible-galaxy role install artis3n.tailscale -p roles/
ansible-galaxy role install GROG.package -p roles/
ansible-galaxy role install GROG.fqdn -p roles/
ansible-galaxy role install geerlingguy.docker -p roles/
```

## Ansible Vault
```bash
ansible-vault create vault.yaml
# Input new vault password
# Add tailscale key to file in key pair fashion e.g. tailscale_key: long_key_here
# If you need to edit later (Key expires in 90 days)
ansible-vault edit vault.yaml
```

## Tailscale
Tailscale is used for Github actions. It allows the github runner to connect to the ansible server and make updates whenever there is a successful pull request
```bash
sudo dnf config-manager --add-repo https://pkgs.tailscale.com/stable/rhel/9/tailscale.repo
sudo dnf install tailscale
sudo systemctl enable --now tailscaled
sudo tailscale up
```
Note: If installing on a Proxmox LXC, you need to make a couple changes.

Go to the console of the host that is hosting the LXC
```bash
nano /etc/pve/lxc/<number of ansible lxc>.conf
#Add the following
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```
Shutdown the lxc and start it again. Tailscale should now work.

The following link may also be needed to work with SELinux: https://github.com/abhiseksanyal/tailscale-selinux-policy