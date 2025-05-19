# WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS

## Introduction:
The LAMP stack is a very well-known web development platform, it is open source and it is made up of four main components which are: Linux, Apache, MySQL, and PHP (sometimes Perl or Python). This documentation will show you how to setup, configure and use the LAMP stack for your consumption.

---

## Step 0: Getting Ready
Here, we ensure that the environment (Linux Os) we need to run/serve our website is made ready

1. Spin up an EC2 instance of t2.micro with the Ubuntu 24.04 LTS Operating System in a region closest to you (I used eu-west-2a)

![Create-2c-instance](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/1.Instance%20Details.png)

![Instancerunning](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/2.Instance%20Running.png)



2. Ensure you create an SSH Key pair when launching/creating your instance. This key pair would be crucial in accessing this instance via port 22

3. Also, while creating this EC2 instance, it is important to create a security group configured with the below inbound rules:

- Allow traffic on port 80 (HTTP) from any source IP on the internet
- Allow traffic on port 443 (HTTPS) from any source IP on the internet
- Allow traffic on port 22 (SSH) from any source IP (This rule is allowed by default)

4. Ensure that you select the right VPC when creating the EC2 instance (on my case, I used the default VPC).

5. Download the private ssh key, note the location of download and utilize it to access the EC2 instance from your terminal (a windows OS is assumed here) with the help of running this command:

```bash
ssh -i "your-ec2-key.pem" Username@IPAddress
```

 ![Ec2](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/3.Connecting%20to%20EC2%20instance.png)


---

## Step 1 - Install Apache and Configure the Firewall 

1. First, you have to update and upgrade the package manager's repository

To achieve this, after connecting to your instance, run this command in your terminal.

![update_package_manger](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/4.Update_Package%20Manager.png)


2. Install Apache2

```bash
sudo apt install apache2 -y
```
![install_apache](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/5.%20Install%20Apache2.png)

3. Enable Apache2 and Verify that it is running
```bash
sudo systemctl enable apache2
sudo systemctl status apache2
```

![apache_running](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/6.apache2%20running.png)


4. Verify that the server is running and can be accessed locally via your ubuntu shell by running this command:

```bash
curl http://localhost:80
```

5. Test if you can access the default page apache serves on your server by trying to access your EC2 public IP via a browser

```bash
http://18.119.104.76:80
```

![default_apache_page](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/7.default%20page%20running%20on%20ec2.png)

This indicates that the web server has been correctly installed and is now accessible through the firewall.


## Step 2 - Install Apache and Configure the Firewall 
---
### 1. Install a relatioal database (RDB) to handle your webapp

In our case, we installed MySQL. This is a widely used relational database managemet system that works well within PHP environments.

```bash
sudo apt install mysql-server
```

![install_mysql](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/8.Install%20mysql.png)
type y and press Enter, when prompted.

### 2. Enable mysql and Ensure that it is running by running these commands:
```bash
sudo systemctl enable --now mysql
sudo systemctl status mysql
```

### 3. Log in to mysql console
```bash
sudo mysql
```

Running this command will conect your shell to the MySQL server as the admin database user root assumed by the useof 'sudo' when you were executing the sudo mysql command.

### 4. Assign a password to the root user using the mysql_native_password as the default authetication method. The password we assigned to the user for the purpose of this tutorial is "PassW0RD$"
```bash
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassW0RD$';
```

![set_pass_4_root_user](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/9.set%20password%20for%20mysql%20root%20user.png)

Exit the MySQL shell
```bash
exit
```

### 5. Run the script to secure MySQL
This security script comes pre-installed with mysql. This script helps to remove some insecure default settings and locks down access to the database system.
```bash
sudo mysql_secure_installation
```

![secure_mysql](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/10.secure%20mysql.png)

Whether or not you set up the Validation password plugin, the server will ask you to select and confirm a password for the MySQL root user.

### 6. Login to the MySQL console as root
After you have changed the password, log in to the MySQL console as the root user:
```bash
sudo mysql -u root -p
```

you should see a prompt asking you to insert a password (this is because of the -p flag) as shown in the image below:

![test_new_login_cred](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/11.test%20login%20new%20cred.png)

## Step 3 - Install PHP
---
1. Install php. So far, we have installed Apache to serve our web contents, and we installed Mysql to assist us store and manage our data. Now, we will install Php to process codes inorder to display dynamic content to the consumer user.

To set up php on our server, we are going to need the following installed:

-php package
-php-mysql (this is a PHP module that allows PHP to communicate with MySQL databases)
-libapache2-mod-php _(this helps Apache to handle and understand PHP files) to get this all set up in the machine, run:
```bash
sudo apt install php libapache2-mod-php php-mysql
```

![install_php_modules](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/12.Install%20php%20%26%20modules.png)

Confirm the PHP version by running:
```bash
php -v
```

![verify_installation](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/13.verify%20php%20installation.png)

LAMP (Linux, Apache, MySQL, PHP) stack completely.

To test this setup, we would need to setup an Apache Virtual host to hold the website's files and folders. ( Virtual Hosts makes it possible to have many/different websites located on one machine while abstracting this from the users.)

## Step 4 - Create a Virtual Host
---
### 1. _First, create a document directory for the new website you are about to create near the default web dir (/var/www/html).
```bash
sudo mkdir /var/www/my_project_lamp
```

Assign the ownership of the directory to the current user in the session
```bash
sudo chown -R $USER:$USER /var/www/my_project_lamp
```

![assign_ownership_2_user](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/14.assign%20ownership%20of%20LAMP%20dir%20to%20user.png)

### 2. Create a configuration file for your new website
```bash
sudo vim /etc/apche2/sites-available/my_project_lamp.conf
```

Paste the bare-bones configurations stated below into your config file:
```bash
<VirtualHOst *:80>
  ServerName my_project_lamp 
  ServerAlias www.myprojectlamp
  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/my_project_lamp
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

![barebones_config](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/15.laying_d_barebones_config.png)

3. Show the new file in sites-available
```bash
sudo ls /etc/apache2/sites-available
```

```bash
Output->:
000-default.conf
default-ssl.conf
my_project_lamp.conf
```

![view_sites_available](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/16.view_sites_available.png)

With this configuration setup here (my_project_lamp.conf), the web server (Apache in this case) will serve our webproject (my_project_lamp) utilizing /var/www/my_project_lamp

### 4. Enable the new virtual host
```bash
sudo a2ensite projectlamp
```

### 5. Disable Apache's default website. If we do not disable this default website, Apache's default site will overwrite that in the virtual host, and hence, we won't get to be served the site sitting in the virtual host. To disable Apache's default website, run this command
```bash
sudo a2dissite 000-default
```

![disable_default_sites](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/17.disable_apache_default_sites.png)

### 6. Ensure that the configuration in the configuration file does not contain syntax error

You can achieve the above by running the command below.

```bash
sudo apache2ctl configtest
```

![configtest_check](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/18.configtest%20check.png)

### 7. Reload apache for the changes to be applied
```bash
sudo systemctl reload apache2
```

![reload_apache](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/19.relaod%20apache.png)

### 8. Our site is ready, but Apache needs to serve content from the web root (i.e: /var/www/my_project_lamp), but it is empty. So we will now create an index.html file and then try to access our site via our public IP to test it

![index_html_page](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/20.index%20html.png)

### 9. Try to access your new website using your server's public IP
```bash
http://18.119.104.76/
```

![apache_site_check](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/21.Apache_site_check.png)
This html file can be used as the temporary landing page of this server pending when the index.php file is created and pasted in this same directory. When this happens, the index.html file should be deleted or renamed, as if present, apache would consider serving it first before considering the index.php

## Step 5 - Enable PHP on the website.
---
Currently (i.e: by default), the DirectoryIndex setting on the Apache will always tell Apache to take html files more seriously (higher precedence) than php files. (This setting is good such that, when a php file is being served, and one wants to take the site down for maintainance for a while, he/she could easily setup a customized maintenance html file an place it in server's site directory and Apache would serve that maintenance page, instead of the main php file. and when the site maintenance is done, the file could be deleted or removed to resume normal site functions.)

### 1. To modify the precedence Apache gives to different file types, modify the dir.conf file to look like this.

```bash
sudo vim /etc/apache2/mods-enabled/dir.conf
```

```bash
<IfModule mod_dir.c>
  # Change this:
  # DirectoryIndex index.html index.cgi index.ppl index.php index.xhtml index.htm
  # To this:
  DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

![reorder_indexphp](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/22.reorder%20index.php.png)

### 2. Reload Apache

You need to reload Apache so that the changes you have made would apply.

```bash
sudo systemctl reload apache2
```

![reload_apache_to_apply_change](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/23.reload_apache_to_apply_change.png)

### 3. Utilize the php test script to ensure that you have configured Apache to handle and process requests for PHP files.

Create a new file inside the custom web root folder (/var/www/my_project_lamp)

```bash
vim /var/www/my_project_lamp/index.php
```

Type in the words below into the index.php file

```bash
<?php
phpinfo();
```
![add_valid_code](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/24.add_valid_code_inside_index.php%20file_.png)

### 4. Refresh the page and note down any change you observe
![verify_running_php](https://github.com/Thobilobah/Devops_CloudEngr/blob/master/1.LAMP_Stack/images/25.%20verify_index.php_is_running.png)

The page served here provides you with information about the server from the perspective of PHP. This is very useful in terms of debugging and to make sure that all necessary settings are applied correctly.

When you are done checking the information about the server through this page, It is very important that you remove the file you have created (i.e: index.php) so as to protect the sensitive information about the Php enviroment and the server. This can be repeated when necessary if the information is needed.

```bash
sudo rm /var/www/my_project_lamp_/index.php
```

Conclusion

If you are considering/evaluating what stack to utilize in deploying a website and make it accessible to many people, the LAMP stack we just experienced is a very flexible, robust, and efficient stack to assist you in deploying your developed web applications.

To know how to set up, configure, and maintain a LAMP environment, kindly go through the guidelines stipulated in this documentation and it should help you know how to get the job done.

