Tool to remote-control your Hack machine(s).

It uses the [Ansible](https://www.ansible.com/) automation tool.

The control machine has to be able to reach the managed Hack
machines. For example, by having all machines connected to the same
local network.

# Setup

## Setup managed Hack machine

Go to **Settings > Sharing** and turn **Remote Login** to **On**.

If you have multiple hack machines, also change **Computer Name** in
the same panel, and then restart. If not you can leave the default
name.

<!--- TODO add screenshot --->

Go to **Settings > Details > Users** and setup a password for your
user.

<!--- TODO add screenshot --->

Take a note of the computer name, user name and password. You will
need this information to setup your control machine. The examples
below will use **endless** and **myuser** respectively.

### **OPTIONAL**: Enable password-less sudo.

``` bash
sudo visudo
```

Add the `NOPASSWD:` in this line:

```
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL
```

## Setup control machine

Clone this repository and then call:

``` bash
./bin/bootstrap
```

This is an interactive script. It will ask you to enter the Hack
machine Computer name and User name from the step above. You can add
as many Hack machines as you like.

The bootstrap script can also install Ansible for yourself, in a
self-contained fashion. If you accept this, you will need to enter a
shell running `./bin/shell` before being able to call Ansible.

**NOTE**: The supported OS for a control machine is Linux. Check
[Ansible documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
to do the setup yourself in another OS.

You can customize the default paths used and more options in the
`data/vars.yaml` file. Please check the comments in the file for
instructions.

### **OPTIONAL**: Enable password-less login.

For this, you need to generate SSH keys and copy the public key to
managed machines.

``` bash
ssh-keygen -t rsa -b 4096
ssh-copy-id -i ~/.ssh/id_rsa.pub myuser@endless.local
```

Replace `myuser` with the user name in the Hack machine. And replace
`endless` with the Hack computer name. The login password will be
asked for the last time. Still, the password to become root will be
needed. To sort out this, you need to grant password-less sudo in the
managed machine using the `visudo` command. See above.

# Usage:

**NOTE**: If you installed Ansible with the bootstrap script, first
enter the shell running `./bin/shell`. Then you can call the commands
below.

**NOTE**: The `./bin/shell` runs in a sandbox (Flatpak), so it is
restricted to your home directory and has other restrictions.

**NOTE**: If you have password-less sudo in the managed machines, you
don't need to pass `--ask-become-pass` in the commands below.

**NOTE**: See the header of each YAML file for more options.

## Manage Game State:

Fetch the game state from the managed Hack machines:

    ansible-playbook playbooks/fetch_game_state.yaml

Copy a game state to the managed Hack machines:

    ansible-playbook playbooks/copy_game_state.yaml -e "src_game_state=game-states/endless.local/my-state.json"

## Manage Hack Modules:

Ensure the version of each Hack module:

The version is ensured to the branch set in `data/vars.yaml`. You can
edit this file and change **branch** to either master or stable.

    ansible-playbook playbooks/ensure_hack_modules.yaml --ask-become-pass

FIXME document custom

## Manage Other Apps:

    ansible-playbook playbooks/ensure_extra_flatpaks.yaml --ask-become-pass

## Take a Screenshot:

Takes screenshots in the managed Hack machines, and saves them in the
control machine:

    ansible-playbook playbooks/take_screenshot.yaml

## Running arbitrary commands:

Check [Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html)
for the available options. Here are some examples:

Check that the Hack machines are reachable:

    ansible hack-machines -m ping

Copy a file to the Hack machines:

    ansible hack-machines -m copy -a "src=my-file.txt dest=/tmp/my-file.txt"

Upgrade the system:

    ansible hack-machines -a "ostree admin upgrade" --become --ask-become-pass

Update the flatpaks:

    ansible hack-machines -a "flatpak update -y" --become --ask-become-pass

Reboot the Hack machines:

    ansible hack-machines -a "systemctl reboot" --become --ask-become-pass

# Troubleshooting

If you ever change the Computer Name of a managed Hack machine, you
can get this error:

``` bash
Offending key for IP in /path/to/home/.ssh/known_hosts:7
```

To fix it run the following command. Replace the `7` with the number
you get in the error above:

``` bash
sed -i '7d' ~/.ssh/known_hosts
```
