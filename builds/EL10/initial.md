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
