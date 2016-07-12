# Rails with Docker

This is my setup to use Rails together with Docker for local development.  With the release of
[Docker for Mac](https://docs.docker.com/engine/installation/mac/#/docker-for-mac), it is so easy to use it with any local setup now.

## .dockerignore

[`.dockerignore`](https://docs.docker.com/engine/reference/builder/#/dockerignore-file) contains a list a files that you want to ignore when
building your docker container.  This will make sure that the build
process is efficient.
