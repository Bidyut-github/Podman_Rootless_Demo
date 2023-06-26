# Running a Rootless Container With Podman on RHEL

This documentation walks you through the steps of running a rootless container using Podman on Red Hat Enterprise Linux (RHEL). Make sure to replace `(your_username)` with your actual username in the steps below.

## Step 1: Create a User

Create a new user "rhuser":

```bash
useradd rhuser
passwd rhuser
```
Use “redhat” for the password:

```bash
usermod -aG wheel rhuser
```

## Step 2: Validate User Existence and Verify Access

Validate that the user exists and log in to verify the user access:

```bash
id rhuser
ssh rhuser@bastion
```

## Step 3: Install Podman

Download podman and check its version:

```bash
sudo yum install -y podman
podman --version
```

## Step 4: Create an Image From a Container File

We will create a simple HTTPD web service:

```bash
mkdir public-html
cd public-html
```

Create an HTML file:

```bash
vi index.html
```

Press “i” and input the following HTML content:

```html
<html>
<body>
  <h1>Welcome to my web service!</h1>
</body>
</html>
```

To save the file, press “esc”, type “:wq” and press “Enter”.

Next, create the container file:

```bash
vi Containerfile
```

Press “i” and in the container file, input the following:

```Dockerfile
FROM docker.io/library/httpd:latest
COPY ./ /usr/local/apache2/htdocs/
```

Save the file in the same way as before, then build the image:

```bash
podman build -t mywebapp .
```

## Step 5: Push the Image to Registry

First, login to quay.io:

```bash
podman login quay.io
```

Next, tag the image:

```bash
podman tag mywebapp quay.io/(your_username)/mywebapp:latest
```

Finally, push the image:

```bash
podman push quay.io/(your_username)/mywebapp:latest
```

## Step 6: Create the Container

Pull the image from the registry and create a container:

```bash
podman pull quay.io/(your_username)/mywebapp:latest
podman run -d --name mywebapp-demo -p 8080:80 quay.io/(your_username)/mywebapp:latest
```

## Step 7: Check the Status

Check the status of the container:

```bash
podman ps
```

## Step 8: Verify Accessibility

Check if the web service is accessible using curl:

```bash
curl -N localhost:8080
```

## Step 9: Set Up Systemd Service

Create a systemd service to get it started after system reboot automatically:

First, create a systemd service file:

```bash
mkdir -p ~/.config/systemd/user/
cd ~/.config/systemd/user
```

Generate the unit file:

```bash
podman generate systemd --name mywebapp-demo --files --new
```

Stop and then delete the `mywebapp-demo` container:

```bash
podman stop mywebapp-demo
podman rm mywebapp-demo
```

Reload the systemd daemon configuration, and then enable and start your new `container-mywebapp-demo` user service:

```bash
systemctl --user daemon-reload
systemctl --user enable --now container-mywebapp-demo
```

Verify that the web server responds

 to requests:

```bash
curl http://localhost:8080
```

And verify that the container is running:

```bash
podman ps
```

Use the container ID information to confirm that the systemd daemon creates a container when you restart the service:

```bash
systemctl --user stop container-mywebapp-demo
podman ps --all
systemctl --user start container-mywebapp-demo
podman ps
```

Ensure that the services for your user start at system boot, then restart your machine:

```bash
loginctl enable-linger
loginctl show-user rhuser
sudo systemctl reboot
```

After rebooting, log back in and verify that the systemd daemon started the `mywebapp-demo` container, and that the web content is available:

```bash
ssh rhuser@bastion
podman ps
curl http://localhost:8080
```

Done!

## Clearing Containers and User

If needed, you can clear the containers and user using the following commands:

```bash
podman rm -af
podman rmi -af
exit
pkill -9 -u rhuser
userdel -r rhuser
```
