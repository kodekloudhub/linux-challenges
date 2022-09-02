# Linux Challenge 5

We got a couple of tasks that need to be done on `centos-host` server. Most of these tasks are dependent on each other but not all of them.

All the tasks require you to be root, so the first step is to become root

```bash
sudo -i
```

### dns

<details>
<summary>Add a local DNS entry for the database hostname "mydb.kodekloud.com" so that it can resolve to "10.0.0.50" IP address.</summary>

```bash
vi /etc/hosts
```

Add the following line and save

```
10.0.0.50    mydb.kodekloud.com
```
</details>

### network

<details>
<summary>Add an extra IP to "eth1" interface on this system: 10.0.0.50/24.</summary>

```bash
ip address add 10.0.0.50/24 dev eth1
```

</details>

### database

<details>
<summary>Install "mariadb" database server on this server and "start/enable" its service.</summary>

A google search reveals `mariadb-server` to be the package you require

```bash
yum install mariadb-server -y
systemctl enable mariadb
systemctl start mariadb
```

</details>

### security

<details>
<summary>Set a password for mysql root user to "S3cure#321".</summary>

The mariadb package installs a utility `mysqladmin` which is the command to use to do this

```bash
mysqladmin -u root password 'S3cure#321'
```

</details>

### root

<details>
<summary>The "root" account is currently locked on "centos-host", please unlock it.</summary>

```bash
usermod -U root
```

</details>

<details>
<summary>Make user "root" a member of "wheel" group.</summary>

```bash
usermod -G wheel root
```

</details>

### docker-image

<details>
<summary>Pull "nginx" docker image.</summary>

```bash
docker pull nginx
```

</details>

### docker-container

<details>
<summary>Create and run a new Docker container based on the "nginx" image. The container should be named as "myapp" and the port "80" on the host should be mapped to the port "80" on the container.</summary>

Note use of `-d` to make the container run in the background.

```bash
docker run -d -p 80:80 --name myapp nginx
```

</details>

### container-start.sh

<details>
<summary>Create a bash script called "container-start.sh" under "/home/bob/" which should be able to "start" the "myapp" container. It should also display a message "myapp container started!"</summary>


```bash
vi /home/bob/container-start.sh
```

Add these lines and save

```bash
#!/usr/bin/env bash

docker start myapp
echo "myapp container started!"
```

Make executable

```bash
chmod +x /home/bob/container-start.sh
```

</details>

### container-stop.sh

<details>
<summary>Create a bash script called "container-stop.sh" under "/home/bob/" which should be able to stop the "myapp" container. It should also display a message "myapp container stopped!"</summary>


```bash
vi /home/bob/container-stop.sh
```

Add these lines and save

```bash
#!/usr/bin/env bash

docker stop myapp
echo "myapp container stopped!"
```

Make executable

```bash
chmod +x /home/bob/container-stop.sh
```

</details>

### cron


<details>
<summary>Add a cron job for the "root" user which should run "container-stop.sh" script at "12am" everyday.<br>Add a cron job for the "root" user which should run "container-start.sh" script at "8am" everyday.</summary>

```bash
crontab -e
```

Now add the following two lines, one for each job, then save

```
0 0 * * * /home/bob/container-stop.sh
0 8 * * * /home/bob/container-start.sh
```

</details>

### pam

<details>
<summary>Edit the PAM configuration file for the "su" utility so that this utility only accepts the requests from the users that are part of the "wheel" group and the requests from the users should be accepted immediately, without asking for any password.</summary>

```bash
vi /etc/pam.d/su
```

We will find two lines beginning `#auth` which relate to the `wheel` group. Uncomment both and save


</details>

# Automate the entire lab in a single script!

Pretty much everything done above, in the same order. We automate the edit of `/etc/hosts` with an append redirect, the script creation by redirecting a [heredoc](https://linuxize.com/post/bash-heredoc/) with `cat` and each `crontab` entry with the following one-line trick which breaks down as follows:

* `(` - begin subshell - groups the commands within the parens.
* `crontab -l` - list current crontab entries
* `2>/dev/null` - If crontab is empty, an error will be printed, so discard it.
* `;` - command separator
* `echo ...` the new line to append to crontab. It is appended to whatever came out of `crontab -l`
* `)` - end subshell
* `| crontab -` - `crontab` will ingest all the above output and create a new crontab.

Note that in the real world, you would *not* store the actual password in a script like this! You would get a pre-generated password from a secure system like Hashicorp Vault using its CLI, manually authenticating yourself with Vault before running your user setup script.

<details>
<summary>Single Script Automation</summary>

```bash
# Start lab and paste this entire script to the command prompt.
# When it completes, press the check button.
sudo -i

#################################
#
# DNS
#
#################################

# Add a local DNS entry for the database hostname "mydb.kodekloud.com" so that it can resolve to "10.0.0.50" IP address.
echo "10.0.0.50    mydb.kodekloud.com" >> /etc/hosts

#################################
#
# Network
#
#################################

# Add an extra IP to "eth1" interface on this system: 10.0.0.50/24
ip address add 10.0.0.50/24 dev eth1

#################################
#
# Database
#
#################################

# Install "mariadb" database server on this server and "start/enable" its service.
yum install mariadb-server -y
systemctl enable mariadb
systemctl start mariadb

#################################
#
# Security
#
#################################

# Set a password for mysql root user to "S3cure#321"
mysqladmin -u root password 'S3cure#321'

#################################
#
# Root
#
#################################

# The "root" account is currently locked on "centos-host", please unlock it.
usermod -U root
# Make user "root" a member of "wheel" group
usermod -G wheel root

#################################
#
# Docker image
#
#################################

# Pull "nginx" docker image.
docker pull nginx

#################################
#
# docker-container
#
#################################

docker run -d -p 80:80 --name myapp nginx

#################################
#
# container-start.sh
#
#################################

cat <<EOF > /home/bob/container-start.sh
#!/usr/bin/env bash

docker start myapp
echo "myapp container started!"
EOF

chmod +x /home/bob/container-start.sh

#################################
#
# container-stop.sh
#
#################################

cat <<EOF > /home/bob/container-stop.sh
#!/usr/bin/env bash

docker stop myapp
echo "myapp container stopped!"
EOF

chmod +x /home/bob/container-stop.sh

#################################
#
# Cron - Here I demonstrate how to automate contab additions
#
#################################

# Add a cron job for the "root" user which should run "container-stop.sh" script at "12am" everyday.
(crontab -l 2>/dev/null; echo "0 0 * * * /home/bob/container-stop.sh") | crontab -
# Add a cron job for the "root" user which should run "container-start.sh" script at "8am" everyday.
(crontab -l 2>/dev/null; echo "0 8 * * * /home/bob/container-start.sh") | crontab -

#################################
#
# PAM
#
#################################

# Edit the PAM configuration file for the "su" utility  ... etc.
# Here we have to uncomment both lines starting #auth
sed -i 's/#auth/auth/' /etc/pam.d/su
```
</details>
