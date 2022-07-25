# Linux Challenge 2

The app server called `centos-host` is running a Go app on the `8081` port. You have been asked to troubleshoot some issues with `yum/dnf` on this system, Install `Nginx` server, configure Nginx as a `reverse proxy` for this Go app, install `firewalld` package and then configure some `firewall rules`.

All the tasks require you to be root, so the fist step is to become root

```bash
sudo -i
```

# Individual Steps

### Centos Host

Here we are required to diagnose why `yum` is not working, and fix it. This has to be done before we can do anything else!

The error we get if we try to use `yum` to install a package indicates something is up with DNS resolution, to the first place to look is `resolv.conf`

We note that there are no name servers defined, therefore this is the reason why nothing can be resolved.

<details>
<summary>Fix DNS resolution</summary>

```bash
vi /etc/resolv.conf
```

Add Google nameserver as the first line in the file and save

```
nameserver 8.8.8.8
```

</details>

### Packages

<details>
<summary>Install "nginx" package</summary>

```bash
yum install -y nginx
```
</details>

<details>
<summary>Install "firewalld" package</summary>

```bash
yum install -y firewalld
```
</details>

### Security

<details>
<summary>Start and Enable "firewalld" service.</summary>

```bash
systemctl enable firewalld
systemctl start firewalld
```
</details>

Add firewall rules to allow only incoming port "22", "80" and "8081"
The firewall rules must be permanent and effective immediately.

<details>
<summary>Add firewall rules, make permanent and effective.</summary>

```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=8081/tcp --permanent
firewall-cmd --zone=public --add-port=22/tcp --permanent
firewall-cmd --reload
```
</details>

### Go App

<details>
<summary>Start GoApp by running the "nohup go run main.go &" command from "/home/bob/go-app/" directory</summary>

```bash
pushd /home/bob/go-app
nohup go run main.go &
popd
```
</details>

You can use the following to determine when the go app is fully started. Re-run it periodically till the grep returns a match.

<details>
<summary>Check go app is running</summary>

```bash
ps -faux | grep -P '/tmp/go-build\d+/\w+/exe/main'
```
</details>

### Nginx

<details>
<summary>Configure Nginx as a reverse proxy for the GoApp so that we can access the GoApp on port "80"</summary>

```bash
vi /etc/nginx/nginx.conf
```

At line 48 insert the following line after `location / {`

```
            proxy_pass  http://localhost:8081;
```
</details>

<details>
<summary>Start and Enable "nginx" service</summary>

```bash
systemctl enable nginx
systemctl start nginx
```
</details>

### bob

Click the GoApp button above the terminal. You should get a login screen.

# Automate the entire lab in a single script!

Pretty much everything done above, in the same order. We automate the `vi` steps by using `sed` to do the insertions into `resolv.conf` and `nginx.conf` and we automate Bob's login using `curl`

<details>
<summary>Single Script Automation</summary>

```bash
# Start lab and paste this entire script to the command prompt.
# When it completes, press the check button.
sudo -i

#################################
#
# Centos Host
#
#################################

## Fix yum DNS errors by inserting google nameserver
# Can't use sed -i here as we get "Device or resource busy"
sed '1inameserver 8.8.8.8' /etc/resolv.conf > /tmp/resolv.conf
# Can't use cp or mv for the same reason
cat  /tmp/resolv.conf > /etc/resolv.conf

#################################
#
# Packages
#
#################################

yum install -y nginx firewalld

#################################
#
# Security
#
#################################

# Start and Enable "firewalld" service.
systemctl enable firewalld
systemctl start firewalld
# Add firewall rules to allow only incoming port "22", "80" and "8081" and make permanent
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=8081/tcp --permanent
firewall-cmd --zone=public --add-port=22/tcp --permanent
firewall-cmd --reload

#################################
#
# GoApp
#
#################################

# Start GoApp by running the "nohup go run main.go &" command from "/home/bob/go-app/" directory
pushd /home/bob/go-app
nohup go run main.go &

# Wait for it to be running (usually 15-20 seconds as it has to compile first)
while ! ps -faux | grep -P '/tmp/go-build\d+/\w+/exe/main'
do
    sleep 2
done
sleep 2
popd

#################################
#
# Nginx
#
#################################

# Configure Nginx as a reverse proxy for the GoApp so that we can access the GoApp on port "80"
# Do this by inserting a proxy_pass line after "location / {" at line 48
sed -i '48i\            proxy_pass  http://localhost:8081;' /etc/nginx/nginx.conf

# Start nginx
systemctl enable nginx
systemctl start nginx

#################################
#
# bob
#
#################################

# bob is able to login into GoApp using username "test" and password "test"
curl -u test:test http://localhost:80 || echo -e "\n\nBob cannot log in!"
```
</details>