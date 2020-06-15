# New Relic

Confirm valid tokens in `newrelic/newrelic.ini`

    cp newrelic/newrelic.sample.ini newrelic/newrelic.ini

## Confirm tokens in place

Inside cli container

    php -i | grep "newrelic.license"

## Verify daemon

Inside cli container

    ps -ef | grep newrelic-daemon

## Status

Inside cli container

    /etc/init.d/newrelic-daemon status

## Start / stop / restart

Inside cli container

    /etc/init.d/newrelic-daemon start

    /etc/init.d/newrelic-daemon stop

    /etc/init.d/newrelic-daemon restart

## Container mount

    /newrelic/newrelic.ini => /usr/local/etc/php/conf.d/newrelic.ini

## New relic agent log

    tail -f /var/log/newrelic/php_agent.log

## Configure within Magento

Stores > Configuration > General > New Relic Monitoring