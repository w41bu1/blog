---

title: WordPress Local and Debugging
description: A powerful and the most popular content management system (CMS).
date: 2025-09-20 17:00:00 +0700
categories: [Technology, Web]
tags: [cms, php, wordpress]

---

**WordPress** is a powerful and the most popular content management system (CMS) that allows you to create, manage, and customize websites and blogs easily. It’s an open-source CMS, built on PHP, and uses either MySQL or MariaDB databases.

* Released in 2003, initially as a blogging platform, later evolved into a full-featured system for websites, online stores, forums, landing pages, etc.
* Today, more than 40% of websites worldwide run on WordPress.

There are two versions of WordPress:

1. **WordPress.com**

   * Hosting service provided by Automattic
   * You only need to register an account, no installation required
   * Limited customization, advanced features require payment

2. **WordPress.org**

   * Open-source, you download and install it on your own hosting/server
   * Full customization, install plugins, themes, write code, build any website you want

---

## Ecosystem

* **Core**: The main CMS
* **Plugins**: Add-on software installed on a WordPress site to extend functionality and add new features
* **Themes**: Add-on software that defines the visual appearance and layout of a WordPress site

---

## Why WordPress Hacking?

[State of WordPress Security in 2024](https://patchstack.com/whitepaper/state-of-wordpress-security-in-2024)

### Most Popular

* Currently, over 40% of all websites worldwide run WordPress
* This means hackers only need to find one common vulnerability => they can exploit millions of sites at once
* Like the saying: *“fish where the fish are”*

### Plugin and Theme

* WordPress Core has been reviewed for a long time by thousands of **developers** and **researchers**, making it difficult for attackers to break in
* However, tens of thousands of plugins and themes from various sources are in use, with inconsistent quality
* Many plugins have poor security code or are no longer updated. Hackers only need to scan plugins/themes for outdated versions and exploit them

---

## Setup WordPress for Hacking

There are many ways to set up WordPress; searching on **Google** will show plenty of guides. Here, I’ll set it up on an **Ubuntu (22.04) virtual machine**:

* No effect on the host machine’s services
* WordPress is lightweight enough to run in a VM

### Install and Configure WordPress

#### Install Dependencies

Install the full stack needed to run WordPress (web server + database + PHP + required extensions).

```shell
sudo apt install apache2 \
                 ghostscript \
                 libapache2-mod-php \
                 mysql-server \
                 php \
                 php-bcmath \
                 php-curl \
                 php-imagick \
                 php-intl \
                 php-json \
                 php-mbstring \
                 php-mysql \
                 php-xml \
                 php-zip
```

#### Install WordPress

Download and install WordPress source code into the web directory.

```shell
# Create a folder to store the website source code
sudo mkdir -p /srv/www

# Change owner to www-data, the default user for Apache/Nginx web server
sudo chown www-data: /srv/www

# Download the latest WordPress package from the official site
# Extract it into /srv/www
curl https://wordpress.org/latest.tar.gz | sudo -u www-data tar zx -C /srv/www
```

To download a specific version:

```shell
curl https://wordpress.org/wordpress-6.6.2.tar.gz | sudo -u www-data tar zx -C /srv/www
```

> Installing from `wordpress.org` is the most reliable method:
>
> * Ubuntu provides a **wordpress** package in its repo, but it’s usually outdated compared to the official release.
> * The WordPress community only supports the official release. If you encounter issues with the Ubuntu package, they won’t help.

#### Configure Apache for WordPress

Create `/etc/apache2/sites-available/wordpress.conf`:

```text
<VirtualHost *:80>  
    DocumentRoot /srv/www/wordpress 
    <Directory /srv/www/wordpress>
        Options FollowSymLinks
        AllowOverride Limit Options FileInfo 
        DirectoryIndex index.php 
        Require all granted 
    </Directory>

    <Directory /srv/www/wordpress/wp-content> 
        Options FollowSymLinks
        Require all granted
    </Directory>
</VirtualHost>
```

**Enable WordPress site**:

```shell
sudo a2ensite wordpress
```

**Enable mod_rewrite**:

```shell
sudo a2enmod rewrite
```

**Disable default site (optional)**:

```shell
sudo a2dissite 000-default
```

Or add `ServerName mywordpress.local` in `VirtualHost` and update your system’s `hosts` file.

**Reload Apache**:

```shell
sudo service apache2 reload
```

#### Configure Database

Open MySQL:

```shell
sudo mysql -u root
```

Create DB and user:

```sql
CREATE DATABASE wordpress;
CREATE USER wordpress@localhost IDENTIFIED BY '<your-password>';
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO wordpress@localhost;
FLUSH PRIVILEGES;
quit
```

Restart MySQL:

```shell
sudo service mysql start
```

#### Configure WordPress DB Connection

Copy and edit config:

```shell
sudo -u www-data cp /srv/www/wordpress/wp-config-sample.php /srv/www/wordpress/wp-config.php
```

Update DB info:

```shell
sudo -u www-data sed -i 's/database_name_here/wordpress/' /srv/www/wordpress/wp-config.php
sudo -u www-data sed -i 's/username_here/wordpress/' /srv/www/wordpress/wp-config.php
sudo -u www-data sed -i 's/password_here/<your-password>/' /srv/www/wordpress/wp-config.php
```

Add secret keys/salts from: [https://api.wordpress.org/secret-key/1.1/salt/](https://api.wordpress.org/secret-key/1.1/salt/)

#### Configure WordPress

Visit [http://localhost](http://localhost) and set **site title**, **username**, **password**, **email** for the `admin` user.

![Configure WordPress](https://ubuntucommunity.s3.us-east-2.amazonaws.com/original/2X/e/ebe4d6066e0c32f14beca85ffd53e5915a4ab278.png)

### Setup Debug on VSCode

To understand request flow, we need debugging.

#### Add PHP Debug Extension

In **VS Code Extensions (Ctrl+Shift+X)** => search **PHP Debug (by Felix Becker)** => **Install**.

#### Install Xdebug

```shell
sudo apt install php-xdebug -y
php -v
```

If you see `with Xdebug v3.x.x`, installation is OK.

#### Configure Xdebug

Edit php.ini:

```ini
zend_extension=xdebug.so
xdebug.mode=debug
xdebug.start_with_request=yes
xdebug.client_host=127.0.0.1
xdebug.client_port=9003
```

Restart Apache:

```shell
sudo systemctl restart apache2
```

#### Configure VSCode `launch.json`

Open project in VS Code:

```
code /srv/www/wordpress
```

Add `launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Listen for Xdebug",
      "type": "php",
      "request": "launch",
      "port": 9003,
      "pathMappings": {
        "/srv/www/wordpress": "${workspaceFolder}"
      }
    }
  ]
}
```

---

Now WordPress is fully set up locally with VS Code debugging.

---

## Extend

### Required Version

Each WordPress version requires a compatible PHP version. Check this to avoid setup errors.

### WordPress Auto Update

Since WordPress **3.7 (2013)**, background automatic updates exist for:

* Security releases
* Maintenance releases
* Not for major versions (unless explicitly enabled)

WordPress checks `api.wordpress.org` for updates and applies them automatically.

To disable auto-updates, add in `wp-config.php`:

```php
define( 'WP_AUTO_UPDATE_CORE', false );
```

### Increase Uploadable Plugin Size


By default, WordPress relies on PHP configuration (`upload_max_filesize`, `post_max_size`, and sometimes `max_execution_time`) to determine the maximum file size that can be uploaded. On many servers, this limit is set as low as **2MB – 8MB**, which is not enough for modern plugins.

#### Why Increase the Limit?

* **Large plugins**: Some plugins such as WooCommerce extensions, page builders, backup tools, or security suites can easily reach **20MB – 100MB**.
* **Avoid errors**: If the plugin `.zip` file exceeds the limit, you’ll see errors like:

  * *“The uploaded file exceeds the upload\_max\_filesize directive in php.ini”*
  * Or the upload process fails silently.
* **Ease of use**: Increasing the limit allows administrators to upload plugins directly from the WordPress Dashboard without needing FTP/SSH access.
* **Development and testing**: Developers testing their own plugins (with bundled assets like JS, CSS, images, or libraries) may generate larger `.zip` files. A higher limit makes local testing smoother.

#### How to Increase Plugin Upload Size

There are several methods depending on your hosting environment:

1. **Edit `php.ini` (if you control the server)**

   ```ini
   upload_max_filesize = 64M
   post_max_size = 64M
   max_execution_time = 300
   ```

2. **Edit `.htaccess` (for Apache servers)**

   ```apache
   php_value upload_max_filesize 64M
   php_value post_max_size 64M
   php_value max_execution_time 300
   php_value max_input_time 300
   ```

3. **Update `wp-config.php`**
   Add before the line `/* That's all, stop editing! Happy publishing. */`:

   ```php
   @ini_set( 'upload_max_filesize' , '64M' );
   @ini_set( 'post_max_size', '64M');
   @ini_set( 'max_execution_time', '300' );
   ```

👉 It’s recommended to set a **reasonable value** (e.g., 64MB or 128MB). Avoid setting it too high (hundreds of MBs), as it may open the door to abuse (e.g., users uploading excessively large files that overload the server).

---