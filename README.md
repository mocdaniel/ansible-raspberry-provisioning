# ansible-raspberry-provisioning

This Ansible role provisions Raspberry Pis with some basic configuration such as:

- [x] SSH hardening
- [x] Creating a defined `sudo` user including SSH keys and a password
- [x] Configurating network-related settings, i.e. `/etc/hosts` entries and the actual host name
- [x] Expanding the `root` partition (Rockylinux takes just 3GB on flashing)
- [x] Updating package cache and executing pending upgrades
- [x] Installing some additional tools that might come in handy

It currently works for Raspberries running:

- [x] [RockyLinux8](https://rockylinux.org)

## Usage:

### Setup

You will need to install `ansible` in order to execute the playbooks. This is best to be done within a **virtual environment**:

```
git clone https://github.com/mocdaniel/ansible-raspberry-provisioning && cd ansible-raspberry-provisioning
python3 -m venv venv && source venv/bin/activate
pip install ansible
```

You will also need to copy an SSH key for the standard user to the Raspberry Pi:

```
ssh-keygen -f ~/.ssh/ansible-key # create a key if needed
ssh-copy-id -i ~/.ssh/ansible-key rocky@raspberry-ip
```

### Playbook and inventory

An example playbook could look like this:

```yaml
---
- hosts: raspberries
  gather_facts: yes
  become: yes
  roles:
  - role: raspi_provisioning
```

and could get its host information from an inventory file like this:

```ini
[raspberries]
pi1 ansible_host=192.168.178.188 ansible_user=rocky ansible_private_key_file=~/.ssh/ansible-key
pi2 ansible_host=192.168.178.198 ansible_user=rocky ansible_private_key_file=~/.ssh/ansible-key
```

Note how the key you created and copied to the remote Raspberry is being used by Ansible here.

### Configurable options

Some options should be given at runtime due to security reasons:

| variable         | value        | required | notes                                                |
|:----------------:|:------------:|:--------:|:----------------------------------------------------:|
| password         | `very_secret`| **yes**  | a password of your chosing                           |
| ansible_sudo_pass| `rockylinux` | **yes**  | as alternative, grant `rockylinux` passwordless root |

All other options can be set in [`defaults/main.yml`](defaults/main.yml):

| variable         | value                        | required | notes                                                        |
|:----------------:|:----------------------------:|:--------:|:------------------------------------------------------------:|
| username         | `raspi_user`                 | **yes**  | the name of your future user, it will get sudo rights        |
| ssh_key          | `~/.ssh/raspi_key.pub`       | **yes**  | path to the SSH key you will use for logging onto the system |
| domain           | `intranet_domain`            | **no**   | the domain name you want to use, if you have one             |

Thus, a valid execution of this role could look something like this:

```
ansible-playbook book.yml -i hosts.ini --extra-vars="ansible_password=rockylinux ansible_user=rocky password=$NEWUSERPASSWORD ansible_sudo_pass=rockylinux"
```

## Contributing

You'd like to add to this Ansible role by adding more provisioning, configurability, different distros etc.? You're more than welcome! Open **issues** or **feature requests** using the respective issue templates or add to the repository yourself and create a **pull request**!

### Development setup

Much like the setup for plain usage of this repository, setting up your development environment works best using a **virtual environment**:

```
git clone https://github.com/mocdaniel/ansible-raspberry-provisioning && cd ansible-raspberry-provisioning
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
```

Now, all required packages for testing and debugging the Ansible role while developing should be installed.

### Testing

> :warning: **This currently only works on Linux/Unix-based systems due to systemd - if you're on Windows/MacOS, just commit**

In order to avoid more work *after* creating a pull request, you can test your changes locally, but you will need to install [Docker](https://docs.docker.com/get-docker/) to do so:

```
# make sure Docker is running and you activated the virtual environment
molecule test .
```

You will see a lot of output on your CLI, which hopefully will be mainly **green/yellow**.

Don't worry about creating pull requests, though! Your contribution will be tested automatically once you open one. 

## License

This project is licensed under **MIT license**. You can do *basically* whatever you want with it. For more information and a copy of the license, please see [LICENSE](LICENSE).
