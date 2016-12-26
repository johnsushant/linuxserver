# Linux Server Configuration

Server IP Address : 35.166.134.51

SSH Port : 2200

URL of hosted web application : http://ec2-35-166-134-51.us-west-2.compute.amazonaws.com/

### Configuration Steps

- Login to server as root using the private key.
- Create a new user "grader" by the following command. Follow the steps on screen to create the user.
```sh
    adduser grader
```
- To give sudo access to user, first open the sudoers file as follows
```sh
    vi /etc/sudoers
```
- Add the following entry in the file
```sh
    grader  ALL=(ALL:ALL) ALL
```
- Create an SSH-2 RSA key-pair. You can use [puttygen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) for this.
- Login to the remote server as grader. Run the following commands to add the public key.
```sh
    mkdir .ssh
    touch .ssh/authorized_keys
    vi .ssh/authorized_keys
```
- Paste the public key in the authorized_keys file. Run the following commands to change permissions of the folder and file just created.
```sh
    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys
```
- To disable root login by SSH, make the following changes in sshd_config. We shall also change the default ssh port from 22 to 2200.
```sh
    vi /etc/ssh/sshd_config
    PermitRootLogin no
    Port 2200
```
- Update all currently installed packages by running the following commands.
```sh
    sudo apt-get update
    sudo apt-get upgrade
```
- To change timezone to UTC, run the following command and follow onscreen instructions
```sh
    sudo dpkg-reconfigure tzdata
```
- Cofigure the firewall as follows.
```sh
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow 2200/tcp
    sudo ufw allow http
    sudo ufw allow ntp
    sudo ufw enable
```
- Install apache and mod_wsgi as follows.
```sh
    sudo apt-get install apache2
    sudo apt-get install libapache2-mod-wsgi
    sudo a2enmod mod-wsgi
```
- Create a new configuration in /etc/apache2/sites-availble/ as follows and then add the code.
```sh
    sudo vi /etc/apache2/sites-availble/catalog.conf
    
    <VirtualHost *:80>
    ServerName localhost

    WSGIDaemonProcess catalog user=grader group=grader threads=5
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi

    <Directory /var/www/catalog>
        WSGIProcessGroup catalog
        WSGIApplicationGroup %{GLOBAL}
        Require all granted
    </Directory>
    </VirtualHost>
```
- Clone the Item Catalog project in /var/www/catalog using git.
- Disable the default apache site and enable the new configuration as follows
```sh
    sudo a2ensite catalog
    sudo a2dissite 000-default
```
- Install PostgreSQL and configure the user grader as follows
```sh
    sudo apt-get install postgresql
    sudo -u postgres psql
    CREATE USER grader WITH PASSWORD 'xxxx';
    GRANT ALL PRIVILEGES ON DATABASE catalog to grader;
    \c catalog
    GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO grader;
    GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public to grader;
```
- The site can now be accessed [here](http://ec2-35-166-134-51.us-west-2.compute.amazonaws.com/)
