# Secure-Digital-Infrastructure  
**Student Number:** 747171  

## System Overview
Ubuntu 22.04 VM configured per ACW specification. Follows least-privilege principles; tested via `curl` and the automated client.

### User Accounts & SSH Keys
| Account     | Status / Access |
|------------|----------------|
| sysadmin   | Deleted (including home) |
| ubuntu     | Default; password updated |
| maintenance | Existing; unchanged |
| marketing  | SFTP `/srv/www`; no design/audit access |
| design     | SSH shell; read/write home; no web upload |
| audit      | SFTP-only; read-only to marketing, design, `/srv/www` |

- SSH keys installed for `marketing`, `design`, `audit`  
- Public keys in `~/.ssh/authorized_keys`, private keys stored locally  

### Design Directory Structure
~/project_rocket/
cad/
render/
~/project_cheese/
research/
tests/

yaml
Copy code

---

## SFTP Access
| User      | Access |
|-----------|--------|
| marketing | Read/write `/srv/www` |
| design    | Read/write home only |
| audit     | Read-only SFTP; chrooted |

Configured via `/etc/ssh/sshd_config` Match blocks, proper `chown`/`chmod`.

---

## Web Server (Apache)
- Serves static files from `/srv/www`  
- `/srv/www/student/index.txt` contains `747171`  
- Site config (`sdi.conf`) with `DocumentRoot /srv/www` and `/student/` alias  
- Marketing uploads allowed  

**Test:**
```bash
curl http://10.31.224.48/student/
curl -H "Host: stu-747171-vm1.net.dcs.hull.ac.uk" http://10.31.224.48/
Docker & Reverse Proxy
Cloned & built app:

bash
Copy code
git clone https://github.com/sbrl/SDI-Docker.git
docker build -t sdi-app .
Runs automatically via sdi-docker.service:

bash
Copy code
docker run -p 3000:3000 --name sdi_container sdi-app
Reverse proxy (docker.conf):

docker.stu-747171-vm1.net.dcs.hull.ac.uk → localhost:3000

Proxy modules enabled:

bash
Copy code
a2enmod proxy proxy_http
Test Docker:

bash
Copy code
curl -H "Host: docker.stu-747171-vm1.net.dcs.hull.ac.uk" http://10.31.224.48/
Maintenance
bash
Copy code
# Restart services
sudo systemctl restart apache2
sudo systemctl restart sdi-docker

# Check status
systemctl status apache2
systemctl status sdi-docker

# Update system
sudo apt update && sudo apt upgrade

# Rebuild Docker image after changes
docker build -t sdi-app .
systemctl restart sdi-docker
Apache logs: /var/log/apache2/error.log & /var/log/apache2/access.log

Notes
Least-privilege and clear role separation applied

Static site and Docker app securely available via correct hostnames

Permissions and SSH key placement ensure secure access for all users
