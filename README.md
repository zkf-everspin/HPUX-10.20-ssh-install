
# HP-UX 10.20b - SSH INSTALL GUIDE

## Installation

1. Create /tmp/packages
```bash
cd /tmp
mkdir packages
cd packages
```
2. Copy files for Installation
```bash
cp /proj/temp/openssl* .
cp /proj/temp/openssh* .
cp /proj/temp/bzip2* .
cp /proj/temp/gzip* .
cp /proj/temp/libz* .
```
3. Install bzip2 and gzip
```bash
swinstall -s /tmp/packages/bzip* # action > install analysis > done
swinstall -s /tmp/packages/gzip* # action > install analysis > done
```
4. Decompress packages
```bash
source ~/.cshrc # or open new dtterm to update path variables
cd /tmp/packages # if you opened up a new dtterm
bzip2 -d openssl*
bzip2 -d openssh*
gunzip -c libz-1.2.1-pa1.1.tgz | tar xvf -
```
5. Setup libz
```bash
mv libz-1.2.1.sl libz.sl            # rename
cp libz.sl /usr/local/lib           # copy to lib
ln -s /usr /pro                     # create sym link
chmod 755 /usr/local/lib/libz.sl
```
6. Install openssl
```bash
swinstall -s /tmp/packages/openssl* # action > install analysis > done
```
7. Install openssh
```bash
swinstall -s /tmp/packages/openssh* # action > install analysis > done
```
8. Create random seed
```bash
cp /opt/openssh/sbin/prngd /dev/random
cp /opt/openssh/sbin/prngd /dev/urandom
```
9. Generate Host keys (run as root)
```bash
/usr/local/bin/ssh-keygen -t rsa -f /usr/local/etc/ssh_host_rsa_key -N ""
/usr/local/bin/ssh-keygen -t dsa -f /usr/local/etc/ssh_host_dsa_key -N ""
```
10. Configure SSH
```bash
vi /usr/local/etc/sshd_config # edit to have settings below, can look at sys2 or sys6 as a reference
```
>Port 22

>Protocol 2

>AddressFamily inet

>HostKey /usr/local/etc/ssh_host_rsa_key

>HostKey /usr/local/etc/ssh_host_dsa_key

>PermitRootLogin yes

>PubkeyAuthentication yes

>AuthorizedKeysFile .ssh/authorized_keys

>PasswordAuthentication yes

>PermitEmptyPasswords yes

>AllowTcpForwarding no

>X11Forwarding yes

>X11DisplayOffset 10

>X11UseLocalhost no

>TCPKeepAlive yes

>PermitUserEnvironment yes

>Compression yes

>UseDNS yes

>Subsystem   sftp    /usr/local/libexec/sftp-server

>XAuthLocation /usr/bin/X11/xauth


11. Set permissions on configs (as root)
```bash
chmod 644 /usr/local/etc/sshd_config
chmod 600 /usr/local/etc/ssh_host_*_key
chmod 644 /usr/local/etc/ssh_host_*_key.pub
```
12. Configure and Start SSH Service
```bash
vi /etc/services # add the following
```
> ssh 22/tcp

```bash
vi /etc/inetd.conf # add the following
```
> ssh stream tcp nowait root /usr/local/sbin/sshd -I

```bash
vi /var/adm/inetd.sec # add the following
```
> ssh:ALL:ALL:NONE

13. Restart inetd to load SSH
```bash
inetd -c
```