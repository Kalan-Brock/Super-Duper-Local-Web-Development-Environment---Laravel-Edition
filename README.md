# Super Duper Local Web Development Environment

A self provisioning [Vagrant](https://www.vagrantup.com/ "Learn More About Vagrant") box based on Ubuntu 16.04 (ubuntu/xenial64).  No custom boxes, just a ready to go local web development environment you can customize to your heart's content.

## Laravel Edition

This is the same project as the original, only configured specifically for Laravel development.

Original:  https://github.com/Kalan-Brock/Super-Duper-Local-Web-Development-Environment

## Ready to Go

At your first ```vagrant up```, the following is automatically installed and configured.

* Apache (http and https)
* PHP 7.2
* Composer
* Node JS
* NPM
* MySQL (MariaDB 10.3)
* Redis
* Memcached
* Gulp
* Yarn
* Bower
* Sass
* MailHog
* Git
* Vim
* Curl
* Fresh copy of Laravel
* Laravel Debugbar

## Automatic Configuration

- Default MySQL database created and configured to use right away.
- Laravel cron task added (Task Scheduling) - https://laravel.com/docs/5.8/scheduling.
- Laravel Debugbar configuration.
- Site URL
- Debug Mode

## Develop Your Project

The provided public folder is automatically synced to your machine at "/var/www".  This allows you to work on your project with your favorite code editor locally on your computer.

Apache will configure itself to point to /var/www/public/public (Laravel structure)

## Access Your Project

You can access your project from either http://192.168.33.10 or https://192.168.33.10.  

Alternatively, you can set your hosts file to point any domain you want to this ip address.

## Back Up Your Project

```$ vagrant ssh```

```$ projectbackup```

Your databases and site files will be archived and placed in /var/www/backups.

## Connect to MySQL (MariaDB 10.3)

No need for an SSH tunnel.  MySQL (MariaDB 10.3) is configured automatically to allow external connections.

Host: 192.168.33.10 (User: root  Password: root)

## MailHog Interface

If you're developing emails, you can visit the [MailHog](https://github.com/mailhog/MailHog "Learn More About MailHog") interface at http://192.168.33.10:8025.

## Want Something Different?

You're not tied to a custom box with Super Duper Local Web Development Environment.  Simply change the provisioning in Vagrantfile.




