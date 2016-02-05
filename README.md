# Dokku Wordpress on DigitalOcean

## 1. [Create a droplet](https://cloud.digitalocean.com/droplets/new) on digitalocean

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

## 2. Download a copy of the Wordpress package

I am using the latest copy of wordpress but you can opt for a specific version available on [wordpress.org archives](https://wordpress.org/download/release-archive). Simply replace the url after `curl` (or wget)

```
curl http://wordpress.org/latest.tar.gz
tar xfz latest.tar.gz
rm -f latest.tar.gz
```
What the commands mean:
  - `curl http://wordpress.org/latest.tar.gz` - download wordpress archive
  - `region closest to you` - extract the archive
  - `rm -f latest.tar.gz` - delete the archive after extracting


## 3. Setup the container

#### Procfile Configuration

Create the Procfile, fire up your editor and enter these values in it. It's `vim Procfile` on my setup.

```
web: vendor/bin/heroku-php-nginx -C config/nginx-custom.conf web
```

These mean:
  - `vendor/bin/heroku-php-nginx` use nginx as our web server
  - `-C config/nginx-custom.conf` add the contents of `config/nginx-custom.conf` to the nginx configuration
  - `web` serve the website from the 'web/' directory

#### Nginx Configuration

Create the nginx configuration file on `config/nginx.custom.conf`. It's `mkdir config && vim nginx.custom.conf` on my setup.

This is entirely up to your wordpress setup, refer to [wordpress nginx recommendations](https://codex.wordpress.org/Nginx) for more details.

```
# WordPress
location / {
    try_files $uri $uri/ /index.php?$args;
}

# Everything below here is optional, but recommended

# Add trailing slash to */wp-admin requests.
rewrite /wp-admin$ $scheme://$host$uri/ permanent;

# Block php files
# https://bjornjohansen.no/block-access-to-php-files-with-nginx

## uploads
location ~* /(?:uploads|files)/.*.php$ {
	deny all;
	access_log off;
	log_not_found off;
}

## content
location ~* /wp-content/.*.php$ {
	deny all;
	access_log off;
	log_not_found off;
}

# Set time to expire for headers on assets
location ~* .(js|css|png|jpg|jpeg|gif|ico|svg)$ {
    expires 1y;
}

```

### 4. Now we're ready to deploy!

```
git remote add dokku dokku@YOURHOST:WORDPRESS
```

This will add dokku to your repository's remotes. `YOURHOST` is your website's hostname, while `WORDPRESS` is the name of the container.

```
git push dokku master
```

Now the moment of truth...
