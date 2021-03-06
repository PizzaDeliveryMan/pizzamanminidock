# Dockerhero

## Version 1.3.2

### What is Dockerhero?

Dockerhero is a local development tool. Out of the box, it should only take a "docker-compose start" to get all your local PHP projects working. Yes, all of them. At the same time.

It has support for Laravel, Codeigniter, Wordpress and other PHP projects.

The goal is also to make it customizable. You can easily add your own nginx configurations, cronjobs and via phpMyAdmin you can create your own databases.

Dockerhero includes the following software (containers):

- nginx (latest)
- mySQL (5.7)
- Redis (latest)
- PHP (7.1-fpm by default, [or choose a different version](#picking-a-php-version))
- Mailhog
- and more to come!

Dockerhero includes the following useful tools:

- phpMyAdmin
- phpRedisAdmin
- Cron
- Mailhog
- Composer
- Xdebug
- NVM (the default node version is 8)
- NPM
- Yarn
- Gulp
- Bower
- Vue-cli
- Laravel Artisan autocompletion
- Laravel Dusk support
- laravel-dump-server support
- and more to come!

Localtest.me is used to make everything work without editing your hosts file! Just like magic!

## Table of Contents

1. [Installation](#installation)
    1. [Docker and Docker Compose](#docker-and-docker-compose)
    2. [Folder placement](#folder-placement)
    3. [Picking a PHP version](#picking-a-php-version)
    4. [Trusting the self-signed certificate](#trusting-the-self-signed-certificate)
2. [Updating](#updating)
    1. [Dockerhero itself](#dockerhero-itself)
    2. [Dockerhero images](#dockerhero-images)
3. [Usage](#usage)
    1. [Starting](#starting)
    2. [Stopping](#stopping)
    3. [Laravel public folder](#laravel-public-folder)
    4. [Private composer packages](#private-composer-packages)
4. [Databases](#databases)
    1. [MySQL](#mysql)
        1. [Changing the MySQL version](#changing-the-mysql-version)
        2. [Upgrading MySQL](#upgrading-mysql)
        3. [Changing the SQL mode](#changing-the-sql-mode)
    2. [Redis](#redis)
5. [CLI Access](#cli-access)
6. [Custom nginx configs](#custom-nginx-configs)
7. [Cronjobs](#cronjobs)
8. [Mailhog](#mailhog)
9. [Overriding default settings](#overriding-default-settings)
10. [Connecting from PHP to a local project via URL](#connecting-from-php-to-a-local-project-via-url)
11. [Making a local website publicly available](#making-a-local-website-publicly-available)
12. [Miscellaneous](#miscellaneous)
    1. [Laravel Dusk](#laravel-dusk)
    2. [laravel-dump-server](#laravel-dump-server)
13. [Known issues](#known-issues)
    1. [MacOS](#macos)
14. [Contributing](#contributing)
15. [Thank you](#thank-you)
16. [Project links](#project-links)
17. [Todo](#todo)

## Installation

### Docker and Docker Compose

Follow the instructions on the docker website to install [docker](https://docs.docker.com/engine/installation/) and [docker-compose](https://docs.docker.com/compose/install/).

### Folder placement

Next, it is essential to make sure Dockerhero is inside the folder containing all the projects you wish to use with Dockerhero. So if you want `https://mysuperproject.localtest.me` to be accessible, place and run Dockerhero inside the same folder `mysuperproject` is located. For example, if the path to `mysuperproject` is: `/home/john/webdev/mysuperproject` - Dockerhero needs to be located in `/home/john/webdev/dockerhero`.

This is because dockhero mounts its parent folder (`./../`) as `/var/www/projects/`, which is the location nginx will look for when it receives a request on `http://*.localtest.me`

_Remember: anything you do inside the container is deleted upon closing docker! Only changes to mounted folders (like your projects, databases) are persisted because those changes are actually done on your system._

### Picking a PHP version
By default, PHP 7.1 is active. If you would like to change this to another version, you can do so by overriding the option using the `docker-compose.override.yml` to change image.

For more information, please see this section: [Overriding default settings](#overriding-default-settings)

Available PHP Images:

PHP 7.3: `johanvanhelden/dockerhero-php-7.3-fpm:latest`

PHP 7.2: `johanvanhelden/dockerhero-php-7.2-fpm:latest`

PHP 7.1: `johanvanhelden/dockerhero-php-7.1-fpm:latest`

PHP 5.6: `johanvanhelden/dockerhero-php-5.6-fpm:latest`

PHP 5.4: `johanvanhelden/dockerhero-php-5.4-fpm:latest`


### Trusting the self-signed certificate

Dockerhero has full support for https. This is done with a self-signed certificate. In order to skip the warning in your browser, you can trust the certificate by importing it in the browser or keychain. The certificate can be found [here](https://github.com/johanvanhelden/dockerhero-nginx/blob/master/.certs/localtest.me.crt).

This part is however entirely optional, and you do not have to do this. You can simply ignore the browser warning and continue.

## Updating

### Dockerhero itself

Simply download or pull the latest release from [GitHub](https://github.com/johanvanhelden/dockerhero) and [update the images](#updating-images).

### Dockerhero images

To ensure you have the latest images, you can run `docker-compose pull` in the Dockerhero folder.

## Usage
### Starting
`$ cd` into the Dockerhero folder on your local machine and execute:

```
$ docker-compose up
```

This will give you real-time log information and useful when debugging something. If anything fails, you can simply `ctrl-c` docker and it will shut down.

If you would rather prefer to run everything in the background, use:

```
$ docker-compose start
```

### Stopping

To stop the containers, simple stop the `docker-compose up` process using `ctrl-c`.

If you had it running in the background, you can use:

```
$ docker-compose stop
```

### Laravel public folder
If you are working with a default Laravel project, you will probably get a `File not found.` error. This is because Laravel uses a `public` folder instead of a `public_html` folder. Dockerhero is configured out-of-the-box to use a `public_html` as the root.

There are a few ways to solve this:
- Either reconfigure Laravel to use a public_html folder, for instructions, view this [Gist](https://gist.github.com/johanvanhelden/29096a9ff31eac9b039d36056919abbb#file-reconfigure-laravel-instructions-md)
- Or create a symbolic link, for instructions, view this [Gist](https://gist.github.com/johanvanhelden/29096a9ff31eac9b039d36056919abbb#file-symlink-instructions-md)
- Or add a custom nginx vhost to use the public folder, for instructions, view this [Gist](https://gist.github.com/johanvanhelden/29096a9ff31eac9b039d36056919abbb#file-vhost-instructions-md)

### Private composer packages
If you need to access private composer packages, you might want to link your local `/home/username/.composer` folder (containing your auth.json file) to Dockerhero. You can do so by adding a new volume to the workspace image in your `docker-compose.override.yml`  (if you do not have one,
[please create it](#overriding-default-settings)) like so:

```
version: '2'

services:
  workspace:
    volumes:
      - /home/username/.composer:/home/dockerhero/.composer
```

You will now be able to install and update private packages inside Dockerhero.

## Databases

### MySQL

Via phpMyAdmin you can create new databases and users. The database host you would need to use in your projects would be:

```
mySQL host: dockerhero_db
mySQL port: 3306
```

You can visit phpMyAdmin by going to `https://phpmyadmin.localtest.me`

If you want to import databases from the file system, place them in `./databases/upload`.

Any exported databases to the file system can be found in `./databases/save`

#### Changing the MySQL version
If you would like to change the MySQL version, you can do so by editing the `docker-compose.override.yml` (if you do not
have one, [please create it](#overriding-default-settings)) like so:
```
version: '2'

services:
  db:
    image: mysql:5.6
```

#### Upgrading MySQL
If you changed the MySQL image to a newer version, it will be necessary to upgrade your current databases.
You can do so by logging into the database container and running the `mysql_upgrade` command, like so:
`docker exec -it dockerhero_db bash`

Once inside the database container you need to run the following command:
`mysql_upgrade -u root -pdockerhero`

After the upgrade is done, please restart Dockerhero.

#### Changing the SQL mode
By default, I've set the same SQL mode as MySQL 5.6 to ensure maximum backwards compatibility. If you would like to
set it to the 5.7 default setting, you can do so by editing the `docker-compose.override.yml` (if you do not have one,
[please create it](#overriding-default-settings)) like so:
```
version: '2'

services:
  db:
    command: --sql_mode="ONLY_FULL_GROUP_BY"
```

### Redis

In order to use Redis in your projects, you need to define the following host:

```
Redis host: dockerhero_redis
Redis port: 6379
```

You can visit phpRedisAdmin by going to `https://phpredisadmin.localtest.me`

## CLI Access

You can enter the bash environment of the containers by executing:

```
docker exec -it --user=dockerhero dockerhero_workspace bash
```

All projects are available in `/var/www/projects/`

You can replace `dockerhero_workspace` with any container name. The `--user=dockerhero` part is needed to prevent files from being generated with the root user and group. You will need to leave out this argument for other containers.

When you enter the bash environment, you will be starting in `/var/www/projects`

### Artisan autocompletion

If you are inside a Laravel folder, you can type `artisan` (instead of `./artisan` or `php artisan`) and tab to autocomplete.

### Protip! Make yourself a bash alias!

Make your life easier and create a function in your ~/.bash_aliases file like so:

```
sshDockerhero() {
  docker exec --user=dockerhero -it dockerhero_workspace bash
}
```

Now, in a new terminal, you can simply execute `sshDockerhero` and you will be inside the container.

## Custom nginx configs

You can place your own `*.conf` files into the `nginx/conf` folder. They will be automatically included once the container starts.

## Cronjobs

Create a new file in `./crons/` called `crons`. In this file, define all the cron lines you want. For an example, see the `./crons/crons.sample` file.

## Mailhog

All outgoing mail is caught by default. You do not need to configure anything. To view the e-mail that has been send, visit the [Mailhog GUI](http://localhost:8025)

For some reason, this autocatching does not work properly with Laravel artisan commands (like queue workers). In order to make it work, you can set the .env settings like so:

```
MAIL_DRIVER=smtp
MAIL_HOST=dockerhero_mail
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

## Overriding default settings
You can create a brand new `docker-compose.override.yml` in the root of Dockerhero to override default settings or customize things.
It might look a bit like this:

```
version: '2'

services:
  php:
    image: johanvanhelden/dockerhero-php-7.2-fpm:latest
    extra_hosts:
      - "projectname.localtest.me:172.18.0.6"
  workspace:
    extra_hosts:
      - "projectname.localtest.me:172.18.0.6"
```

## Connecting from PHP to a local project via URL

Add the following entry to the `docker-compose.override.yml` file in the `php:` section:

```
extra_hosts:
  - "projectname.localtest.me:172.18.0.6"
```
Where 172.18.0.6 is the IP of the dockerhero_web container. To find the IP address you could use:

`$ docker inspect dockerhero_web | grep IPAddress`

Now, if PHP attempts to connect to projectname.localtest.me, it will not connect to his localhost, but to the nginx container.

## Making a local website publicly available

If you are developing for an API, webhook or if you want to demonstrate something to someone, it can be extremely useful to forward your local website to the public internet.

In order to do this:

- Download ngrok from: <https://ngrok.com/>
- Extract the zip file
- Run the following command from the command line:

`$ ./ngrok http 127.0.0.1:80 -host-header=project.localtest.me`

Where the host-header flag contains the URL of the project you would like to forward.

Ngrok will now present you with a unique ngrok URL. This is the URL you can give out to clients or use in the API/webhook settings.

## Miscellaneous

### Laravel Dusk

In order to make Laravel Dusk work, you need to add your Laravel project URL to the "extra_hosts" section of the docker compose workspace section, as explained in the "[Connecting from PHP to a local project via URL](#connecting-from-php-to-a-local-project-via-url)" section.

### laravel-dump-server

[laravel-dump-server](https://github.com/beyondcode/laravel-dump-server) is a great package that allows you to capture dump contents so that it does not interfere with HTTP / API responses.

In order to make it work with dockerhero, simply override the config and point it to the workspace container, like so:
```
'host' => 'tcp://dockerhero_workspace:9912',
```

Next, ssh into to workspace image, and simply run: `$ artisan dump-server` and start dumping to your heart's content.

## Known issues

### MacOS
On MacOS there is an issue with linking the timezone. I do now own a Mac myself, so I am unable to produce a proper solution, but for now, I suggest you timezone links from the `volumes:` section for each container (`workspace`, `php`, `web`, `db`) that links the time-zones. If you are someone who owns a Mac, please let me know how I can properly address this, if you can.

## Contributing

Feel free to send in pull requests! Either to the image repos or the Dockerhero repo itself. Do keep the following in mind:

- Everything needs to be as generic as possible, so do not try and add something that is super specific to your own use that no-one else will use.
- Everyone needs to be able to use it out of the box, without additional configuration. However, it is fine if a feature would be disabled without configuring. As long as users can still just clone the project and "go".
- If something needs documentation, add it to the readme.md.
- Test, test and test your changes.

## Thank you

- **localtest.me** - a big thank you goes out to localtest.me for providing a domain that points to 127.0.0.1\. You can visit their website [here](http://readme.localtest.me/)
- **LaraDock** - also a huge shout out to LaraDock for providing me with a lot of sample code and inspiration. You can visit their GitHub page [here](https://github.com/LaraDock/laradock).

## Project links

- [Dockerhero - GitHub](https://github.com/johanvanhelden/dockerhero)
- [Dockerhero - Workspace GitHub](https://github.com/johanvanhelden/dockerhero-workspace)
- [Dockerhero - Nginx GitHub](https://github.com/johanvanhelden/dockerhero-nginx)
- [Dockerhero - PHP 7.3-fpm GitHub](https://github.com/johanvanhelden/dockerhero-php-7.3-fpm)
- [Dockerhero - PHP 7.2-fpm GitHub](https://github.com/johanvanhelden/dockerhero-php-7.2-fpm)
- [Dockerhero - PHP 7.1-fpm GitHub](https://github.com/johanvanhelden/dockerhero-php-7.1-fpm)
- [Dockerhero - PHP 5.6-fpm GitHub](https://github.com/johanvanhelden/dockerhero-php-5.6-fpm)
- [Dockerhero - PHP 5.4-fpm GitHub](https://github.com/johanvanhelden/dockerhero-php-5.4-fpm)

## Todo

- Make the timezone a setting that can be overwritten when starting containers
- Set up a GitHub page
