# SonarQube Installation  ( Updated By Joe )

SonarQube provides the capability to not only show health of an application but also to highlight issues newly introduced. With a Quality Gate in place, you can fix the leak and therefore improve code quality systematically.


### Prerequisites
1. Java
2. Database (MySQL) 

### Installation

Install MySQL client version 

 ```sh
  # yum install mysql
 ```
# Download stable SonarQube version from below website. 
- Website: https://www.sonarqube.org/downloads/
- Note: This Article written for SonarQube version

Download & unzip SonarQube Latest Version
```sh
# cd /opt
# wget https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-6.7.6.zip
# unzip sonarqube-6.7.6.zip
# mv /opt/sonarqube-6.7.6 /opt/sonar
```
Create new user give ownership to /opt/sonar directory 
```sh
 # useradd sonar
 # password sonar
 # chown -R sonar:sonar /opt/sonar
```
# Install MySQL Server and Client Database
```sh
# sudo yum localinstall https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
```
```sh
# sudo yum install mysql-community-server
```
```sh
# sudo systemctl enable mysqld
# sudo systemctl start mysqld
```
```sh
# sudo systemctl status mysqld
```
 Check Temporary Password 
```sh
sudo grep 'temporary password' /var/log/mysqld.log
```

Make note of the password, because the next command will ask you to enter the temporary root password.
Run the mysql_secure_installation command to improve the security of our MySQL installation:
```sh
# sudo mysql_secure_installation
```
Connecting to MySQL from the command line

To interact with MySQL through the terminal we will use the MySQL client which is installed as a dependency of the MySQL server package.
To log in to the MySQL server as the root user type:
```sh
mysql -u root -p
```

```sh 
mysql -h <RDS_Instance_endpoint>:3306 -u <DB_USER_NAME> -p <DB_PASSWORD> 
```
Create a new sonar database
```sh
CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;
```

Create a local and a remote user
```sh
CREATE USER sonar@localhost IDENTIFIED BY 'sonar';
CREATE USER sonar@'%' IDENTIFIED BY 'sonar';
```

Grant database access permissions to users 
```sh
GRANT ALL ON sonar.* TO sonar@localhost;
GRANT ALL ON sonar.* TO sonar@'%';
```

check users and databases 
```sh
use mysql
show databases;
SELECT User FROM mysql.user;
FLUSH PRIVILEGES;
QUIT
```
So for you have configured required database information . Let’s Jump back to your instance and enable SonarQube properties file to connect his Database.

### ON Instance
Edit sonar properties file to uncomment and provide required information for below properties. 

- File Name: /opt/sonar/conf/sonar.properties
  - sonar.jdbc.username=`sonar`
  - sonar.jdbc.password=`sonar`
  - sonar.jdbc.url=jdbc:mysql://`<RDS_DATABAE_ENDPOINT>:3306`/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
  - sonar.web.host=`0.0.0.0`
  - sonar.web.context=`/sonar`

Switch to sonar user and start SonarQube service 
```sh
# su - sonar
# cd /opt/sonar/bin/linux-x86-64/
# ./sonar.sh start
```

##### Run SonarQube as a default service 

Implement SonarQube server as a service
```sh
Copy sonar.sh to etc/init.d/sonar and modify it according to your platform.
# sudo cp /opt/sonar/bin/linux-x86-64/sonar.sh /etc/init.d/sonar
# sudo vi /etc/init.d/sonar
```

Add below values to your /etc/init.d/sonar file
```sh
Insert/modify below values
SONAR_HOME=/opt/sonar
PLATFORM=linux-x86-64

WRAPPER_CMD="${SONAR_HOME}/bin/${PLATFORM}/wrapper"
WRAPPER_CONF="${SONAR_HOME}/conf/wrapper.conf"
PIDDIR="/var/run"
```

Start SonarQube server
```sh
# service sonar start
```
SonarQube application uses port 9000. access SonarQube from browser
```sh
  http://<YourIP OR Hostname>:9000/sonar
```
###  Troubleshooting 

1. Check whether you enabled port 9000 in instance security group
2. Check whether you enabled instance IP range in security group
3. /etc/security/limits.conf
4. /etc/sysctl.conf

### Next Step
- [x] [Integrate SonarQube with Jenkins]()
