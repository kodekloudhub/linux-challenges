# Linux Challenge 3

Some new developers have joined our team, so we need to create some `users/groups` and further need to setup some `permissions` and `access rights` for them.

All the tasks require you to be root, so the fist step is to become root

```bash
sudo -i
```

# Individual Steps

### admins

<details>
<summary>Create a group called "admins".</summary>

```bash
groupadd admins
```
</details>

### David

<details>
<summary>Create a user called "david" , change his login shell to "/bin/zsh".</summary>

```bash
useradd -s /bin/zsh david
```
</details>

<details>
<summary>Set "D3vUd3raaw" password for this user.</summary>

```bash
passwd david
```

Enter the given password and confirm it.
</details>

<details>
<summary>Make user "david" a member of "admins" group.</summary>

```bash
usermod -G admins david
```

</details>

### Natasha

<details>
<summary>Create a user called "natasha" , change her login shell to "/bin/zsh".</summary>

```bash
useradd -s /bin/zsh natasha
```
</details>

<details>
<summary>Set "DwfawUd113" password for this user.</summary>

```bash
passwd natasha
```

Enter the given password and confirm it.
</details>

<details>
<summary>Make user "natasha" a member of "admins" group.</summary>

```bash
usermod -G admins natasha
```

</details>

### devs

<details>
<summary>Create a group called "devs".</summary>

```bash
groupadd devs
```
</details>

### Ray

<details>
<summary>Create a user called "ray" , change his login shell to "/bin/sh".</summary>

```bash
useradd -s /bin/sh ray
```
</details>

<details>
<summary>Set "D3vU3r321" password for this user.</summary>

```bash
passwd ray
```

Enter the given password and confirm it.
</details>

<details>
<summary>Make user "ray" a member of "devs" group.</summary>

```bash
usermod -G devs ray
```

</details>

### Lisa

<details>
<summary>Create a user called "lisa" , change her login shell to "/bin/sh".</summary>

```bash
useradd -s /bin/sh lisa
```
</details>

<details>
<summary>Set "D3vUd3r123" password for this user.</summary>

```bash
passwd lisa
```

Enter the given password and confirm it.
</details>

<details>
<summary>Make user "lisa" a member of "devs" group.</summary>

```bash
usermod -G devs lisa
```

</details>

### Bob, data

These two tasks are really one...

<details>
<summary>Make sure "/data" directory is owned by user "bob" and group "devs"...</summary>

```bash
chown bob:devs /data
```

</details>

<details>
<summary>...and group "devs" and "user/group" owner has "full" permissions but "other" should not have any permissions.</summary>

```bash
chmod 770 /data
```

</details>

### access control

</details>

<details>
<summary>Give some additional permissions to "admins" group on "/data" directory so that any user who is the member the "admins" group has "full permissions" on this directory.</summary>

```bash
setfacl -m g:admins:rwx /data
```

[Manual page](https://linux.die.net/man/1/setfacl)

</details>

### sudo

<details>
<summary>Make sure all users under "admins" group can run all commands with "sudo" and without entering any password.</summary>

```bash
visudo
```

Enter the following line at the end of the file and save

```
%admins ALL=(ALL) NOPASSWD:ALL
```

</details>

### sudo(dnf)

<details>
<summary>Make sure all users under "devs" group can only run the "dnf" command with "sudo" and without entering any password.</summary>

```bash
visudo
```

Enter the following line at the end of the file and save

```
%devs ALL=(ALL) NOPASSWD:/usr/bin/dnf
```

</details>

### limits

<details>
<summary>Configure a "resource limit" for the "devs" group so that this group (members of the group) can not run more than "30 processes" in their session. This should be both a "hard limit" and a "soft limit", written in a single line.</summary>

```bash
vi /etc/security/limits.conf
```

Enter the following line at the end of the file and save

```
@devs            -       nproc           30
```
</details>

### quota

<details>
<summary>Edit the disk quota for the group called "devs". Limit the amount of storage space it can use (not inodes). Set a "soft" limit of "100MB" and a "hard" limit of "500MB" on "/data" partition.</summary>

First, determine the device path for `/data`

```bash
mount | grep '/data'
```

Then set the quota on the device

```bash
setquota -g devs 100M 500M 0 0 /dev/vdb1
```

[Manual page](https://linux.die.net/man/8/setquota)

</details>

# Automate the entire lab in a single script!

Pretty much everything done above, in the same order. We automate the `vi` and `visudo` steps by using the append redirection to the appropriate files, and the password settings by feeding `passwd` from stdin via a pipe.

Note that in the real world, you would *not* store the actual passwords in a script like this! You would get pre-generated passwords from a secure system like Hashicorp Vault using its CLI, manually authenticating yourself with Vault before running your user setup script.

<details>
<summary>Single Script Automation</summary>

```bash
# Start lab and paste this entire script to the command prompt.
# When it completes, press the check button.
sudo -i


#################################
#
# admins
#
#################################

# Create a group called "admins"
groupadd admins

#################################
#
# David
#
#################################

# Create a user called "david" , change his login shell to "/bin/zsh"
useradd -s /bin/zsh david
# and set "D3vU3r321" password for this user
echo "D3vUd3raaw" | passwd --stdin david
# Make user "david" a member of "admins" group.
usermod -G admins david

#################################
#
# Natasha
#
#################################

# Create a user called "natasha" , change her login shell to "/bin/zsh"
useradd -s /bin/zsh natasha
# and set "DwfawUd113" password for this user
echo "DwfawUd113" | passwd --stdin natasha
# Make user "natasha" a member of "admins" group.
usermod -G admins natasha

#################################
#
# devs
#
################################## Make sure all users under "admins" group can run all commands with "sudo" and without entering any password.

# Create a group called "devs"
groupadd devs

#################################
#
# ray
#
################################## Make sure all users under "admins" group can run all commands with "sudo" and without entering any password.

# Create a user called "ray" , change his login shell to "/bin/sh"
useradd -s /bin/sh ray
# and set "D3vU3r321" password for this user
echo "D3vU3r321" | passwd --stdin ray
# Make user "ray" a member of "devs" group.
usermod -G devs ray

#################################
#
# lisa
#
##################################

# Create a user called "lisa" , change her login shell to "/bin/sh"
useradd -s /bin/sh lisa
# and set "D3vU3r321" password for this user
echo "D3vUd3r123" | passwd --stdin lisa
# Make user "lisa" a member of "devs" group.
usermod -G devs lisa

#################################
#
# bob, data
#
#################################

# Make sure "/data" directory is owned by user "bob" and group "devs"
chown bob:devs /data
# group "devs" and "user/group" owner has "full" permissions but "other" should not have any permissions.
chmod 770 /data


#################################
#
# access control
#
#################################

# Give some additional permissions to "admins" group on "/data" directory so that any user who is the member the "admins" group has "full permissions" on this directory.
setfacl -m g:admins:rwx /data


#################################
#
# Sudo
#
##################################

# Make sure all users under "admins" group can run all commands with "sudo" and without entering any password.
echo '%admins ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

#################################
#
# sudo(dnf)
#
##################################

# Make sure all users under "devs" group can only run the "dnf" command with "sudo" and without entering any password
echo '%devs ALL=(ALL) NOPASSWD:/usr/bin/dnf' >> /etc/sudoers

#################################
#
# limits
#
#################################

#Configure a "resource limit" for the "devs" group ...
echo '@devs            -       nproc           30' >> /etc/security/limits.conf

#################################
#
# quota
#
#################################

# Edit the disk quota for the group called "devs"...
setquota -g devs 100M 500M 0 0 /dev/vdb1
