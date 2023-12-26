# Home Server Ansible Playbooks

## Ansible Playbooks to help you set up your own server at home

These playbooks will configure and update Ubuntu, install Docker, and deploy the containers.
It uses Traefik as it's reverse proxy manager and Authelia for Two Factor Authentication. See the services list [here](serviceslist.md). All of the services will be accessible at https://\<service>.\<yourdomain>

**Important**: Make sure you set SSL/TLS encryption mode to **full** in Cloudflare.

## Acknowledgments

This project is based heavily on [Rishav Nandi's Ansible Homelab.](https://github.com/rishavnandi/ansible_homelab)

## Prerequisites

In order for you to use these playbooks, you'll need a couple things:

- Basic knowledge of internet protocols (SSH, TCP/UDP etc.), Ansible and Linux
- Router with ports 80 and 443 forwarded to your servers IP address
- Google Account
- Domain name
- Cloudflare account with your domain name [nameservers](https://www.youtube.com/watch?v=uqlo3lCqiy0) pointed to their DNS Servers
- A [supported VPN](https://haugene.github.io/docker-transmission-openvpn/supported-providers/) to use with Transmission*
- A fresh install of Ubuntu Server LTS 22.04

*If you don't plan on using Transmission then the VPN is not needed.

## Setup (Before ansible)

Install Ubuntu server 23.10 and select minimal install, select openssh install if promted. Configure basic networking during install.
Once installed reconfigure networking to a static ip by editing `/etc/netplan/50-cloud-init.yaml`

```yaml
network:
  version: 2
  ethernets:
    {{ networks.lan.interface }}:
      dhcp4: true
  wifis:
    {{ networks.wifi.interface }}:
      optional: true
      access-points:
        "{{ networks.wifi.access_point }}":
          password: "{{ networks.wifi.password }}"
      dhcp4: {{ networks.wifi.dhcp4 }}
      dhcp6: {{ networks.wifi.dhcp6 }}
      addresses: [<enter your ip>]
      gateway4: <enter your gateway ip>
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
      routes:
        - to: default
          via: <enter your gateway ip>
```

Then run `sudo netplan apply` to apply the changes

Configure ssh to run on port 69. 

vim `/lib/systemd/system/ssh.socket`
and update `ListenStream=22` to `ListenStream=69`
now run
```
sudo systemctl daemon-reload
sudo systemctl restart ssh
```

Then run `netstat -ant` and you should see 69 listed (`apt-get install net-tools` if you don't have netstat)

## Setup

Once you've installed Ubuntu, you'll need an SSH Key for Ansible to use. You will need to create an one and copy it to the server. This can be done with the following commands:

We add the email as a comment

```bash
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/homeserver -C <your_email>
ssh-copy-id -i ~/.ssh/homeserver <user>@<server>
```

If this is a re-installation, and you have a previous know host clear it out with ` ssh-keygen -f "~/.ssh/known_hosts" -R <server>`

Note: I'd recommend storing the ssh file at `~/.ssh/homeserver`

Fork this repository, then clone it to your local machine and run the following command to install the required roles:

```bash
git clone https://github.com/nickjg1/homeserver-ansible
ansible-galaxy install -r requirements.yml
```

Change directories and create an ansible vault file with the following command and enter a password when prompted:

```bash
cd homeserver-ansible
ansible-vault create group_vars/all/vault.yml
```

Open the vault file with the following command:

```bash
ansible-vault edit group_vars/all/vault.yml
```

Paste the following into the vault file and replace the values with your own:

```yaml
user_password: "<your sudo password>"
```

## Configuration

Make all the necessary changes to the `group_vars/all/vars.yml` and `hosts/hosts` files to match your environment. Extra packages can be added to `group_vars/all/vars.yml`. Any unwanted services can be removed in the `services/tasks/main.yml` file. See [variable help](variablehelp.md) for more information.

check hosts are listed correctly by running `ansible servers --list-hosts` or `ansible servers --list-hosts -i ./hosts/hosts`

## Notes and Disclaimers

This playbook opens your server up to the internet and potentially malicious attacks. Two factor authentication, Cloudflare and [Jeff Geerling's Security Role](https://github.com/geerlingguy/ansible-role-security) offer good layers of protection, but it's always good practice to be mindful of the risks. Further configuration in Cloudflare can strengthen your security.

This also changes the default listening port of SSH to 69. It can be changed in `group_vars/all/vars.yml`.

## Installation

Run this command, enter your sudo password and vault password when prompted:

```bash
ansible-playbook run.yml -K --ask-vault-pass -i ./hosts/hosts
```

## Post Installation and Troubleshooting

If you need help setting services up or have any issues with your installation, see [post installation help](postinstallation.md).

## Credit

- [Rishav Nandi](https://github.com/rishavnandi)
- [Jeff Geerling](https://github.com/geerlingguy)
