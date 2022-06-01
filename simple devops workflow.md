__Installation of Jenkins in Ubuntu20.04__

update and upgrade the server
```
$ sudo apt update -y && apt upgrade -y
```
Install Java
```
$ sudo apt install openjdk-11-jre
$ java --version
```
install dependency
```
$ sudo apt install net-tools
```
update
```
$ sudo apt update -y
```
download the jenkins package
```
$ wget https://pkg.jenkins.io/debian-stable/binary/jenkins_2.332.3_all.deb
$ ls
```
install the package using dpkg
```
$ dpkg -i jenkins_2.332.3_all.deb
```
```
$ sudo systemctl status jenkins.service
$ sudo systemctl enable jenkins.service
```
http://server_ip:8080 \
note the password from
```
# cat /var/lib/jenkins/secrets/initialAdminPassword
```
input the password in the login wizard and login and
start with jenkins

Next install git 
```
$ sudo apt install git
$ git --version
```

__Install Jenkins on AMI Linux__
```
# amazon-linux-extras
# amazon-linux-extras install epel
# amazon-linux-extras install java-openjdk11
# java --version
# sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
# sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
# yum install fontconfig
# yum install jenkins
# service jenkins start
# systemctl enable jenkins.service
```

__Integrate Git with Jenkins__

go to jenkins GUI 
click -> manage jenkins
click -> manage plugins
click -> available
search -> github
select -> github plugin
click -> without restart

To configure jenkins
click -> manage jenkins
click -> global tools configuration
under git 
give name to Git
under path to Git executable to git
click -> apply
click -> save

__Run Jenkins job to pull code from GitHub__

click -> New item
Enter an item name: PullCodeFromGitHub
click -> freestyle project
click -> ok
Description: Pull Code From GitHub

Source Code Management
choose -> Git
Repository URL: https://github.com/tkdhanasekar/hello-world.git
click -> apply
click -> save
click -> Build Now

__set Maven on Jenkins Server__
Download Apache Maven
```
# cd /opt
# wget https://dlcdn.apache.org/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.tar.gz
# ll
# tar -xvzf apache-maven-3.8.5-bin.tar.gz
# ll
# mv apache-maven-3.8.5 maven
# cd maven
# cd bin
# ./mvn -v
```
outside of mvn folder it will not work
```
# cd ~
# vim .bash_profile
M2_HOME=/opt/maven
M2=/opt/maven/bin
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.......
PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2
:wq!
#echo $PATH
# source .bash_profile
#echo $PATH
# mvn -v
```
now you can check version of maven from any folder
In Jenkins GUI
click -> manage jenkins
click -> manage plugins
click -> available
search -> maven
select -> maven integration
click -> Install without restart

click -> Manage jenkins 
click -> Global tool configuration
under JDK 
Add  JDK 
disable install automatically
Name: Java-11
JAVA_HOME:  /usr/lib/jvm/java-11-openjdk-11........

under Maven
Add Maven
doable install automatically
Name: maven-3.8.5
MAVEN_HOME: /opt/maven
click -> apply and save


__Build Java Project Using Jenkins__

click -> new item
FirstMavenProject
select -> Maven Project
click -> ok
Description: First Maven Project
Under Git
Repository URL : https://github.com/tkdhanasekar/hello-world
under goals and options -> clean install
click -> apply and save
click -> Build Now

__Install Tomcat server__

install java 
```
# amazon-linux-extras
# amazon-linux-extras install java-openjdk11 
# java --version
```
change to opt directory
```
$ cd /opt
$ wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.63/bin/apache-tomcat-9.0.63.tar.gz
$ tar -xvzf apache-tomcat-9.0.63.tar.gz 
$ mv apache-tomcat-9.0.63 tomcat
$ chmod +x /opt/tomcat/bin/startup.sh 
$ chmod +x /opt/tomcat/bin/shutdown.sh
```
create link files for tomcat startup.sh and shutdown.sh
```
ln -s /opt/apache-tomcat/bin/startup.sh /usr/local/bin/tomcatup
ln -s /opt/apache-tomcat/bin/shutdown.sh /usr/local/bin/tomcatdown
```
```
$ tomcatdown
$ tomcatup
```
if needed Change port number in conf/server.xml file under tomcat home

http://<Public_IP>:8080
tomcat application doesnt allow to login from browser. changing a default parameter in context.xml does address this issue
```
$ find / -name context.xml
```
/opt/tomcat/webapps/host-manager/META-INF/context.xml
/opt/tomcat/webapps/manager/META-INF/context.xml

in the above two files comment the values parameter 

```
$ tomcatdown  
$ tomcatup
```

Update users information in the tomcat-users.xml
which is under /opt/tomcat/conf
remove the users details and add this
```
<role rolename="manager-gui"/>
 <role rolename="manager-script"/>
 <role rolename="manager-jmx"/>
 <role rolename="manager-status"/>
 <user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
<user username="deployer" password="deployer" roles="manager-script"/>
<user username="tomcat" password="Cent@123" roles="manager-gui"/>
```

Restart serivce and try to login to tomcat application from the browser.

__Integrate Tomcat with Jenkins__

click -> manage jenkins
click -> available
search -> deploy to container
select -> install without restart

click -> manage jenkins
click -> manage credentials
click -> jenkins
click -> global credentials
click -> add credentials
kind -> username with password
username -> deployer
password -> deployer
ID -> tomcat_deployer
description -> tomcat_deployer
click -> ok
click -> new job
Enter item name -> BuildAndDeployJob
select -> Maven project
click -> ok
Description -> Build Code with help of Maven and deploy on tomcat server
on Source code management -> select git
repository URL -> https://github.com/tkdhanasekar/hello-world.git
branch specifier -> */master
Goals and options -> clean install
on post build actions -> deploy war/ear to container
WAR/EAR files -> **/*.war
Containers -> tomcat 8.x remote
credentials -> deployer/*****(tomcat deployer)
tomcat url -> http://tomcat-server-ip:8080
click -> apply and save

click -> build now
click -> tomcatserver:8080/manager
click ->webapp 

```
$ git clone ssh://github.com/tkdhanasekar/hello-world.git
$ cd webapp/src/main/webapp/
$ vim index.jsp
```
git commit new codes
```
$ git init
$ git status
$ git add .
$ git status
$ git commit -m "new commit"
$ git push origin master
```

click -> Build and deploy Job
click -> source code management
on Build triggers -> enable poll SCM
schedule * * * * *
click -> apply and save 
