# Tobira Ansible Example

This repository contains an Ansible playbook that can serve as starting point to setup Tobira.
This can be used to quickly spin up your own Tobira for testing, but this is also suitable as a base for a production Tobira, as long as you make some important changes to it.
Basic understanding of Ansible is required.

## How to get started

Clone/download this repository to your local computer and then make the following changes:

- Adjust hostname in `inventory/test/hosts.yml`.
- Replace dummy passwords in `inventory/test/group_vars/tobira/vault.yml`:
  - `vault_opencast_admin_password`: password of the `admin` user of your Opencast
  - Replace the rest with good generated password
- *Note*: this playbook likely won't be updated for every Tobira version.
  Check the Tobira version in `inventory/test/group_vars/tobira/vars.yml`.
  If it's outdated, consider updating it to the newest version now.
  This might also require updating `roles/tobira/templates/config.toml` and potentially other files.

At this point, you can already deploy this for testing.
To do that, make sure you have SSH access to the specified hostname, ideally via SSH key without password prompt (otherwise pass `-k` flag to `ansible-playbook`).
This also assumes that you can run `sudo` without entering a password on the target machine (otherwise pass `--ask-become-pass` to `ansible-playbook`).
Then you can run:

```sh
ansible-playbook deploy.yml
```

This should run without errors.
After it is done, you should be able to open `http://your.host.name:3080` in your browser and see an empty Tobira.
(Unless your target host is blocking the connection. Try `curl localhost:3080` on the target instead.)

## Managing this Playbook with git

It is strongly recommended to keep your Ansible playbook in a git repository and commit any changes.
There are many many advantages to doing that.
However, do not commit any secrets (e.g. passwords) in plain text!
All secrets should be in an encrypted Ansible vault.
The password for that vault should be kept in a proper password manager.
For example, with this simple example, all secrets are in `inventory/test/group_vars/tobira/vault.yml`.
Encrypt it like this:

```sh
ansible-vault encrypt inventory/test/group_vars/tobira/vault.yml
```

## Configuring Tobira

With this minimal dummy version running, you want to configure Tobira to your needs.

- First of all, go through `roles/tobira/templates/config.toml` and search for `TODO`.
  Look at all occurences and adjust the config if required.
- Now that `opencast.host` is properly set,
- Set up a reverse proxy (e.g. nginx) that is in front of Tobira. That would listen on port 80 and 443. Important tasks of the reverse proxy:
  - Deal with HTTPS
  - Compress certain files (e.g. JS) to improve web app performance
  - Helps with Tobira authentication
- Follow [the setup docs](https://elan-ev.github.io/tobira/setup) and configure everything you need. Most notably:
  - Theme: logos, favicons, colors, fonts
  - User authentication (so that users can log into Tobira)
  - Configure JWT in Opencast to make the uploader, Studio and Editor work
- It's a good idea to once read through all of `roles/tobira/templates/config.toml` to see if there is anything else that's relevant for you.


## Preparing for production

To make this setup work for a production environment, do the following things:

- Add second Ansible environment (e.g. `prod`) by adding a folder in `inventory/`.
- If you want to use an existing PostgreSQL installation, adjust the scripts accordingly.
- Setup some automatic DB backup!
- Be sure you have fully read every file in this Playbook to understand what exactly it does.
- ... potentially more! You have to do some thinking of your own at this point to make sure your server is securely configured.
