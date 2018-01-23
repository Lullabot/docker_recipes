# Drupal 8

This is a recipe for a basic Docker configuration for a Drupal 8 site, using Docker4Drupal.

## docker-compose.yml
Place this file in the root of your repository. It contains basic settings that should work in most cases. You can swap in different versions in each container depending on the version of Drupal and PHP you want to run on. See many other options on [Docker4Drupal](https://github.com/wodby/docker4drupal/blob/master/docker-compose.yml).

Note that this configuration also passes your ssh credentials and drush aliases into the container so you can do things like `drush sql-sync @prod @self` to copy a production database into Docker.

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

The setting for `COMPOSE_PROJECT_NAME` controls everything else. If you set that to `drupal8`, you would go to `http://drupal8.docker.localhost` in your browser to see your site.