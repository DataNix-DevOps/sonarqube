# SonarQube Installation 

SonarQube provides the capability to not only show health of an application but also to highlight issues newly introduced. With a Quality Gate in place, you can fix the leak and therefore improve code quality systematically.


# Prerequisites
1. Java
2. Database ( MySQL is Deprecated so we are going to focus on PostgresSQL)

Increase the vm.max_map_count kernal ,file discriptor and ulimit for current session at runtime.
```sh
$ sudo sysctl -w vm.max_map_count=262144
$ sudo sysctl -w fs.file-max=65536
$ ulimit -n 65536
$ ulimit -u 4096
```
```sh
vi  /etc/sysctl.conf  # Add Below To The End of The File
```
```sh
fs.file-max = 2097152
vm.max_map_count = 262144
vm.swappiness = 1
```
```sh
vi /etc/security/limits.conf  # Add Below To The End of The File
```
```sh
elasticsearch soft memlock unlimited
elasticsearch hard memlock unlimited
elasticsearch - nofile 65536
elasticsearch - nproc 4096
root soft memlock unlimited
root hard memlock unlimited
root - nofile 65536
root - nproc 4096
sonar - nofile 65536
sonar - nproc 4096
sonarqube - nofile 65536
sonarqube - nproc 4096
```

# Installation
Step 1: Install OpenJDK
```sh
$ sudo yum install java-11-openjdk-devel
$ sudo java -version
```

# Step 2: Install PostgreSQL
```sh
$sudo yum install https://download.postgresql.org/pub/repos/yum/10/redhat/rhel-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm -y
```
```sh
$ sudo yum install postgresql10-server postgresql10-contrib -y
$ sudo /usr/pgsql-10/bin/postgresql-10-setup initdb
$ sudo vi /var/lib/pgsql/10/data/pg_hba.conf
```
Find below lines at the bottom of the file and change peer to trust and ident to md5 on local, ipv4 and ipv6 connections
```sh
peer  change to trust
ident change to md5
```
Start PostgreSQL Database server
```sh
$ sudo systemctl start postgresql-10
```
Enable it to start automatically at System Startup
```sh
$ sudo systemctl enable postgresql-10
```
Change the password for the default PostgreSQL user.
```sh
$ sudo passwd postgres
```
Switch to the postgres user.
```sh
su - postgres
```
Create a new user by typing:
```sh
createuser sonaruser
```
Switch to the PostgreSQL shell.
```sh
psql
```
Set a password for the newly created user for SonarQube database.
```sh
ALTER USER sonaruser WITH ENCRYPTED password 'admin123';
```
Create a new database for PostgreSQL database by running:
```sh
CREATE DATABASE sonarqube OWNER sonar;
```
Exit from the psql shell:
```sh
\q
```
Switch back to the sudo user by running the exit command.
```sh
exit
```

### Now That DB Setup Is Completed, Let's Move On To SONARQUBE Installation & Configuration 

# Download stable SonarQube version from below website. 
- Website: https://www.sonarqube.org/downloads/
- Note: This Article written for SonarQube version

Download & unzip SonarQube Latest Version
```sh
# cd /opt
# wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.1.0.31237.zip
# unzip sonarqube-8.1.0.31237.zip
# mv /opt/sonarqube-8.1.0.31237 /opt/sonarqube
```
Create new user give ownership to /opt/sonarqube directory 
```sh
 # useradd sonar
 # passwd sonar
 # chown -R sonar:sonar /opt/sonarqube
```
SonarQube Server Instance
Edit sonar properties file to uncomment and provide required information for below properties. 
```sh
vi /opt/sonarqube/conf/sonar.properties
```

```sh
- File Name: /opt/sonarqube/conf/sonar.properties
  - sonar.jdbc.username=`sonaruser`
  - sonar.jdbc.password=`admin123`
  - sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
```
Switch to sonar user and start SonarQube service : Edit the sonar script file and set RUN_AS_USER
```sh
sudo vi  /opt/sonarqube/bin/linux-x86-64/sonar.sh
$RUN_AS_USER="sonar"
```
Execute sonar.sh as service account "sonar user"
```sh
# su - sonar
# cd /opt/sonarqube/bin/linux-x86-64/
# ./sonar.sh start
```

### Configure Systemd service
Create a systemd service file for SonarQube to run as System Startup.

Add the below lines,
```sh
vi /etc/systemd/system/sonarqube.service
```
```sh
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```
Save and close the above file. 

Stop existing sonarqube run that was trigger by sonar and login as "root" to rerun sonar as a server . Start the Sonarqube daemon by running:

```sh
# sudo systemctl start sonarqube.service
```
Enable the SonarQube service to automatically  at boot time System Startup.
```sh
# sudo systemctl enable sonarqube
```
Check if the sonarqube service is running,
```sh
# sudo systemctl status sonarqube
```
SonarQube application uses port 9000. access SonarQube from browser
```sh
  http://<YourIP OR Hostname>:9000
```
```sh
username: admin
password: admin
```

Troubleshooting 
1. Check whether you enabled port 9000 in instance security group
2. Check whether you enabled instance IP range in security group


