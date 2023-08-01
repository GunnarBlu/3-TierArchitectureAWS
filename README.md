<h1>3-Tier Architecture on AWS.
<h2>Overview</h2>
This project consists of the construction of a 3-tier architecture on AWS with SeedDMS installed along with an HAProxy load balancer. The load balancer will provide session persistence with failover.
<h2>Youtube Demo</h2>
(https://youtu.be/CSzfsZT7ofY)
<br />

<h2>Utilities</h2>

- <b>AWS EC2</b> 
- <b>3 Apache Web Servers</b>
- <b>MySQL Database</b>
- <b>HAProxy Load Balancer</b>

<h2>Environments Used </h2>

- <b>Amazon Linux 2</b>
- <b>Ubuntu 20.04</b>

<h2>Project walk-through:</h2>

<p align="center">
Network Topology: <br/>
<img src="https://i.imgur.com/IoEpRBS.png" height="80%" width="80%" alt="Network Topology"/>
<br />
<br />
Create AWS Instances:
  
1.	Log into Amazon Web Service.
2.	Navigate to EC2.
3.	Click Launch Instance.
4.	Select Amazon Linux 2 as your AMI.
5.	Select Free Tier as your instance type.
6.	Change number of instances to 4.
7.	Create a new key pair and download the .pem file to your computer.
8.	Edit the security group by adding both SSH and HTTP ports.
9.	Select Launch Instance.
10.	Validate all your instance are up and running. 
<p align="center">
<br/>
<img src="https://i.imgur.com/kx1f60V.png" height="80%" width="80%" alt="AWS Instances"/>
<br />
<br />
SSH into Instances:
  
1.	Select instance and click connect.
2.	Click the SSH Client tab.
3.	Copy the example provided and paste it into your terminal.
4.	Change the user from root to ec2-user and click enter. (See screenshot).
5.	Repeat these steps for each VM.

<p align="center">
<br/>
<img src="https://i.imgur.com/gAKDZcd.png" height="80%" width="80%" alt="SSH into Instances"/>
<br />
<br />
Create/Configure Database Server:
  
1.	Install MariaDB on any one of the instances with the following commands.
  
    ```
    yum update
  
    yum install -y mariadb-server
  
    systemctl enable mariadb
  
    systemctl start mariadb
  
    systemctl status mariadb
    ```
2.	Access MariaDB and create a database with the following commands.
  
    ```
    sudo mysql
    ```
    In MariaDB create a database with a name of your choosing using the following command:
    ```
    CREATE DATABASE name;
    ```
    Create a user with a name of your choosing along with a password using the following command:
    ```
    CREATE USER ‘USER’@’%” IDENTIFIED BY ‘password’;
    ```
    Set the privileges of the user with the following command replacing the name with one of your choosing:
    ```
    GRANT ALL PRIVILEGES ON *.* TO ‘USER’@’%’;
    
    FLUSH PRIVILEGES;
    ```
    Confirm the database was created with the following command: (See screenshot, in my case the database is named “seeddms”).
    ```
    SHOW DATABASES;
    ```
<p align="center">
<img src="https://i.imgur.com/3kxNxMI.png" height="80%" width="80%" alt="db"/>

  
3.	Make sure the user was created (See screenshot, in our case the user was named “newuser”).
  
     ```
     SELECT User from mysql.user;
     ```
<p align="center">
<img src="https://i.imgur.com/SX4jZwx.png" height="80%" width="80%" alt="newuser"/>
<p align="center">
<br />
Configure the 3 Web Servers:
  
1.	Once logged into one of the other instances, copy and paste the following commands to install Apache and appropriate PHP packages: (We are making this our web server).
  
    ```
    yum update
  
    sudo yum install -y httpd.x86_64
  
    sudo systemctl start httpd.service
  
    sudo systemctl enable httpd.service
  
    sudo yum install php php-{pear,cgi,common,curl,mbstring,gd,mysqlnd,gettext,bcmath,json,xml,fpm,intl,zip,imap}
    ```
2.	Download and extract the SeedDMS package:
  
    ```
    wget https://sourceforge.net/projects/seeddms/files/seeddms-5.1.22/seeddms-quickstart-5.1.22.tar.gz
    
    tar zxvf seeddms-quickstart-5.1.22.tar.gz
    ```
3.	Move the SeedDMS files using the following commands:
  
    ```
    cd seeddms51x
  
    sudo mv conf data pear seeddms www /var/www/html
  
    sudo mv seeddms-5.1.22 /var/www/html
    ```
4.	Navigate to the /html directory using the following command:
    
    ```
    cd /var/www/html
    ```
5.	Grant Apache permission to the conf and data files using the following command:
    
    ```
    sudo chown -R apache:apache /var/www/html/conf
    
    sudo chown -R apache:apache /var/www/html/data
    ```
6.	Remove settings.xml from the /html directory

    ```
    sudo rm settings.xml
    ```
7.	Create the ENABLE_INSTALL_TOOL
  
    ```
    sudo touch ENABLE_INSTALL_TOOL
    ```
8.	Navigate to /etc/httpd/conf and configure the httpd.conf file to match the screenshot below.
    
<p align="center">
<br/>
<img src="https://i.imgur.com/THozh51.png" height="80%" width="80%" alt="SSH into Instances"/>
<br />
  
9.	Restart Apache
  
    ```
    sudo systemctl restart httpd.service
    ```
10.	Open a web browser and go to the IP address of the instance you just configured. (The SeedDMS page should open).
  
11.	Fill out the information to match your system. (screenshot is only an example)
    
    Database Type – mysql.
  
    Server Name – IP address of Database Instance.
  
    Database Name – the name of the database you created.
  
    Username –  the name of the user you created.
  
    Password – the password to the user you created.
  
12.	Check the “Create database tables:” box and click apply.

<p align="center">
<br/>
<img src="https://i.imgur.com/Gei9pLh.png" height="80%" width="80%" alt="SSH into Instances"/>
<br />
<br />

13.	Select delete file ENEBLE_INSTALL_TOOL if possible.
  
14.	Now you should be at the sign in page. Login in with username “admin”, password “admin”, and select the language then click sign in.

<p align="center">
<br/>
<img src="https://i.imgur.com/irjaz8N.png" height="80%" width="80%" alt="SSH into Instances"/>
<br />
  
15.	Now you are at the main page of the SeedDMS site.  
  
<p align="center">
<br/>
<img src="https://i.imgur.com/7lv1uWp.png" height="80%" width="80%" alt="SSH into Instances"/>
<br />
  
16.	Change the name of the server to whatever you want to easily differentiate between them.

17.	Repeat steps 1-16 for the other two servers.

<p align="center">
<br />
Creating and configuring the HAProxy server:
  
1.	Log into Amazon Web Service.
2.	Navigate to EC2.
3.	Click Launch Instance.
4.	Select Ubuntu as your AMI.
5.	Select Free Tier as your instance type.
6.	Select the key pair created earlier.
7.	Edit the security group by adding SSH, HTTP and HTTPS port.
8.	Select Launch Instance.
9.	Validate all your instances are up and running.
  
<p align="center">
<br/>
<img src="https://i.imgur.com/SVZeIl1.png" height="80%" width="80%" alt="SSH into Instances"/>
<br />
  
  
10.	SSH into the HAProxy instance using the steps shown earlier.
  
11.	Install HAProxy using the following commands:
  
    ```
    sudo apt-get install --no-install-recommends software-properties-common
  
    sudo add-apt-repository ppa:vbernat/haproxy-2.6
  
    sudo apt-get install haproxy=2.6.\*
    ```
12.	Ensure HAProxy is running with the following command:
  
    ```
    sudo systemctl status haproxy
    ```
13.	Create the SSL certificate with the following commands:
  
    ```
    sudo openssl genrsa -out /etc/ssl/xip.io/xip.io.key 2048
    
    sudo openssl req -new -key /etc/ssl/xip.io/xip.io.key                    -out /etc/ssl/xip.io/xip.io.csr

    sudo openssl x509 -req -days 365 -in /etc/ssl/xip.io/xip.io.csr                     -signkey /etc/ssl/xip.io/xip.io.key                     -out /etc/ssl/xip.io/xip.io.crt

    sudo cat /etc/ssl/xip.io/xip.io.crt /etc/ssl/xip.io/xip.io.key            | sudo tee /etc/ssl/xip.io/xip.io.pem
    ```
14.	Navigate to the /etc/ssl/xip.io directory and ensure the SSL keys were created.

    ```
    sudo cat xip.io.pem
    ```
15.	Navigate to the /etc/haproxy/ directory
  
16.	Access the haproxy.cfg file and edit it to match the one shown in the screen shot. (Replace the IP address of the servers shown with your own).

<p align="center">
<br/>
<img src="https://i.imgur.com/OhfOCGh.png" height="80%" width="80%" alt="SSH into Instances"/>
<br />
  
17.	Restart HAProxy with the following command:

    ```
    sudo systemctl restart haproxy
    ```
18.	Open a browser and navigate to the IP address of your HAProxy instance. (Make sure to use https://IPADDRESS and add /stats at the end).

<p align="center">
<br/>
<img src="https://i.imgur.com/62rus5t.png" height="80%" width="80%" alt="SSH into Instances"/>
<p align="center">
<br />
Testing the HAProxy server and ensuring it works with SSL:  
  
  **Session persistence allows the HAProxy to cycle through all of the web servers you added into the haproxy.cfg file when one goes down. You know have a working 3-Tier system implemented.**
      
1.	Make sure all instances are running.
  
2.	Navigate to HAProxy instance in a browser using the IP (Make sure to use https://IPADDRESS and skip the warning by proceeding to the site).
  
3.	It will connect you to one of the web servers.
  
4.	Ensure that the SSL certificate is functioning by navigating to the “Not Secure” box and clicking certificate. (See screenshot).

<p align="center">
<br/>
<img src="https://i.imgur.com/om2pWNp.png" height="80%" width="80%" alt="SSH into Instances"/>
<p align="center">
<br />
  
5.	Go to your instances on AWS and stop the instance it connected you to.
  
6.	Refresh the page on the browser and make sure your connected to the next instance.
  
7.	Stop the instance it connects you too again.
  
8.	Refresh the page on the browser to make sure it connects you to the last instance.
  
9.	Stop the instance it connects you too again.
  
10.	Refresh the page on the browser and if you are redirected to the stat page, the 3-Tier architecture is working.

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
