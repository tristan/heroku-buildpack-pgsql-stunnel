# Heroku buildpack: pgsql-stunnel

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks)
that allows one to connect to a postgresql instance that requires ssl with a
base system that doesn't support ssl.

The primary use case for this buildpack was for deploying a system running
[asyncpg](https://github.com/MagicStack/asyncpg) `0.5.4` with python `3.5`
which at the time of writing did not have support for connecting to postgresql
with ssl.

## Usage

First you need to set this buildpack as your initial buildpack with:

```console
$ heroku buildpacks:add -i 1 https://github.com/tristan/heroku-buildpack-pgsql-stunnel.git
```

Then confirm you are using this buildpack as well as your language buildpack
like so:

```console
$ heroku buildpacks
=== frozen-potato-95352 Buildpack URLs
1. https://github.com/tristan/heroku-buildpack-pgsql-stunnel.git
2. heroku/python
```

For more information on using multiple buildpacks check out
[this devcenter article](https://devcenter.heroku.com/articles/using-multiple-buildpacks-for-an-app).

The buildpack currently only supports one database and the client must connect
to a specific port set by stunnel. Set your database configuration `host`
variable to: `/tmp/.s.PGSQL.6101`

Next, for each process that should connect to Postgresql securely, you will need
to preface the command in your `Procfile` with `bin/start-stunnel`. In this
example, we want the `web` process to use a secure connection to Heroku
Postgresql:

    $ cat Procfile
    web:    bin/start-stunnel python -m toshiapp --port=$PORT --config=$CONFIGFILE

We're then ready to deploy to Heroku with an encrypted connection between the dynos and Heroku
Redis:

    $ git push heroku master


## Options

`PGSQL_STUNNEL_ENABLED` : Set to `0` to disable the stunnel (default `1`)

`ENABLE_STUNNEL_AMAZON_RDS_FIX` : Enable this option to prevent stunnel failure with Amazon RDS when a dyno resumes after sleeping

`STUNNEL_LOGLEVEL` : `debug` config option for stunnel (default `notice`).

`STUNNEL_CONNECTION_RETRY` : `retry` config option for stunnel (default `no`).
