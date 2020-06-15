# Quickstart Guide

## 1 Fetch from git

git clone git@github.com:DominicWatts/Magento2CloudDocker.git ./

## 2 Add the following entry to OS hosts file

    127.0.0.1 magento2.docker
    
## 3 Create magento folder

    mkdir htdocs
    
## 4 Download

Inside `./htdocs`

### 4.1 Hypernode

    wget -qO- https://magento.mirror.hypernode.com/releases/magento2-latest.tar.gz | tar xfz -

Or

    wget -qO- https://magento.mirror.hypernode.com/releases/magento-2.3.4.tar.gz | tar xfz -

### 4.2 Direct Download
 
Download magento from https://magento.com/tech-resources/download
 
## 5 Start Containers

Copy tokens into newrelic.ini

    cp newrelic/newrelic.sample.ini newrelic/newrelic.ini

Start Docker

    docker compose up -d
    
Note: wait for MySQL to initialise if running for first time
 
## 6 Install

### 6.1 CLI

    docker-compose run --rm cli magento-command setup:install --admin-firstname Admin --admin-lastname User --admin-email dominic@xigen.co.uk --admin-user admin --admin-password test123 --base-url http://magento2.docker/ --base-url-secure https://magento2.docker/ --backend-frontname xpanel --db-host db --db-name magento2 --db-user magento2 --db-password magento2 --language en_GB --currency GBP --timezone UTC --use-rewrites 1 --session-save files --use-secure 1 --use-secure-admin 1

### 6.2 Web Setup Wizard

http://magento2.docker/setup/

  - **db host:** db
  - **db name:** magento2
  - **db user:** magento2
  - **db password:** magento2
  
## 7 Tweak

PHP settings can be adjusted via

    `./app/php.ini`
