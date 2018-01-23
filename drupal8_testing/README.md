# Drupal 8

This is a recipe for a basic Docker configuration for a Drupal 8 site, using Docker4Drupal.

## docker-compose.yml
Place this file in the root of your repository. It contains basic settings that should work in most cases. You can swap in different versions in each container depending on the version of Drupal and PHP you want to run on. See many other options on [Docker4Drupal](https://github.com/wodby/docker4drupal/blob/master/docker-compose.yml).

Note that this configuration also passes your ssh credentials and drush aliases into the container.

## /mariadb-init

Copy this folder to the top level of your repository. Any database dump in this folder will be loaded when the container is created. For writing tests, I use our [Pantheon Drupal 8 Demo site](https://dashboard.pantheon.io/sites/0b158d4b-5278-414b-8150-6afb75ac1f8e#dev/code)  as a test environment (credentials are in LastPass), and I have a database dump from that site in this folder.

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

## phpunit.xml

This phpunit.xml file has been populated with values that will work correctly in a Drupal4Docker site. Copy this file to `/core/phpunit.xml`.

Some of the values that matter are:

```
// Make sure the output will display on the command line in the container.
printerClass="\Drupal\Tests\Listeners\HtmlOutputPrinter

// Simpletest values, confusingly required even if you use the new browser tests instead of Simpletest.
<env name="SIMPLETEST_BASE_URL" value="http://nginx"/>
<env name="SIMPLETEST_DB" value="mysql://drupal:drupal@mariadb/drupal"/>
<env name="BROWSERTEST_OUTPUT_DIRECTORY" value="/var/www/html/sites/default/files/simpletest"/>

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


## Running Tests

```
// One time, make sure all testing requirements are installed.
composer install  —dev

// Run all one core test
docker-compose run --user 82 php vendor/bin/phpunit -c core core/tests/Drupal/Tests/Core/Password/PasswordHashingTest

// Run functional tests for views
docker-compose run --user 82 php vendor/bin/phpunit -c core --group views  --verbose --testsuite=functional

// Run unit tests for metatag
docker-compose run --user 82 php vendor/bin/phpunit -c core modules/contrib/metatag --verbose --testsuite=unit

// If you omit the printer command from phpunit.xml, you can call it from the command line.
docker-compose run --user 82 php vendor/bin/phpunit -c core --group views --printer="\Drupal\Tests\Listeners\HtmlOutputPrinter"

```

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
