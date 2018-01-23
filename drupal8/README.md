# Drupal 8

This is a recipe for a basic Docker configuration for a Drupal 8 site, using Docker4Drupal.

## docker-compose.yml
Place this file in the root of your repository. It contains basic settings that should work in most cases. You can swap in different versions in each container depending on the version of Drupal and PHP you want to run on. See many other options on [Docker4Drupal](https://github.com/wodby/docker4drupal/blob/master/docker-compose.yml).

Note that this configuration also passes your ssh credentials and drush aliases into the container.

## /mariadb-init

Copy this folder to your repository. Any database dump in this folder will be loaded when the container is created. This default file just creates an empty database so Drupal will work initally. You can swap in a database dump from your actual production site, or use `drush sql-sync` once the container has launched.

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

## Usage

Launch the container:

```
docker-compose up -d
```
Watch the logs to see when it is ready:

```
docker-compose logs -f
```

The containers are ready when you start to see messages like `mailhog_1    | [APIv1] KEEPALIVE /api/v1/events`

Exit watching the logs with ctl-c.

The setting for `COMPOSE_PROJECT_NAME` controls everything else. If you set that to `drupal8`, you would go to `http://drupal8.docker.localhost:8000` in your browser to see your site once it's up and running.


Run drush commands to check status, copy a database and files from production, clear caches, etc:

```
docker-compose exec --user 82 php drush
```
For ease in use, create an alias for that in your bash_profile.

More Docker commands:

```
// See all the containers:
docker ps

// Start your containers:
docker-compose up -d

// Stop your containers without destroying data:
docker-compose stop

// Destroy containers and all their data:
docker-compose down -v

// Watch the logs (exit with ctl-c):
docker compose logs -f

// Execute a command inside the php container:
docker-compose exec --user 82 php

// Run drush:
docker-compose exec --user 82 php drush st

// Execute mysql:
docker-compose exec --user 82 php mysql -udrupal -pdrupal -hmariadb


```