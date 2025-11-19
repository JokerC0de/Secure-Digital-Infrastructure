## Secure-Digital-Infrastructure

# student number: 747171

# 1. System Configuration Overview

This server is an Ubuntu 24.04 VM configured according to the Secure Digital Infrastructure ACW specification. All work was completed using SSH, following least-privilege principles, and tested with curl and the automated client.

User Accounts

Deleted: sysadmin (user + home directory).

Existing: ubuntu, maintenance.

Created: marketing, design, audit.

SSH Keys

SSH keypairs for marketing, design, and audit were downloaded using:
sdi-client sshkey --acct <user> --type public/private
Public keys were placed in each account’s ~/.ssh/authorized_keys with correct permissions. Private keys were stored locally.

Account Restrictions

marketing: SFTP + write access to /srv/www for website uploads. No access to design or audit directories.

design: Shell access + full access to own home directory. No web upload permissions.

audit: SFTP-only, no shell. Read-only access to marketing home, design home, and /srv/www.

Permissions were implemented using user groups, directory ownership, and chmod/chown.

Directory Structure (design)
~/project_rocket/cad
~/project_rocket/render
~/project_cheese/research
~/project_cheese/tests

## 2. File Transfer (SFTP)

marketing: read/write to /srv/www via SFTP.

design: read/write only inside their home directory.

audit: SFTP-only chroot, read-only to design, marketing, and /srv/www.

Implementation included:

Subsystem SFTP configuration in /etc/ssh/sshd_config.

Match blocks enforcing SFTP-only and read-only restrictions for audit.

Correct directory ownership under /home/*.

## 3. Web Server (Apache HTTP)

Apache was installed and configured to serve static files from:

/srv/www


The required file /srv/www/student/index.txt contains:

747171


A custom Apache site (sdi.conf) was created with:

DocumentRoot /srv/www

Directory permissions allowing marketing uploads

Alias for /student/

Tested using:

curl http://10.31.224.48/student/


Static site correctly loads for:

IP: 10.31.224.48

Domain: stu-747171-vm1.net.dcs.hull.ac.uk

## 4. Docker Application & Reverse Proxy

The SDI-Docker application was cloned and built:

git clone https://github.com/sbrl/SDI-Docker.git
docker build -t sdi-app .


A systemd service (sdi-docker.service) runs the container automatically on boot:

docker run -p 3000:3000 --name sdi_container sdi-app

Reverse Proxy

A second Apache site (docker.conf) proxies:

docker.stu-747171-vm1.net.dcs.hull.ac.uk  →  localhost:3000


Proxy modules enabled:

a2enmod proxy proxy_http


Tested with:

curl -H "Host: docker.stu-747171-vm1.net.dcs.hull.ac.uk" http://10.31.224.48/

## 5. Maintenance Tasks

Restart services:

sudo systemctl restart apache2
sudo systemctl restart sdi-docker


Check service status:

systemctl status apache2
systemctl status sdi-docker


Update system packages:
sudo apt update && sudo apt upgrade

View Apache logs:
/var/log/apache2/error.log and access.log

Rebuild Docker image after changes:

docker build -t sdi-app .
systemctl restart sdi-docker

## 6. Notes

The configuration follows least-privilege principles, separates user roles clearly, and ensures both the static site and Docker application are securely available through the correct hostnames.
