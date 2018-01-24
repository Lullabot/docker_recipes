# Drupal 8

This is a recipe for a basic Docker configuration for a Drupal 8 site, using Docker4Drupal.

## docker-compose.yml
Place this file in the root of your repository. It contains basic settings that should work in most cases. You can swap in different versions in each container depending on the version of Drupal and PHP you want to run on. See many other options on [Docker4Drupal](https://github.com/wodby/docker4drupal/blob/master/docker-compose.yml).

Note that this configuration also passes your ssh credentials and drush aliases into the container.

## /mariadb-init

Copy this folder to the top level of your repository. Any database dump in this folder will be loaded when the container is created. The default file just creates an empty database so Drupal will work initally. You can swap in a database dump from your actual production site, or use `drush sql-sync` once the container has launched.

## .env
The .env file goes in the root of your repository. You can use it to set up environment variables that you can pass into your containers. By changing variables in this file, you only have to change variables in one place and it will be picked up in your docker-compose file. `COMPOSE_PROJECT_NAME` is a special variable used by Docker that will set the prefix for the container names. In this example, it is also used in the docker-compose file to set the browser url so each project has its own url.

## settings.docker.local.php
Move this file to `/sites/default` to tell Drupal what settings to use for the database. Add the following snippet to `settings.php` so it can be discovered and used:

```
// Add settings for Docker4Drupal containers.
if (!empty($_SERVER['WODBY_DIR_FILES'])) {
  settings.docker.local.php';
}
```

The Drupal4Docker container expects to find your files at `/sites/default/files`, and private files at `/sites/default/files/private`. 
Add these files to your local checkout, or move them in later using drush with something like `drush rsync @prod:%files/ @self:%files`.

## Configure HTTPS/SSL
You may want the containers to use SSL, either to test SSL operations or for consistency with the production urls. This docker-compose.yml file allows for that.

Set up a self-signed SSL certificate for use in local HTTPS containers. On a Mac, do the following, other operating systems may need to be handled differently. All containers using a domain like *.docker.localhost will be able share this cert. The cert is be stored on the host rather than in the container, so this only needs to be done once.

### Make a directory for the cert.

```
mk dir ~/ssl/certs
```
	
### Create a wildcard cert.

```
openssl req \
  -newkey rsa:2048 \
  -x509 \
  -nodes \
  -keyout ~/ssl/certs/key.pem \
  -new \
  -out ~/ssl/certs/cert.pem \
  -subj /CN=*.docker.localhost \
  -reqexts SAN \
  -extensions SAN \
  -config <(cat /System/Library/OpenSSL/openssl.cnf \
      <(printf '[SAN]\nsubjectAltName=DNS:*.docker.localhost')) \
  -sha256 \
  -days 720

```
 

### Add the cert to Mac's keychain.

```
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ~/ssl/certs/cert.pem
```

Adjust all container urls to use HTTPS instead of HTTP, and the port 4443 instead of 8000, for instance:

```
HTTP: http://drupal8.docker.localhost:8000
HTTPS: https://drupal8.docker.localhost:4443
```

## Usage

Launch the container:

```
docker-compose up -d
```
Watch the logs to see when it is ready:

```
docker-compose logs -f
```

The containers are ready when you start to see messages like the following (exit the logs with ctl-c):

```
mailhog_1    | [APIv1] KEEPALIVE /api/v1/events
```

If you set `COMPOSE_PROJECT_NAME` to `drupal8`, you would go to the following location in your browser to see your site once it's up and running. 

```
http://drupal8.docker.localhost:8000
```

Some browsers, like Chrome, will automatically handle any url that ends with `localhost`, otherwise you may have to add this to your hosts file. 

## Docker Commands

```
// Pipe a command through the php container:
docker-compose exec --user 82 php COMMAND

// Run drush:
docker-compose exec --user 82 php drush st

// Execute mysql:
docker-compose exec --user 82 php mysql -udrupal -pdrupal -hmariadb

// See all the containers:
docker ps

// Start your containers in the background (detached):
docker-compose up -d

// Stop your containers without destroying data:
docker-compose stop

// Destroy containers and all their data volumes:
docker-compose down -v

// Watch the logs (exit with ctl-c):
docker compose logs -f

```

For ease in use, create an alias for `docker-compose exec --user 82 php ` in your bash_profile to avoid typing it over and over.
