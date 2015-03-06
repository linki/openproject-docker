# OpenProject Docker

A simple openroject docker image.

## Caveats

* no volume mounts for openproject files and postgres database (should work on linux, but didn't test)
* no reverse-proxy for multiple app containers (therefore, no `fig scale`)

## Usage

First of all we need to build the image.

    docker build -t linki/openproject:simple .

Once that's done we can start a container from the image.
But before running the rails app we need to prepare our database.

Let's fire up a postgres container on our docker host that we can connect to.

    docker run -d --name postgres \
      -e POSTGRES_USER=openproject -e POSTGRES_PASSWORD=openproject postgres:9.4

We use the official postgres 9.4 image and give the container a name for linking later on.
We also create a user with password which we will use when connecting from our openproject container.

Usually rails apps require a couple of rake commands to be issued before we can run the application.
Most of them were already done when building the image, but database related tasks are executed separately.

Let's migrate our database through our openproject image.

    docker run -it --rm --link postgres:database \
      -e DATABASE_URL=postgres://openproject:openproject@database:5432/openproject \
      linki/openproject:simple bundle exec rake db:migrate

    docker run -it --rm --link postgres:database \
      -e DATABASE_URL=postgres://openproject:openproject@database:5432/openproject \
      -e RAILS_ENV=production -e SECRET_TOKEN=foobar \
      linki/openproject:simple bundle exec rake db:seed

Note, that we run `rake db:seed` in the production environment, which results in a minimal seed.
This is a little openproject detail and might be changed in the future.

If you want a more complex seed, run `rake db:seed` in development.
This will add a lot of work packages with lorem ipsum style content.

    docker run -it --rm --link postgres:database \
      -e DATABASE_URL=postgres://openproject:openproject@database:5432/openproject \
      linki/openproject:simple bundle exec rake db:seed

And then finally run the app server in the background.

    docker run -d -p 3000:3000 --link postgres:database \
      -e DATABASE_URL=postgres://openproject:openproject@database:5432/openproject linki/openproject:simple

Voila, now browse to your docker host's ip address on port 3000.
If you're using boot2docker you can probably use `boot2docker.me:3000`.

Login with `admin/admin`

If you want to run it in production just pass the rails env and a secret token.

    docker run -d -p 3000:3000 --link postgres:database \
      -e DATABASE_URL=postgres://openproject:openproject@database:5432/openproject \
      -e RAILS_ENV=production -e SECRET_TOKEN=foobar \
      linki/openproject:simple

The worker process can be run in a separate container.

    docker run -d --link postgres:database \
      -e DATABASE_URL=postgres://openproject:openproject@database:5432/openproject \
      linki/openproject:simple bundle exec rake jobs:work

## Fig

You can do basically the same thing much easier through `fig`.

    # build the openproject image
    fig build

    # migrate the database
    fig run web bundle exec rake db:migrate

    # seed the database
    fig run web bundle exec rake db:seed

    # run database, openproject, and worker process
    fig up -d
