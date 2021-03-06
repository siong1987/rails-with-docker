# Rails with Docker

This is my setup to use Rails together with Docker for local development.  With the release of
[Docker for Mac](https://docs.docker.com/engine/installation/mac/#/docker-for-mac), it is so easy to use it with any local development now.

## Installation

1. First, make sure that you have [Docker for Mac](https://docs.docker.com/engine/installation/mac/#/docker-for-mac) installed.
2. In the project directory, build the `app` container by running:
   `docker-compose build`.
3. Start all the containers by running: `docker-compose up`.
4. Now, you will have a local Rails server up and running at
   [localhost:3000](http://localhost:3000).
5. If you get a missing database error, create the database by running:
   `docker-compose run app rake db:create`.

Notice the pattern of `docker-compose run app rake db:create`, you can
run all commands that way in the main `app` container. I usually just
start with `docker-compose run app bash` and run all my commands via
bash.

## Dockerfile

This is the main docker setup file to run our Rails app container.

The file itself has comments that are pretty self-explanatory. But there
are two things that I would love to point out:

- `BUNDLE_PATH`: This will set the directory where the gems are being installed. We
  make sure that gems are installed on a separated container with a
  shared volumn with the main `app` container.

  This way you don't need to rebuild the `app` container every time you want
  to install a new gem or update an existing gem. You can easily call `docker-compose
  run app bundle install` to install a new gem or `docker-compose run app
  bundle update x` to update an existing gem x.

- `./script/run`: This is a special bash script that utilizes Dockerfile
  [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#/entrypoint) instruction to intelligently decide
  whether to use `bundle exec` for each command that you run on the
  `app` container.

  The idea here is to save that annoying `bundle exec` command that you need
  to type everytime you run a ruby command. This script will first check
  whether all the gems are installed before using `bundle exec`. If not,
  it will just run the command normally without `bundle exec`.

- `./script/start`: When you run `docker-compose up`, this bash script
  will decide whether to install new gems. If there are new gems
  to be installed, install the new gems and start the Rails server. If not,
  start the Rails server immediately.

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

[`.dockerignore`](https://docs.docker.com/engine/reference/builder/#/dockerignore-file) contains a list of files that you want to ignore when
building your docker container. This will make sure that the build
process is efficient without copying unnecessary files into the `app`
container.

## config/environments/development.rb

I added these two lines here to get rid of the annoying `Cannot render
console from...` warning:

```ruby
  # Get rid of the annoying: "Cannot render console from..." warning.
  config.web_console.whitelisted_ips = `ip route|awk '/default/ { print $3 }'`.strip
```

## Contact

You can reach out to me via my gmail address: siong1987@gmail.com or
follow me on [twitter](https://twitter.com/siong1987) for more cool things from me :P

## LICENSE

(The MIT License)

Copyright (c) 2016 Teng Siong Ong

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
