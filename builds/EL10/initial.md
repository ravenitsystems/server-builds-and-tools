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
