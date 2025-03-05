# CONFIGURING OPENNEBULA 6.4.5 ON ALMA LINUX 9 with mysql 8:

# Update system and install dependencies
```
sudo dnf update -y
```
```
sudo dnf install -y epel-release
```
```
sudo dnf install -y wget curl net-tools openssl-devel gcc ruby-devel sqlite-devel systemd-devel libxml2-devel libxslt-devel libcurl-devel
```
```
sudo dnf install -y epel-release wget rsync vim screen ethtool iftop sysstat iotop telnet net-tools iptables-services
```

# Installing Chronyd service 
```
sudo dnf install -y chrony
sudo systemctl enable chronyd
sudo systemctl start chronyd
```

# Create OpenNebula repository file
```
cat > /etc/yum.repos.d/opennebula.repo << 'EOF'
[opennebula]
name=OpenNebula Enterprise Edition
baseurl=https://1e4dc406:m2UU4UmU@enterprise.opennebula.io/repo/6.4.5/AlmaLinux/$releasever/$basearch
enabled=1
gpgkey=https://downloads.opennebula.io/repo/repo.key
gpgcheck=0
repo_gpgcheck=0
EOF
```

# Create Passenger repository file
```
cat > /etc/yum.repos.d/passenger.repo << 'EOF'
[passenger]
name=passenger
baseurl=https://oss-binaries.phusionpassenger.com/yum/passenger/el/$releasever/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://oss-binaries.phusionpassenger.com/auto-software-signing-gpg-key.txt
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
EOF
```

# Add MySQL 8.0 repository
```
sudo dnf install -y https://repo.mysql.com/mysql80-community-release-el9.rpm
```
```
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
```

# Install MySQL server and client
```
sudo dnf install -y mysql-community-server mysql-community-client
```

# Start and enable MySQL service
```
sudo systemctl start mysqld
sudo systemctl enable mysqld
```
# Get temporary MySQL password
```
sudo grep 'temporary password' /var/log/mysqld.log
```

# Login as mysql root user
```
mysql -u root -p'B7a6i2pj1&qS'
```
# Change root password
```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'eWdFj(Xg:4RX12e';
FLUSH PRIVILEGES;
```

# Create oneadmin user and database
```
CREATE DATABASE opennebulanew;
CREATE USER 'oneadmin' IDENTIFIED BY 'eWdFj(Xg:4RX12e';
GRANT ALL PRIVILEGES ON opennebulanew.* TO 'oneadmin'@'%';
FLUSH PRIVILEGES;
EXIT;
```

# Install OpenNebula packages
```
sudo yum clean all
```
```
sudo yum -y install opennebula opennebula-sunstone opennebula-gate opennebula-flow
```
```
sudo yum install -y httpd httpd-tools mod_ssl mod_passenger memcached
```

# Copy From Refference Server : 
```
scp -r main_backup/one/* root@10.10.21.80:/etc/one/
scp -r main_backup/httpd/* root@10.10.21.80:/etc/httpd/
scp -r main_backup/httpd/ssl2021/ root@10.10.21.80:/etc/httpd/
scp -r /var/lib/one/remotes/* root@10.10.21.80:/var/lib/one/remotes/
scp -r /etc/my.cnf root@10.10.21.80:/etc/my.cnf
scp -r config root@10.10.21.80:/var/lib/one/
```

# Fix Database 
```
vim /etc/one/oned.conf
```

# Sample configuration for MySQL
```
DB = [ BACKEND = "mysql",
       SERVER  = "localhost",
       PORT    = 0,
       USER    = "oneadmin",
       PASSWD  = "eWdFj(Xg:4RX12e",
       DB_NAME = "opennebulanew",
    TIMEOUT = 2500 ]
```

# Fix Httpd
```
vim /etc/httpd/conf.d/passenger.conf
```

# Create pre-cleanup script
```
cat > /usr/share/one/pre_cleanup << 'EOF'
#!/usr/bin/bash
if [[ ! -f /var/run/one/oned.pid && -f /var/lock/one/one ]]; then
    rm /var/lock/one/one
fi
EOF
```

# Set permissions
```
chmod +x /usr/share/one/pre_cleanup
chown -R oneadmin:oneadmin /var/lib/one /var/log/one /etc/one /var/run/one 
chmod -R 755 /var/lib/one /var/log/one /etc/one /var/run/one  
sudo chown -R oneadmin:oneadmin /var/run/one
sudo chmod -R 755 /var/run/one
sudo chown oneadmin:oneadmin /var/run/one/ssh-socks
```

# Restart OpenNebula,httpd,mysql services
```
systemctl restart httpd
systemctl restart mysqld
systemctl restart opennebula opennebula-sunstone
```

# Important Logs 
```
tail -f /var/log/one/oned.log
tail -f /var/log/one/sched.log
tail -f /var/log/one/sunstone.log
tail -f /var/log/messages
```
# To Check Sunstone Password
```
cat /var/lib/one/.one/one_auth"
```
