# Docker WordPress Environment

[![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Release Version](https://img.shields.io/github/release/sarahcssiqueira/docker-wordpress.svg)](https://github.com/sarahcssiqueira/docker-wordpress/releases/latest)
[![Support Level](https://img.shields.io/badge/support-may_take_time-yellow.svg)](#support-level)

This is a **Docker WordPress Environment** focused on the development of **WordPress plugins and themes**, supports [WP-CLI](https://wp-cli.org/), [phpMyAdmin](https://www.phpmyadmin.net/), [Xdebug](https://xdebug.org/) and [WordPress Coding Standards - WPCS](https://github.com/WordPress/WordPress-Coding-Standards).

---

> **Disclaimer:** This project is meant to be used for development purposes only. It's **not** meant to be used in production.

---

## Table of Contents

- [Introduction](#docker-wordpress-environment)
- [Requirements](#requirements)
- [Docker](#docker)
- [Folders Structure](#folders-structure)
- [Containers](#containers)
- [Basic Usage](#basic-usage)
  - [Set variables](#variables)
  - [Starting Docker](#starting-docker)
- [More Tools](#more-tools)
  - [Set debugging in WordPress](#set-debugging-in-wordPress)
  - [Xdebug for VSCode](#xdebug-for-vscode)
  - [Set WordPress Coding Standards](#set-wordpress-coding-standards)
  - [Using WordPress Coding Standards](#using-wordpress-coding-standards)
- [Contributing and Support](#contributing-and-support)

## Requirements

- [Docker](https://www.docker.com/)
- [VS Code](https://code.visualstudio.com/)
  - [PHP Debug Extension](https://marketplace.visualstudio.com/items?itemName=xdebug.php-debug)
  - [phpcs](https://github.com/sarahcssiqueira/docker-wordpress/blob/master/.vscode/settings.json)
- [Composer](https://getcomposer.org/)
- [Xdebug](https://xdebug.org/docs/install)

## Docker

### Folders Structure

![docker folder](https://github.com/sarahcssiqueira/docker-wordpress/assets/82296194/eef4010b-9c13-49a8-970b-1a707666defd)

- .vscode
- db
- docker: also install Xdebug and set Xdebug.ini.
- plugins
- themes
- wp-core
- .gitignore
- composer.json
- license
- readme

### Containers

Docker containers used on [docker-compose. yml](https://github.com/sarahcssiqueira/docker-wordpress/blob/master/docker/docker-compose.yml):

- WordPress: Uses the official WordPress latest docker image.
- WP-CLI: The **wpcli container** was added to only run one-off commands. Don’t need it to run as a service, only as a cli tool, for that run `docker-compose run --rm wpcli command`
- db: The MYSQL official image.
- phpMyAdmin: Intended to handle the administration of MySQL, in this case through port 80, on browser [localhost:8080](localhost:8080).

## Basic usage

Clone (or download) this repository `git clone https://github.com/sarahcssiqueira/docker-wordpress` or download one of the [latest releases](https://github.com/sarahcssiqueira/docker-wordpress/releases).
Go to the `/docker` folder and set up your chosen variables on the .env file.

### Set variables

The variables used for WordPress installation are the following ones, you can use the same or change them, it's up to you.

```
MYSQL_DATABASE_NAME=exampledatabase
MYSQL_USER=exampleuser
MYSQL_PASSWORD=examplepass
```

---

> WARNING: DO NOT store your variables on a .env file, a better approach would be to add the .env file to a .gitignore and store the variables as [encrypted secrets.](https://docs.github.com/en/actions/security-guides/encrypted-secrets).

---

### Starting Docker

After setting your variables, run `docker-compose up -d` to start the Docker.

When Docker finishes their work (can take a few minutes at the first time, depending on your connection and machine), you will see the WordPress default installation screen in your browser.

Finish WordPress installation.

![wp-setup-start](https://github.com/sarahcssiqueira/docker-wordpress/assets/82296194/0ac267b6-b2a6-44c2-aa29-a4092d62b80d)

In your terminal, cd the root project folder again to start working on your plugins and/or themes.

For using wp-cli, use `docker-compose run --rm wpcli command` as it was added to only run one-off commands.

🚀 You already can start to work on your themes and plugins, but there are more tools available for using, keep reading.

## ⚙️ More Tools

### Debugging in WordPress

The wp-core is synchronized through the docker-composer.yml just in case you need to check and/or debug something. After running `docker-compose up -d`, you will be able to change the [WordPress debug mode](https://wordpress.org/documentation/article/debugging-in-wordpress/) by changing the following lines on the **wp-config.php** file inside the wp-core folder.

By default, the wp-config.php generated by Docker will look something like this:

```
define( 'WP_DEBUG', !!getenv_docker('WORDPRESS_DEBUG', '') );
```

Replace it with the following:

```
define( 'WP_DEBUG', true);

define( 'WP_DEBUG_LOG' , true );

define ( 'WP_DEBUG_DISPLAY', true );
```

More info about **debugging in WordPress** you can find [here](https://wordpress.org/documentation/article/debugging-in-wordpress/).

### Xdebug for VSCode

Xdebug will be installed by the docker file as you can see in

```
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
```

Xdebug is enabled and configured to work on VSCode using the extension PHP Debug, [install it](https://marketplace.visualstudio.com/items?itemName=xdebug.php-debug). The settings are already configured according to the extension documentation, in the **launch.json** file on the **.vscode folder**, including the pathMapping for remote host debugging.

```
"pathMappings": {
"/var/www/html": "${workspaceRoot}/wp-core",
                "/var/www/html/wp-content/plugins": "${workspaceRoot}/plugins",
                "/var/www/html/wp-content/themes": "${workspaceRoot}/themes"
}
```

### Set WordPress Coding Standards

There are several ways to config WordPress Coding Standards, but in this environment, we will manage it through Composer. To use WordPress Coding Standards we also need to install PHP Code Sniffer.
If you inspect the composer.json on the root folder of the project, you will see the lines:

```
"require-dev": {
    "squizlabs/php_codesniffer": "^3.7",
    "wp-coding-standards/wpcs": "^2.3",
}
```

They are responsible to install PHP Code Sniffer and WordPress Coding Standards when you run:

`composer install`

After that, Composer will download the necessaire dependencies to the vendor folder. To check if it works, run:

`./vendor/bin/phpcs -i`

On the first time, the expected output will be:

_The installed coding standards are MySource, PEAR, PSR1, PSR2, PSR12, Squiz, and Zend_

To use WPCS, we have "to tell" the PHP Code Sniffer about the WordPress Coding Standards. For that, run the command:

`./vendor/bin/phpcs --config-set installed_paths vendor/wp-coding-standards/wpcs`

To confirm if it works, run `./vendor/bin/phpcs -i ` again, and the expected output this time should be:

_The installed coding standards are MySource, PEAR, PSR1, PSR2, PSR12, Squiz, Zend, WordPress, WordPress-Core, WordPress-Docs, and WordPress-Extra_

This way, WordPress Coding Standards are available for all plugins and themes you work with inside this Docker environment.

### Using wpcs

For VS Code **"to understand the standards"**, we need some extension, my chosen one was the phpcs. Install it. The settings are already configured according to the phpcs extension documentation, in the **settings.json** on the **.vscode folder** in this repository.

If you installed everything correctly because the `"phpcs.lintOnSave": true,` line on settings.json, when you hit save any PHP file, it will be linted correctly and the warnings/errors will be displayed on the problems tab of your IDE aka VS Code. We can also run the following commands on the terminal to check a single file:

`./vendor/bin/phpcbf --standard="WordPress" /file-name.php`

To display a report on the terminal:

`./vendor/bin/phpcs --standard="WordPress" /file-name.php`

We can also run the commands to check all the files at once in the project but may experience performance issues depending on your project size, may be even necessaire to adjust our php.ini directives like the max_execution_time. **For that reason, use the following commands carefully**.

`./vendor/bin/phpcbf --standard="WordPress" .`

To display a report on the terminal:

`./vendor/bin/phpcs --standard="WordPress" .`

# 🤝 Contributing and Support

It's **work in progress.** Feel free to contribute informing about [issues](https://github.com/sarahcssiqueira/docker-wordpress/issues) or even through [pull requests](https://github.com/sarahcssiqueira/docker-wordpress/pulls) for improvements.

❤️ Support [buying me a coffee](https://www.buymeacoffee.com/sarahcssiqueira).
