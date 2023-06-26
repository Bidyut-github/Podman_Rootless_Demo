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
