# Deploying Spring boot on Ubuntu / Nginx and MySQL as the database
Instructions for deploying a spring boot application on Ubuntu using nginx as the reverse proxy and MySQL as the database. Setting up a spring boot project is not covered. If you want to set up a quick test project, you can go to
[Spring initializr](https://start.spring.io)

<br ><br >

## Install Nginx
`sudo apt-get update`

`sudo apt upgrade`

`sudo apt-get install nginx`

`sudo apt-get install systemd`  

Check Nginx version to see if installation was successfull: `nginx -v`  
<br ><br ><br >


## Install Java JRE
`sudo apt-get update`

Run: `java -version` to see how to Install Java JRE. 

Check Java JRE version to see if installation was successfull: `java -version`
<br ><br ><br >


## Install Java JDK
`sudo apt-get update` 

Run: `javac -version` to see how to Install Java JDK.  

Check Java JDK version to see if installation was successfull: `javac -version`
<br ><br ><br >


## Instal MySQL
`sudo apt-get update`

`sudo apt install mysql-server`
<br ><br >

Ensure that the server is running using the *systemctl start* command:

`sudo systemctl start mysql.service`
<br ><br >

### Allow Remote Access to MySQL:

Open up the MySQL prompt:

`sudo mysql`
<br ><br >

Then run the following *CREATE USER* command to make the new user authentication a password method. The following example is the new user authentication method with *mysql_native_password*:

`mysql> CREATE USER 'cloudadmin'@'%' IDENTIFIED WITH mysql_native_password BY 'cloudadmin';`
<br ><br >

After creating new users, you can grant appropriate privileges to them:

`mysql> GRANT ALL PRIVILEGES ON *.* TO 'cloudadmin'@'%';`
<br ><br >

Following this, it’s good practice to run the FLUSH PRIVILEGES command. This will free up any memory that the server cached as a result of the preceding CREATE USER and GRANT statements:

`FLUSH PRIVILEGES;`
<br ><br >

Then you can exit the MySQL client:

`mysql> exit`
<br ><br >

One of the more common problems that users run into when trying to set up a remote MySQL database is that their MySQL instance is only configured to listen for local connections. This is MySQL’s default setting, but it won’t work for a remote database setup since MySQL must be able to listen for an external IP address where the server can be reached. To enable this, open up your mysqld.cnf file:

`sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf`

Navigate to the line that begins with the bind-address directive. It will look like this:

```
. . .
lc-messages-dir = /usr/share/mysql
skip-external-locking
#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 127.0.0.1
. . .
```

By default, this value is set to 127.0.0.1, meaning that the server will only look for local connections. You will need to change this directive to reference an external IP address. For the purposes of troubleshooting, you could set this directive to a wildcard IP address, either *, ::, or 0.0.0.0:

```
. . .
lc-messages-dir = /usr/share/mysql
skip-external-locking
#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 0.0.0.0
. . .
```

After changing this line, save and close the file.
Then restart the MySQL service to put the changes you made to mysqld.cnf into effect:

`sudo systemctl restart mysql`

Finally, we can remote mysql on the server to SQL Client such as DBeaver and others.
<br ><br ><br >


## Clone File JAR Project from Github
clone the project JAR file from github, to run on the server
In `/var/www/` enter jar
<br ><br ><br >

## Create a service to start the application

In `/etc/systemd/system/` create a `<myservice>.service` file and add the following.  

```
[Unit]
Description=Final Project Fullstack Java Programming
After=syslog.target

[Service]
User=nanang
ExecStart=/usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java -jar /var/www/SIB6_Deploy_Jobsite/Jobsite_ServerApp.jar
SuccessExitStatus=143
TimeoutStopSec=10
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable systemd service: `sudo systemctl enable <myservice>.service`

Update systemd service: `sudo systemctl daemon-reload`

`sudo systemctl start <myservice>`

`sudo systemctl stop <myservice>`

`sudo systemctl restart <myservice>`

`sudo systemctl status <myservice>`
<br ><br ><br >

## Create an Nginx Config file
In `/etc/nginx/sites-available/` create `<my-appname>` file and add the below setup:  

```
server {
        listen 80;
        listen [::]:80;
        server_name dev.ezcoder.my.id;
        client_max_body_size 10M;

        location / {

                proxy_pass http://localhost:8083;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                rewrite ^/([^/]+)/?$ /$1 break;

                }
}
```

*client_max_body_size* is used to set the maximum size of the request body, because the default value for client_max_body_size is 1 MiB. If the request size exceeds the configured value, it will generate a 413 (Request Entity Too Large) error sent to the client, an example of using client_max_body_size is when uploading large files.
<br ><br >
The default port for the Spring boot application is port 8080, if you change it, make sure to update the port in the above configuration.
For example, the configuration above has updated the port to 8083.  

After creating the file run the following command `sudo ln -s /etc/nginx/sites-available/<my-appname> /etc/nginx/sites-enabled` to symlink the file to the sites enabled directory.  

Enable nginx: `sudo systemctl enable nginx`

Now run `sudo systemctl restart nginx` to restart nginx for the above to take affect.

`sudo systemctl start nginx`

`sudo systemctl stop nginx`

`sudo systemctl restart nginx`

`sudo systemctl status nginx`
<br ><br ><br >

## Deploy
Run `<myservice>.service` to start the jar: `sudo systemctl start <myservice>`

Run nginx: `sudo systemctl start nginx`

Now just go to one of the paths set in spring boot and it will be up and running.
<br ><br ><br >

## SSL/HTTPS

To run the application over HTTPS we will be installing certbot from letsencrypt.  

`sudo apt-get update`  

` sudo apt install certbot python3-certbot-nginx`

Make sure the server name in Nginx uses the domain name.

Automates the process of granting SSL/TLS certificates from Let's Encrypt for websites hosted on Nginx servers:

`sudo certbot --nginx -d "<nama domain>"`

Follow the instructions and complete the certificate installation.  
<br ><br ><br >

### FYI

In all the above wherevere something is in `<>` it means add your own value for this param.

Delete Folder/File: `sudo rm -r <nama folder/file>`

Check java path: `update-alternatives --list java`

Displays a detailed list of files and directories: `ls -l`

Displays Spring Log: `sudo journalctl -u <myservice>.service --since "$(date +'%Y-%m-%d 06:00:00')"`
Press `F` to scroll log

Check status certbot: `sudo systemctl status certbot.timer`

Update SSL Certificate: `sudo certbot renew`

Test Certificate Renewal: `sudo certbot renew --dry-run`

Check Log certbot: `sudo nano /var/log/letsencrypt/letsencrypt.log`

Delete symlink: `sudo rm <symlink location>`

Know the IP address associated with a domain name: `nslookup yourdomain.com`

Example of database connection to psql ubuntu server
```
spring.datasource.url=jdbc:mysql://<ipserver>:3306/<namadatabase>
spring.datasource.username=cloudadmin
spring.datasource.password=cloudadmin
```

Header handling when forwarding requests from the frontend for example, through a proxy or load balancer such as **X-Forwarded-For**, **X-Forwarded-Proto**, to a Spring Boot application.
This command was added to application.properties to handle CORS issues.
```
server.forward-headers-strategy=framework
```

