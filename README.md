# Rails with Docker

This is my setup to use Rails together with Docker for local development.  With the release of
[Docker for Mac](https://docs.docker.com/engine/installation/mac/#/docker-for-mac), it is so easy to use it with any local setup now.

## Installation

1. First, make sure that you have [Docker for Mac](https://docs.docker.com/engine/installation/mac/#/docker-for-mac).
2. In the project directory, build the `app` container by running:
   `docker-compose build`.
3. Start all the containers by running: `docker-compose up`.
4. Now, you will have a local Rails server up and running at
   [localhost:3000](http://localhost:3000).

## Dockerfile

This is the main docker setup file to run our Rails app in this custom
container.

The file itself has comments that are pretty self-explanatory. But there
are two things that I would love to point up.

- `BUNDLE_PATH`: This will set the directory the gems are being installed. We
  make sure that gems are installed on a separated container with a
  shared volumn with the main `app` container.

  This way you don't need to rebuild the `app` container every time you want
  to install a new gem or update a gem. You can easily call `docker-compose
  run app bundle install` to install a new gem or `docke-compose run app
  bundle update x` to update gem x.

- `./script/run`: This is a special bash script that utilizes Dockerfile
  [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#/entrypoint) instruction to intelligently decide
  whether to use `bundle exec` for each command that you run on the
  `app` container or not.

  The idea here is to save that annoying `bundle exec` typing you need
  to do everytime you run a ruby command. This script will first check
  whether all the gems are installed before using `bundle exec`. If not,
  it will just run the command normally without `bundle exec`.

## docker-compose.yml

This file is pretty self-explanatory too. It defines all the dependent
containers that you want to run together with your main `app` container.

Here I added the `postgres` and `redis` containers to be used together
with the main `app` container.

Notice that the `config/database.yml` is modified to use the `postgres`
container. Here are the changes:

```yaml
default: &default
  adapter: postgresql
  encoding: unicode
  # These two lines below. username and host are both `postgres`.
  username: postgres
  host: postgres
```

Also, the `bundle` container is being used for the shared volume
`/bundle` with the main `app` container.

## .dockerignore

[`.dockerignore`](https://docs.docker.com/engine/reference/builder/#/dockerignore-file) contains a list a files that you want to ignore when
building your docker container.  This will make sure that the build
process is efficient.
