# UT Drupal Kit Local Development Tooling

This repository contains Docksal-based tooling for internal development of the UT Drupal Kit. It should **not** be used for local development with an individual Drupal Kit site.

For local development on an individual Drupal Kit site, use one of the options listed at https://drupalkit.its.utexas.edu/docs/getting_started/local_development.html

## For ITS

## Getting started
* Make sure you have [Docksal installed](https://docs.docksal.io/getting-started/setup/)
* Make sure you have a recent version of PHP as your active native php-cli version. If using a Mac with Homebrew, you can do this with https://github.com/shivammathur/homebrew-php
* From the root of where you have cloned the repo, run `fin init`. This will initiate the containers.
* You can then use `fin init-site` to install Drupal.
* If you need to re-install the site, run `fin init-site` as needed without restarting the stack or re-running any of `setup.sh`.
* You can run `fin uli` to do a `drush uli` with the appropriate URI automatically piped in to give you a valid link for admin login.

### Special note: Firefox
If you use Firefox, you will need to install `mkcert` in order for the browser to recognize locally-generated SSL certificates. Run the following:

```
brew install mkcert
brew install nss
mkcert -install
```

## Running tests
* For first time setup, run `fin test-init`. This will copy over the default files from the `.docksal/drupal/testing-defaults` folder, and update them to match your current Docksal virtual host.
* There are 5 commands to run testing:
  - `run-tests`: run the entire UTexas suite of tests using parallel testing
  - `test <directory/file>`: run PHPUnit Functional tests in series, using standard PHP command parameters
  - `test-js <directory/file>`: run PHPUnit FunctionalJS tests in series, using standard PHP command parameters
  - `paratest-functional <directory/file>`: run PHPUnit Functional tests in parallel, using standard PHP command parameters
  - `paratest-js <directory/file>`: run PHPUnit FunctionalJS tests in parallel, using standard PHP command parameters
* More details on the commands can be found in the command files themselves - `.docksal/commands/test` and `.docksal/commands/test-js`. They both function largely the same way, but are configured to use corresponding `phpunit.xml` files located in `.docksal/drupal/testing` folder.
* To ensure proper functionality, the `SIMPLETEST_BASE_URL` has to be updated to match the Docksal virtual host name. In most cases, this is taken care of when running `fin test-init`. As part of this command, it will update the appropriate `phpunit.xml` files automatically. This does a basic find and replace operation via perl, replacing the default `web` string with the Docksal variable `${VIRTUAL_HOST}`.
* If the command doesn't work for you for some reason, you can manually update the `SIMPLETEST_BASE_URL` in the included `phpunit.xml` and `phpunit-js.xml`, located in `.docksal/drupal/testing/`. In Docksal, the name of your host matches the name of your folder. So, if you cloned this into a folder called `drupalin`, your Docksal based URL will be `http://drupalin.docksal`, and this is what your `SIMPLETEST_BASE_URL` should be set to.

## Deprecation checking
Run the following command to run Drupal 9 compatibility checks with [drupal-check](https://github.com/mglaman/drupal-check):

```
vendor/bin/drupal-check path/to/directory
```

## Deprecation fixes via Drupal Rector

Run the following commands to attempt to automate Drupal 9 compatibility fixes with [drupal-rector](https://github.com/palantirnet/drupal-rector)

0. Add the configuration file to the project root

```
cp vendor/palantirnet/drupal-rector/rector-config-web-dir.yml rector.yml
```

1. Analyze the code

```
vendor/bin/rector process web/modules/contrib/[YOUR_MODULE] --dry-run
```

2. Apply suggested changes

```
vendor/bin/rector process web/modules/contrib/[YOUR_MODULE]
```


## Setting different versions of PHP
* The `docksal.env` file contains some sample definitions of the different pieces of the stack. `WEB_IMAGE`, `DB_IMAGE`, and `CLI_IMAGE` can be modified to use different Docker images from the Docksal repo. The PHP version is determined from the `CLI_IMAGE`. You can modify the value to a new version, and run `fin up` to refresh the containers. Docksal will detect the changes, download the new Docker image if it doesn't exist in your cache, and reload your container running the new PHP version. So for example, to run on PHP 7.3, you could update the `CLI_IMAGE` to be `CLI_IMAGE='docksal/cli:2.6-php7.3'`, and run `fin up`, and now the CLI container is running on PHP 7.3. And to clarify, the CLI container is determining what version of PHP that Drupal will be using, since Drupal is in essence running on this container, in orchestration with the `WEB_IMAGE` and `DB_IMAGE` containers, which are networked together automatically behind the scenes. [See more information about this from the Docksal site](https://docs.docksal.io/service/cli/settings/).

## Updating your local version of PHP with Homebrew
* Given the [EOL of PHP versions](https://www.php.net/supported-versions.php), it is recommended to upgrade to PHP 7.3 at the very least. [Homebrew](https://brew.sh/) will help with this task, being a Mac/Linux package manager. You may verify which version php you are using by running `php --version` and you may verify which specific binary you are using by running `which php`, if your binary path shows `Cellar` as part of the path, you already are using Homebrew, if not, refer to [this page](https://docs.brew.sh/Installation) for how to install the tool. Once installed, you will have to login as a super user (sudo is not supported) and run the command `brew install php@7.3`, more php versions can be found [here](https://formulae.brew.sh/formula/php). By default on Mac, Homebrew will install the packages within `/usr/local/Cellar/`, for instance, I got this package installed: `/usr/local/Cellar/php\@7.3/7.3.19/`, which you will have to add to your `PATH` environment variable in `.bash_profile` and export it. You will add something like the following to the end of your file: `export PATH="/usr/local/Cellar/php@7.3/7.3.19/bin:$PATH"`. For more information on how to update your environment variables, [here is an article](https://imstudio.medium.com/path-macos-best-practice-for-path-environment-variables-on-mac-os-35ec4076a486).


## Accessing database through PhpMyAdmin
* You can access PhpMyAdmin by navigating to `http://pma.{YOUR_DOCKSAL_SITE}`, replacing `{YOUR_DOCKSAL_SITE}` with whatever the site URL is.

## Accessing database through SequelPro
* If you have SequelPro installed, you can run `fin sequelpro` from the docroot of the site, and it will open up the database in SequelPro!

## Accessing MailHog
* You can access MailHog by navigating to `http://mail.{YOUR_DOCKSAL_SITE}`, replacing `{YOUR_DOCKSAL_SITE}` with whatever the site URL is.

## Overriding PHP / MySQL settings
* It's possible to override PHP settings as needed. Some examples are included in the `example-php-mysql-overrides` folder.
* Docksal will look for PHP overrides in `.docksal/etc/php` - for more in depth information, see [the Docksal documentation](https://docs.docksal.io/service/cli/settings/)
* Docksal will look for MySQL overrides in `.docksal/etc/mysql` - for more in depth information, see [the Docksal documentation](https://docs.docksal.io/service/db/settings/)

## XDebug
This should provide "out of the box" xdebugging ability
Test steps:
* Add `XDEBUG_ENABLED=1` to your `docksal-local.env`, and do a `fin up` to reload the containers with XDebug enabled. If you haven't already setup a `php.ini`, add one, or update `/etc/php/php.ini` settings with desired XDebug preferences (see `.docksal/example-php-mysql-overrides/etc/php/php.ini`), and do a `fin project restart` to apply the new settings. *NOTE*: Having `XDEBUG_ENABLED=1` will slow down performance, so consider setting back to `0` and running `fin up` to unload it if not actively using.
* Download [VSCode debug extension](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug)
* Go to the "debug" tab and start debugging:
![image](https://media.github.austin.utexas.edu/user/68/files/3fc12580-6cd4-11e9-9ecc-98090c974553)
* When you refresh the site in a spot where that code executes, it should jump you into vscode
![image](https://media.github.austin.utexas.edu/user/68/files/7e56e000-6cd4-11e9-90f0-58c9ac48b006)

# Note on composer.json requirements
* Due to a current limitation with our composer scaffolding, any package that is not the root package will not have it's `require-dev` dependencies installed. Because we use the utexas_profile as a package, the `require-dev` packages defined there won't be installed, since the `utexas_profile` is not the *root* package. That is why we duplicate them in this repo as *`requirements`*, so that when this package is installed, it takes care of properly installing dependencies for testing and other dev specific activities.


