# Linux Challenge 2

The app server called `centos-host` is running a Go app on the `8081` port. You have been asked to troubleshoot some issues with `yum/dnf` on this system, Install `Nginx` server, configure Nginx as a `reverse proxy` for this Go app, install `firewalld` package and then configure some `firewall rules`.

All the tasks require you to be root, so the first step is to become root

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

**Note** As of June 2024 there is an issue with the golang installation and you may see errors performing this step. Until it is fixed, use the following workaround:

```
sudo yum reinstall -y golang golang-bin
```

Run the go app.

```bash
pushd /home/bob/go-app
nohup go run main.go &
popd
```

Although the command prompt will return immediately because the process has been started as a background task, it will take 2-3 minutes before it is actually running due to the time it takes to compile. While that is happening, do the nginx configuration.

</details>

### Nginx

<details>
<summary>Configure Nginx as a reverse proxy for the GoApp so that we can access the GoApp on port "80"</summary>

The GoApp itself is listening on port `8081`. By configuring nginx as a reverse proxy listening on port `80` we can redirect the port `80` request somwhere else, in this case on the same machine but on port `8081`.

```bash
vi /etc/nginx/nginx.conf
```

We need to create a new `location` stanza for the root path within the `server {` stanza and redirect anything arriving there to the go app which is running on `http://localhost:8081`. For this we use a [proxy_pass](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/#:~:text=other%20than%20HTTP.-,Passing%20a%20Request%20to%20a%20Proxied%20Server,-When%20NGINX%20proxies) directive within the `location` stanza.

The existing server stanza looks like this

```text
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
```

Within this, insert the new location stanza so it looks like this

```text
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location = / {
            proxy_pass  http://localhost:8081;
        }

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
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

First, become root

```bash
sudo -i
```

Then

```bash
{
# Paste this entire script to the command prompt.
# When it completes, press the check button.

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

# Compile and run go-app from "/home/bob/go-app/" directory
# Do the compile in the foreground so as not to have to poll a background task for completion
pushd /home/bob/go-app
go build -o go-app .
nohup ./go-app &

popd

#################################
#
# Nginx
#
#################################

# Configure Nginx as a reverse proxy for the GoApp so that we can access the GoApp on port "80"
# Do this by inserting a location stanza for root path at line 47 specifying the proxy pass to the go app
sed -i '47i\        location / {\n\            proxy_pass  http://localhost:8081;\n        }\n' /etc/nginx/nginx.conf

# Start nginx
systemctl enable nginx
systemctl start nginx

#################################
#
# bob
#
#################################

# bob is able to login into GoApp using username "test" and password "test"
if curl -s -u test:test http://localhost:80 > /dev/null
then
    echo
    echo "Success! Bob logged in!"
    echo "Press 'Check' button to complete lab"
else
    echo
    echo "Bob cannot log in!"
fi
}
```
</details>