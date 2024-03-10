# Ansible secure vault-pass PoC

This proof of concept shows how to store the vault password in a secure location,
instead of storing it as plain text in a user's directory.

## Requirements

- [VirtualBox](https://www.virtualbox.org/): tested with version 6.1.50
- [Vagrant](https://www.vagrantup.com/): tested with version 2.4.1

## Usage

### Setup

``` bash
vagrant up
vagrant ssh
cd /vagrant
ansible-playbook playbook.yml
```

You will see a secret message that was encrypted with the vault password. The vault password is stored securely.

### Teardown

On the machine where this repo was cloned:

``` bash
vagrant destroy -f
```

## How it works

The password manager [pass](https://www.passwordstore.org/) is used to store the vault password.
These are the steps needed to store the vault password in pass:

1. Create a GPG key. In this PoC this is done automatically, normally you would run ```gpg --generate-key``` and fill in the required information.
2. Find the GPG key ID. This is shown when running ```gpg --list-keys```, look for the long hexadecimal string.
3. Run ```pass init <GPG key ID>```.
4. Add the ansible vault password by running ```pass insert ansible-vault```.

The password can be retrieved with ```pass show ansible-vault```.
This will ask for the GPG password, but only once every 10 minutes or so and this will reset after every usage of the show command.

Where we normally configure a path to a plain text file in ansible.cfg we now configure a script that is called to retrieve the password: [get-vault-pass.sh](get-vault-pass.sh).
The only thing it's doing is calling ```pass show ansible-vault```.
This will only ask for the GPG key once in a while.

Ansible will automatically use the password supplied by the script to decrypt all encrypted variables and files.
