# Drupal 8 Migration

This is a recipe for a Drupal 7 to Drupal 8 migration, using Docker4Drupal. It contains two database containers, one for D8 and one for D7.

## docker-compose.yml
Place this file in the root of your repository. It contains basic settings that should work in most cases. You can swap in different versions in each container depending on the version of Drupal and PHP you want to run on. See many other options on [Docker4Drupal](https://github.com/wodby/docker4drupal/blob/master/docker-compose.yml).

Note that this configuration also passes your ssh credentials and drush aliases into the container.

## /mariadb-init and /mariadb7-init

Copy these folders to the top level of your repository. Copy a D7 database dump into the mariadb7 folder. The D8 folder just creates an empty database.

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

The D8 files will be created at `/sites/default/files`, and private files at `/sites/default/files/private`. 

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

## Install the Drupal 8 site

The first time visit the site in a browser you should see the installation script, since you don't have a Drupal 8 installation yet. You can create a Drupal 8 site either by following the prompts in the browser or by using drush:

```
drush si 
```

Confirm the site has been created by visiting it in a browser.

## Enable modules in Drupal 8.

You can do this using drush or in the browser. Enable all modules that the migration needs as well as all modules that define content types, fields, formatters, or configuration. To be safe, just enable all the modules you would expect to use on a working site.

## Create a migration

Once all the modules are enabled, the following will initiate the migration:

```
drush migrate-upgrade --legacy-db-key="drupal_7" --legacy-root="../private/d7files" --configure-only
```

Use drush to see the status of the migration:

```
drush ms
```

And use drush to run it:

```
drush mi --all
```

It will take quite a while to run. It may stop before it completes. Check the status with `drush ms`. If it's not done, restart it with another `drush mi --all`, or run specific migrations by name.


