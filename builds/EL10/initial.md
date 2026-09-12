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

### Install anti virus software

Config files are `/etc/freshclam.conf` and `/etc/clamd.d/scan.conf`

```
dnf install -y clamav clamd clamav-freshclam

freshclam

mkdir -p /run/clamd.scan

chown clamupdate:clamupdate /run/clamd.scan

chmod 750 /run/clamd.scan

systemctl start clamd@scan

systemctl enable clamd@scan

systemctl enable clamav-freshclam

systemctl start clamav-freshclam
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

passwd <username>
```

### Add a public key to authenticate with no password

```
mkdir /home/<username>/.ssh

cat >/home/<username>/.ssh/authorized_keys <<EOL
<your public key>
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
