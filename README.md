# 747171 - Secure Digital Infrastructure VM Configuration

---

## 1. User Accounts & Permissions

- **maintenance**: Default account, **untouched** (used by automated testing).  
- **marketing**:  
  - Can upload/read/write files to `/srv/www/` via SFTP.  
  - Cannot access `design` or `audit` directories.  
- **design**:  
  - SSH access, can read/write only home directories.  
  - Home subdirectories (nested):
    ```
    project_rocket/cad/
    project_rocket/render/
    project_rocket/project_cheese/
    project_rocket/research/
    project_rocket/tests/
    ```
- **audit**:  
  - Read-only SFTP access to `/home/design/`, `/home/marketing/`, `/srv/www/`.  
  - No shell access.  
- **sysadmin**: Deleted, including home directory.  
- **SSH keys** enabled for all accounts, root login disabled.

---

## 2. SFTP & File Transfer

- **marketing**: read/write `/srv/www/`  
- **design**: read/write home directories only  
- **audit**: read-only, no shell  
- Directories created as per specification for `design`.

---

## 3. Web Server (Static Files)

- **Apache2** installed, serving HTTP on port 80  
- Root directory: `/srv/www`  
- Default test file for automated check:  
/srv/www/student/index.txt
Content: 747171



- Permissions: static files readable by all, writable by `marketing` only

---

## 4. Docker Container & Reverse Proxy

- Docker installed, container from:  
[SDI-Docker GitHub](https://github.com/sbrl/SDI-Docker.git)  
- Container runs HTTP server on port 3000, starts automatically on boot  
- Apache2 reverse proxy configuration:
- `stu-747171-vm1.net.dcs.hull.ac.uk` → static web server  
- `docker.stu-747171-vm1.net.dcs.hull.ac.uk` → Docker container  
- Ensures all HTTP traffic goes through port 80

---

## 5. Security Measures

- Root login disabled; all access via **SSH keys**  
- Home directories permissioned per ACW specification  
- **Firewall (UFW)** enabled: allow 22 (SSH), 80 (HTTP)  
- Regular system updates applied:
```bash
sudo apt update && sudo apt upgrade -y
6. Common Maintenance Tasks
Update system:

bash

sudo apt update && sudo apt upgrade -y
Check service status:

bash
 
sudo systemctl status apache2
sudo systemctl status docker
Restart services:

bash
 
sudo systemctl restart apache2
sudo systemctl restart docker
Backup web directory:

bash
 
sudo tar -czvf /backup/www_backup.tar.gz /srv/www/
Add/remove users:

bash
 
sudo adduser [username]
sudo deluser [username] --remove-home
7. Notes
Fully compliant with automated testing for accounts, SFTP, web server, and Docker

All services configured to survive VM reboot

No passwords included; all accounts use SSH keys only

Nested directories created as required; permissions strictly enforced

  
 
