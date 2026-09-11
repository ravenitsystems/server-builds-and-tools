# Initial high security build for SE10

These instructions build a secure base on top of a fresh install of EL10 and assumes you are running all commands as the root user. Please follow each step very carefully as its easy to get locked out of the server because of the strict SSH and firewall rules.

## Base setup 

In this section we define the repositories the server will use and install some basic tools to make the rest of the setup easier

### Set SELinux To Enforce
```
setenforce 1

cat >/etc/selinux/config <<EOL
SELINUX=enforcing
SELINUXTYPE=targeted
EOL
```

### Configure swap space
```
fallocate -l 4G /swapfile

chmod 600 /swapfile

mkswap /swapfile

swapon /swapfile

cat >>/etc/fstab <<EOL
/swapfile swap swap defaults 0 0
EOL

sysctl vm.swappiness=2

echo "vm.swappiness = 2" >> /etc/sysctl.conf
```

### Set DNF to automatically update packages
```
dnf install -y dnf-automatic

sed -i 's/^apply_updates = no$/apply_updates = yes/' /etc/dnf/automatic.conf

sed -i 's/^emit_via = stdio$/emit_via = motd/' /etc/dnf/automatic.conf

systemctl enable --now dnf-automatic.timer
```

### Defined the package repositories we will be using
```
dnf install -y epel-release

/usr/bin/crb enable

dnf -y install https://rpms.remirepo.net/enterprise/remi-release-10.rpm
```

### Install basic command line tools
```
dnf install -y nano wget bind-utils net-tools git zip unzip tar mc
```

### Update all packages to latest stable
```
dnf update -y
```

### Finally reboot so any kernel update loads
```
reboot
```



## Add an administrator user

It is very important you have at least one admin user (wheel group) because the steps after this will amongst other things prevent root logins 

### Create your user and give it a password
```
adduser <username>

usermod -aG wheel <username>

passwd raven
```

### Add a public key to authenticate with no password

```
mkdir /home/<username>/.ssh

cat >/home/<username>/.ssh/authorized_keys <<EOL
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDZYBdHXE8G2YvjTpDCJT674vasNTXMYu0v4r93KrtZFAzPimDcZ6aD2sVtWyxPrg9NVjKA+WQKgXVcpsU/Piz2UcP3p7bycp5pkOmmuAD5iVnvhd9ngu9TXHLFFKeO7Bz0vEfS9N61+lvj/k+oDNxg8uaeD0dF9pRiktqrm/j1ZSJ5XkERvQnYVETAZrA2UkgWiid3Gj4AdO0Uf9a5e1U7/fGIomn0boh+GC7tcgGu8C6U/k40Th1gmMIOiNdaCCnNjL7oAiF15jL2QjHqz3vpD2EU/Su3qtSl9/8oECydDPHjse7tKFqqK8ndhOwaQcqnL5Zjvomrq6KWterYEW5tbAI+6KF69DOorHZ0mbWQsKqxFUsv1fGQCWvEz1L0V8KFBq0nOTzjl6i175zHykvUANdfFbQBchcoV7aIj3mN1Cbws+A3nNre1t5AMhrGYNH4l0JEPgMy7y5pyEGWg9n6GQCAsNL2vDt3HgPBT3xTFs6N+4ni8MjCQKUVXxhktEV3BuuqnTeY99Z+yImnKtUXo2YLYjdYZgoSX4PR7gZHswJ3eTTDyMgRaqU+BRmzREckRAgoKo2aKQvOaJjpvzXCX5K8MTPsgwBEWJjPNXQVw/dztoj+zV7sFIJq7s6Tb4HGCD0MWUSbqRF7hvDTsBRHxpVB+3s/1Utslq+hr6DCzw== ft-nashley-06-25
EOL

chown -R <username>:<username> /home/<username>

chmod 700 /home/<username>/.ssh

chmod 600 /home/<username>/.ssh/authorized_keys
```

### Test your admin user (IMPORTANT)

Now you have your admin user you need to test that you can sign in via SSH using your key (no password). 

- Can you sign in via SSH using your key (no password)
- Can you run a sudo command when entering your password

Once you are sure you can do both actions on the list you can proceed to the hardening step
