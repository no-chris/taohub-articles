<!--
parent: 'Administrator Guide'
created_at: '2013-06-21 10:05:44'
updated_at: '2015-10-13 16:53:52'
authors:
    - 'Joel Bout'
tags:
    - 'Administrator Guide'
-->

Nginx
=====

Nginx is not officially supported by Tao. The following instructions are for experiments only:

Nginx and Tao 3.1
-----------------

Tao now supports a single entry point simplifying the config even further:

    location ~* ^/([^//]*)/views/.* {
    }

    location /tao/install {
    }

    location /tao/getFile.php {
        rewrite  ^(.*)$ /tao/getFile.php last;
    }

    location /favicon.ico {
    }

    location / {
        rewrite  ^  /index.php;
    }

### Known issues

-   max upload filesize is not detected correctly from nginx client_max_body_size configuration

Nginx and Tao 3.0
-----------------

    location ~* ^/([^//]*)/(views|locales)/.* {
    }

    location /tao/install {
    }

    location /tao/getFile.php {
        rewrite  ^(.*)$ /tao/getFile.php last;
    }

    location / {
        rewrite  ^/([^//]*)/.*$  /$1/index.php;
    }

### Known issues

-   max upload filesize is not detected correctly from nginx client_max_body_size configuration
-   HTTP basic authentication not working for REST api

Nginx and Tao 2.6
-----------------

Uses the same configuration as 3.0

### Known issues

-   item preview requires Tao <br/>
>= 2.6.7 to work

Nginx and Tao 2.4
-----------------

If you try to get Tao 2.4 to work with Nginx, you might want to:

\* removed the installation check: tao_install_checks_ModRewrite

\* in tao_helpers_class_I18n remove the double slash

\* convert the .htaccess rules in the Nginx syntax. Here’s an incomplete example:

    location ~* ^/([^//]*)/(views|test|locales)/.* {
    }

    location /tao/install {
    }

    location /taoDelivery/data/compiled {
    }

    location / {
        rewrite  ^/([^//]*)/.*$  /$1/index.php;
    }

-   production mode will not fully work


