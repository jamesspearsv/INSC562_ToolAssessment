# Omeka Classic Self-Hosting Installation Guide

This is intended as a quick references for library staff interested in installing and self-hosting their own instance of Omeka Classic. This guide is meant for individuals with little experience using Linux, web servers, or relation databases. This will cover the basic installation of Omeka Classic dependencies (Apache, PHP, MySql, and ImageMagik) on Ubuntu. These steps may be applicable to other Debian or Ubuntu based operating systems.

Contents:

1. Installing Dependncies
2. Enabling Apache2 mod_rewrite
3. Setting Up MySQL Database and User
4. Installing Omeka on Your Server
5. Additional Links and Resources

*******

## Installilng Dependencies

There are several requirements to self-host your Omeka Classic installation including: 
- a Linux operating system
- an Apache HTTP server (with `mod_rewrite` enabled)
- MySQL version 5.0 or greater
- PHP scrtipting language version 5.4 or higher (with mysqli, exif, and dom extensions installed)
- ImageMagick image manipulation software

To install these dependencies, open your terminal (you can use `CTRL + ALT + T` on Ubuntu) and run this commands:

```bash
sudo apt install apache2, mysql-server, php, php-xml, php-mysql
```

Type in your root password to continue. The above command isntalls Apache HTTP server, MySQL, PHP, and the php extensions mysqli and dom.

The exif php extension should be included with your fresh installation of PHP. To check what php extensions are isntalled running `php -me`. You should see `dom`, `exif`, and `mysqli`.

If you don't see `exif` you may need to enable this extension. I was able to enable mine by editing my `php.ini` file. For me this file was located at **/etc/php/7.4/cli**. You can edit the file by using `sudo nano php.ini` and paging down to the *Dynamic Extensions* section. Look for *;extension=exif*. If this line begins with a '#', then the line is *commented* and is being skipped. Remove the '#' to uncomment the line. Then press `CTRL + O` to save and `CTRL + X` to exit.

To check that you have installed Apache correctly, visit your machines local IP address in a browser and you should see the default Apache index page. You can see your local IP by using the comment `hostname -I`.

To install ImageMagic we need to download the binary file (i.e., the file that runs the program) to our */usr/local/bin* directory. Run this command to move into your user bin directory and download the ImageMagick binary file.

```bash
cd /usr/local/bin && sudo wget https://download.imagemagick.org/ImageMagick/download/binaries/magick
```

To check that you successfully downloaded the binary, run `magick identify -version`.

Congratulations! You have installed all the system requirements for Omeka Classic.

*****

## Enabling Apache2 mod_rewrite

Before we can launch our apache web server we need to enable mod_rewrite so that Omeka can modify our urls as needed. 

First, start your apache server if it is not already running. Use the command:

```bash
sudo systemctl apache2
```

If you are using the Windows Subsystem for Linux 2 (WSL2) like I am then you should run `sudo service apache2 start`.

Next we need to prep our server to enable mod_rewite. We can do that by running this command:

```bash
sudo a2enmod rewrite
```

Then restart the apache server by using either `sudo systemctl restart apache2` or `sudo service apache2 restart`.

We will now need to edit our `apache2.conf` file. You can find this file in */etc/apache2* directory. Use the command `sudo nano /etc/apache2/apache2.conf` to open the file and hen page down. Look for this setting in the file:

```text
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
```

Change `AllowOverride None` to `AllowOverride All`. Then save and exit the file.

Congratualtions! Apache should now be ready to go.

*****

## Setting Up MySQL Database and User

Now we can set up our MySQL database and user the Omeka will use to access the database. 

First log into MySQL using `sudo mysql -u root -p` and type your root password to continue.

Now we need to create the database that Omeka will access. You can name the database whatever you like. In this example we will use *omeka_db*. Use the MySQL commands:

```mysql
CREATE DATABASE IF NOT EXISTS omeka_db
CHARACTER SET utf8
COLLATE utf8_unicode_ci;
``` 

Next, we need to create the user the Omeka will use to access the database. You can name the user whatever you would like and set the password to something you will remember. In this example we are using the username *omeka_user* and the password *password* Use the command:

```mysql
CREATE USER IF NOT EXISTS omeka_user@localhost
IDENTIFIED BY 'password';
```
You should make a note of the database name, username, user host (in this example *localhost*), and user password. We will need all of these later.

Lastly, we need to grant the user we just created permissions to interact with the database we just created. Use the commands:

```mysql
GRANT ALL ON omeka_db.* TO 'omeka_user'@'localhost';
```

Use the command `SHOW GRANTS FOR omeka_user@localhsot;` to show the privileges;. that our new user has. Then use the command `FLUSH PRIVILEGES;` and `/q` to exit MySQL.

Congratulations! Our database should be ready to go now!

*****

## Installing Omeka on Your Server