### Docker installation

```
sudo -i
vim docker-install.sh
```

```
#!/bin/bash
# Add Docker's official GPG key:
sudo apt-get update -y 
sudo apt-get install ca-certificates curl -y 
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

```
sh docker-install.sh
```

```
systemctl start docker
systemctl enable docker
```

=> search docker bench security in browser
=> then, move to this offical repo `https://github.com/docker/docker-bench-security`
=> you can these below commands from this repo

```
git clone https://github.com/docker/docker-bench-security.git
cd docker-bench-security
sudo sh docker-bench-security.sh  # => you can see the very less security score 
```

=> Now, you can see the scaned report, next lets go with auditing 
```
apt install auditd
syatemctl start auditd
syatemctl enable auditd
```

=> to list the audit 
```
auditctl -l
```

=> to edit the rules
```
cd /etc/audit/rules.d
vim audit.rules
```

=> add these lines in `audit.rules` file based the report to fix the vulnerabilities
```
-w /var/lib/docker -p rwa -k DOCKER
-w /etc/docker rwxa -k DOCKER
```

=> after fixing, run this again to see the latest report, and you can see the increased score
```
cd docker-bench-security
sudo sh docker-bench-security.sh
```
