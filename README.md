# Local Docker Nginx Proxy Manager Setup

# Preresquite

- Virtualization software (VirtualBox / VMWare).
- OS RHEL-based (CentOS).

# Steps

## Hypervisor Installation

1. Launch virtualbox.
2. In the `virtualbox manager`, click on the `new` button.
3. Enter a name for the vm.
4. Set the type to `Linux` and the version to `Red Hat (64-bit)`.
5. Choose the amount of RAM to allocate to the VM. At least 2GB is recommended, though 4GB or more is better for smoother performance.
6. Select `Create a virtual hard disk now` and click `Create`.
7. Choose the hard disk file type `VDI`.
8. Select the hard disk `dynamically allocated`.
9. Allocate at least 20GB of disk space.
10. With the new VM selected, click on `Settings`.
11. Go to `Storage`, and under `Controller: IDE`, click on the `empty disc icon`.
12. Click on the `disc icon` to the right and select `Choose a disk file`.
13. Browse to the location of the downloaded CentOS ISO and select it.
14. Go to `Network` and ensure that `Attached to` is set to `Bridged Adapter`.
15. Start VM and follow the OS installation instructions.

## OS Installation

1. Start VM.
2. The VM will boot from the CentOS ISO. Select `Install CentOS 9`.
3. Select your `preferred language` and click `Continue`.
4. In the `Installation Summary` screen, click on `Installation Destination`.
5. Select the virtual hard disk you created earlier.
6. choose `Automatically configure partitioning` and click `Done`.
7. Click on `Network & Hostname` and turn on the network interface.
8. Click `Begin Installation`.
9. While the installation is proceeding, you can set the root password and create a user.
10. After the installation is complete, click `Reboot`.
11. The VM will reboot, and you should now be booting into your newly installed CentOS system.

## Configure SSH Acces

1. Install OpenSSH Server
   ```
   sudo dnf install openssh-server
   ```
2. Start SSH service
   ```
   sudo systemctl start sshd
   ```
3. Enable SSH Service
   ```
   sudo systemctl enable sshd
   ```
4. Check status SSH Service
   ```
   Sudo systemctl status sshd (check status ssh)
   ```
5. Configure firewall to allow SSH
   ```
   sudo firewall-cmd --permanent --add-service=ssh
   ```
6. Restart firewall
   ```
   sudo firewall-cmd --reload
   ```

## Activate SELinux

1. Check current SELinux status to see if it is enabled.
   ```
   sestatus
   ```
   if SELinux is disabled or permissive mode, you'll need to enable it.
2. Edit the SELinux configuration file, open configuration file using `nano` or `vi`.
   ```
   sudo nano /etc/selinux/config
   ```
   locate the following line:
   ```
   SELINUX=disabled
   ```
   or
   ```
   SELINUX=permissive
   ```
   change it to:
   ```
   SELINUX=enforcing
   ```
3. Reboot the system
   ```
   sudo reboot
   ```

## Docker Installation

1. Install docker
   ```
   sudo dnf install docker
   ```
2. Start and enable docker
   ```
   sudo systemctl start docker
   sudo systemctl enable docker
   ```
3. Verify docker installation
   ```
   sudo docker --version
   ```
4. Install docker compose
   ```
   sudo dnf install -y yum-utils
   sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```
5. Check docker compose version
   ```
   docker compose version
   ```

## Services Installation

### Webserver

install webserver use docker-compose.yml type

    sudo docker compose up -d

### Reverse Proxy

access your ip address in your browser `192.168.xxx.xxx:<ports>` with those port list:

- 80 : nginx proxy manager
- 81 : admin nginx
- 443 : forwarded and certified nginx

add your `custom domain` to `./etc/hosts`

    <ip host> <custom domain>

#### forwarding nginx to nginx proxy manager

login to port `80` using `admin@example.com` as username and `changeme` as password.
go to host and add host like this setting
| Config | Value |
|-|-|
| hostname | example.com|
| protocol | Http|
| Hostforwarded | name_of_container |
| Port | 80 |

**creating Custom SSL Certificate**  
in your arch install `openssl` then create your custom certs

```
mkdir /etc/ssl/private
cd /etc/ssl/private
openssl genpkey -algorithm RSA -out selfsigned.key -aes256
openssl req -new -x509 -key server.key -out selfsigned.crt -days 365
```

follow the instruction, remember passphrase and your name of certs if forgot type

```
openssl x509 -in ./private/selfsigned.crt -text -noout
```

**Register custom SSL Certificate to nginx proxy manager**  
to register just go tab SSL Certificate > add custom Certificate.
upload all te certificate that need to upload such key and stuff
