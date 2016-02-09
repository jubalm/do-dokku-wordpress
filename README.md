# Digitalocean's $5 droplet Wordpress Using Dokku

Setup a Wordpress app in Digitalocean's $5 droplet using Dokku.

## How to use this package

It is worth noting that this setup uses the wordpress convention of moving the core wordpress stuff to it's own directory so we can use it as a dependency.

You don't really need to tinker much with the file system but here's a little breakdown:

```
public
  ├─ uploads        # wordpress uploads folder
  ├─ wordpress      # wordpress core
  ├─ wp-content     # for wordpress plugins and theme customizations
  ├─ index.php      # slightly modified index.php from wordpress
  └─ wp-config.php  # modified wordpress configuration
vendor              # contains composer dependencies
.gitignore          # some very basic list of files to ignore
composer.json       # recipe for our app
nginx.conf          # see nginx.conf section below
Procfile            # see Procfile section
README.md           # you are here
```

#### 1. [Create a droplet](https://cloud.digitalocean.com/droplets/new) on digitalocean

- Choose an image - Dokku v0.4.12 on 14.04 (at the time of writing)
- Choose a size - $5/mo
- Choose a datacenter region - Region closest to your target audience
- Select additional options
  - Private Networking - optional
  - Backups - optional
  - IPv6 - optional
  - User Data - optional
- Add your SSH keys
  - [How To Use SSH Keys with DigitalOcean Droplets](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-keys-with-digitalocean-droplets)
- Finalize and Create
  - How many droplets? - optional (1 will suffice)
  - Choose a hostname - optional

#### 2. Install Composer

Grab the installer [here](https://getcomposer.org/download/) according to your system and install as per instructions on their website.

_* The succeeding instructions assume that `composer.phar` is executable globally. Mine resides in `/usr/local/bin/composer`_

#### 3. Download and setup the dokku-wordpress package

- Download the package at [https://github.com/jubalm/do-dokku-wordpress/archive/master.zip](https://github.com/jubalm/do-dokku-wordpress/archive/master.zip) and extract to your project folder.
- Fire up the terminal and navigate to your project folder
- Run the following command (remember I renamed my composer executable)
  ```
  composer install
  ```
- Once completed, you will find a `composer.lock` file.

#### 4. Create the project repository

_* This assumes [git](https://git-scm.com) is installed on your system_

```
git init
git add .
git commit -m 'init'
```

Of course, the commit message on `-m` can be anything.

#### 5. Deploy

As dokku is pretty much similar to heroku, you can deploy your app by simply pushing a git commit to your dokku instance.

```
git remote add dokkuserv dokku@HOSTNAME:wordpress
git push dokkuserv master
```

Breakdown of the codes above

- `dokkuserv` is the remote repository alias
- `dokku` is the default username for the dokku instance
- `HOSTNAME` should be your digitalocean hostname
- `wordpress` will be the dokku app (container) that will be created

When the push completes, you'll see dokku does it's magic building your app. The log should look like

```
-----> Cleaning up...
-----> Building wordpress from herokuish...
-----> Adding BUILD_ENV to build environment...
-----> PHP app detected
-----> Bootstrapping...
-----> Installing system packages...
       - php (7.0.2)
       - Apache (2.4.16)
       - Nginx (1.8.0)
-----> Enabling PHP extensions...
       - ext-zend-opcache (automatic)
-----> Installing dependencies...
       Composer version 1.0.0-alpha11 2015-11-14 16:21:07
       Loading composer repositories with package information
       Installing dependencies from lock file
       - Installing johnpbloch/wordpress-core-installer (0.2.1)
       Loading from cache

       - Installing johnpbloch/wordpress (4.4.2)
       Loading from cache

       Generating optimized autoload files
-----> Preparing runtime environment...
-----> Checking for additional extensions to install...
-----> Discovering process types
       Procfile declares types -> web
-----> Releasing wordpress (dokku/wordpress:latest)...
-----> Deploying wordpress (dokku/wordpress:latest)...
-----> DOKKU_SCALE file found
=====> web=1
-----> Running pre-flight checks
       For more efficient zero downtime deployments, create a file CHECKS.
       See http://progrium.viewdocs.io/dokku/checks-examples.md for examples
       CHECKS file not found in container: Running simple container check...
-----> Waiting for 10 seconds ...
-----> Default container check successful!
=====> wordpress container output:
       DOCUMENT_ROOT changed to 'public/'
       Using Nginx server-level configuration include 'nginx.conf'
       No dyno detected; using defaults for 1X...
       4 processes at 128MB memory limit.
       Starting php-fpm...
       Starting nginx...
=====> end wordpress container output
-----> Running post-deploy
-----> Configuring wordpress.yourdomain.com...
-----> Creating http nginx.conf
-----> Running nginx-pre-reload
       Reloading nginx
-----> Setting config vars
       DOKKU_APP_RESTORE: 1
-----> Shutting down old containers in 60 seconds
=====> 8ec330943b5655041111128287281d3eb28467876580e34990b9a89abdba521b
=====> Application deployed:
       http://wordpress.yourdomain.com
```

#### 6. Setup the database

Fist we need to install the mariadb plugin on our dokku instance to create our database.

Login to your instance using ssh and run the following.

```
dokku plugin:install https://github.com/dokku/dokku-mariadb.git mariadb
dokku mariadb:create wpdb
```

Once done, let's link our database to our wordpress container

```
dokku mariadb:link wpdb wordpress
```

- `mariadb:link` is a dokku plugin command to link our database
- `wpdb` is the name of the database we just created
- `wordpress` simply provides the name of our app

#### 7. Setup Domain name

Nows a good time to setup the wordpress domain name.

```
dokku domains:add wordpress wordpress.yourdomain.com
```

- `domains:add` is a dokku plugin that adds a domain name to our app
- `wordpress` is the name of our app
- `wordpress.yourdomain.com` is the domain name

Our wordpress container also needs an environment variable `DOMAIN_NAME` to setup wordpress specific urls.

```
dokku config:add wordpress DOMAIN_NAME=wordpress.yourdomain.com
```

Now visit your blog from the domain name you provided and you will be redirected to the wordpress install page.

## Further Customization

You can find valuable resouce on dokku's webiste on how these configurations work.

#### Procfile

This basically sets up commands upon starting your app. You can find more information about this file [here](https://devcenter.heroku.com/articles/procfile) and setting up php [here](https://devcenter.heroku.com/articles/custom-php-settings)

```
web: vendor/bin/heroku-php-nginx -C nginx.conf public/
```

These mean:

  - `vendor/bin/heroku-php-nginx` use nginx as our web server
  - `-C nginx.conf` add the contents of `nginx.conf` to the nginx configuration
  - `public/` serve the website from the `public/` directory

#### nginx.conf

These are very basic configuration for your nginx server. More info can be found [here](https://codex.wordpress.org/Nginx).

```
# Set index
index index.php;

# Global restrictions configuration file.
# Designed to be included in any server {} block.
location = /favicon.ico {
	log_not_found off;
	access_log off;
}

location = /robots.txt {
	allow all;
	log_not_found off;
	access_log off;
}

# Deny all attempts to access hidden files such as .htaccess, .htpasswd, .DS_Store (Mac).
# Keep logging the requests to parse later (or to pass to firewall utilities such as fail2ban)
location ~ /\. {
	deny all;
}

# Deny access to any files with a .php extension in the uploads directory
# Works in sub-directory installs and also in multisite network
# Keep logging the requests to parse later (or to pass to firewall utilities such as fail2ban)
location ~* /(?:uploads|files)/.*\.php$ {
	deny all;
}

# http://wiki.nginx.org/HttpCoreModule
location / {
	try_files $uri $uri/ /index.php?$args;
}

# Add trailing slash to */wp-admin requests.
rewrite /wp-admin$ $scheme://$host$uri/ permanent;

# Directives to send expires headers and turn off 404 error logging.
location ~* ^.+\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf)$ {
  access_log off; log_not_found off; expires max;
}

```
