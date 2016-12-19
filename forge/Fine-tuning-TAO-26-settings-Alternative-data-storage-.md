---
tags: Forge
---

Fine tuning TAO 26 settings (Alternative data storage)
======================================================

You may configure TAO to store all the data related to the test delivery into alternative data storage such as Redis.

Redis
-----

### Installing Redis server and PHP redis

-   Make sure you have installed the packages :\
    “redis-server” (on the server you want to use for the storage)\
    “phpredis” on the tao application server

<!-- -->

    sudo apt-get install redis-server

    sudo apt-get install git
    git clone https://github.com/nicolasff/phpredis
    cd phpredis

    sudo apt-get install php5-dev
    phpize
    ./configure [--enable-redis-igbinary]
    make 
    make test
    sudo make install

### Enabling the Redis PHP extension

You still need to enable the module in the PHP config file. To do so, either edit your php.ini or add a redis.ini file in /etc/php5/conf.d with the following contents: extension=redis.so.

### Configuring TAO to store the data into Redis

Once you have installed Redis, you may start to fine tune your TAO setup to store the data into it. You have some flexibility here in terms of what kind of data may be stored into it. Please find all the details [here](resources/http://forge.taotesting.com/projects/tao/wiki/Data_abstractions).

Couchbase
---------

### Installing Couchbase server and couchbase PHP extension

Make sure you have installed the packages :

-   “couchbase-server” (on the server you want to use for the storage)
-   pecl “couchbase” library on the tao application server

<!-- -->

    sudo apt-get install couchbase-server

    sudo pecl install couchbase

You still need to enable the module in the PHP config file. To do so, either edit your php.ini or add a couchbase.ini file in /etc/php5/conf.d with the following contents: extension=couchbase.so.

### Configuring TAO to store the data into Couchbase

Once you have installed Couchbase, you may start to fine tune your TAO setup to store the data into it. You have some flexibility here in terms of what kind of data may be stored into it. Please find all the details [here](resources/http://forge.taotesting.com/projects/tao/wiki/Data_abstractions).

Enabling the Apache mod deflate
-------------------------------

If your connection is the bottleneck, you might benefit from compressing the data using the Apache mod deflate (https://httpd.apache.org/docs/2.2/mod/mod\_deflate.html). A sample [[Apache Module mod\_deflate configuration]]
