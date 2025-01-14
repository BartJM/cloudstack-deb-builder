# CloudStack DEB Package builder using Docker

[![Build Status](https://github.com/BartJM/cloudstack-deb-builder/workflows/ci/badge.svg)](https://github.com/BartJM/cloudstack-deb-builder/actions)
[![Docker Pulls](https://img.shields.io/docker/pulls/scclouds/cloudstack-deb-builder.svg)](https://store.docker.com/community/images/scclouds/cloudstack-deb-builder)
[![license](https://img.shields.io/github/license/scclouds/cloudstack-deb-builder.svg)](https://github.com/scclouds/cloudstack-deb-builder/blob/master/LICENSE)

Docker images for building Apache CloudStack DEB packages.

This will give portable, immutable and reproducable mechanism to build packages
for releases. A very good candidate to be used by the Jenkins slaves of the project.

## Table of Contents

- [Supported tags and respective `Dockerfile` links](#supported-tags-and-respective-dockerfile-links)
- [Packges installed in conatiner](#packges-installed-in-conatiner)
- [Build DEB packages](#build-deb-packages)
  - [Pull Docker images](#pull-docker-images)
  - [Build local repository](#build-local-repository)
    - [Clone Apache CloudStack source code](#clone-apache-cloudstack-source-code)
    - [Build packages of local repository](#build-packages-of-local-repository)
  - [Build remote repository](#build-remote-repository)
    - [Build packages of remote repository](#build-packages-of-remote-repository)
- [Building tips](#building-tips)
  - [Maven cache](#maven-cache)
  - [Adjust host owner permission](#adjust-host-owner-permission)
- [Builder help](#builder-help)
- [License](#license)

## Supported tags and respective `Dockerfile` links

- [`latest`, `ubuntu2004` (ubuntu2004/Dockerfile)][ubuntu2004-dockerfile]
- [`ubuntu2004` (ubuntu2004/Dockerfile)][ubuntu2004-dockerfile]
- [`ubuntu1804-jdk11` (ubuntu1804-jdk11/Dockerfile)][ubuntu1804-jdk11-dockerfile]
- [`ubuntu1804` (ubuntu1804/Dockerfile)](ubuntu1804-dockerfile)
- [`ubuntu1604` (ubuntu1604/Dockerfile)][ubuntu1604-dockerfile]
- [`ubuntu1404` (ubuntu1404/Dockerfile)][ubuntu1404-dockerfile]

## Packges installed in conatiner

List of available packages inside the container:

- dpkg-dev
- devscripts
- debhelper
- genisoimage
- lsb-release
- build-essential
- git
- java 1.8
- maven 3.5.2
- tomcat
- python
- locate
- which

## Build DEB packages

Building DEB packages with the Docker container is rather easy, a few steps are
required:

### Pull Docker images

Let's assume we want to build packages for Ubuntu 20.04 (Focal). We pull that
image first:

```bashe
docker pull scclouds/cloudstack-deb-builder:ubuntu1604
```

You can replace `ubuntu2004` tag by `ubuntu1804`, `ubuntu1404` or `latest` if
you want.

### Build local repository

You can clone the CloudStack source code from repository locally on your machine
and build packages against that.

#### Clone Apache CloudStack source code

The first step required is to clone the CloudStack source code somewhere on the
filesystem, in `/tmp` for example:

```bash
git clone https://github.com/apache/cloudstack.git /tmp/cloudstack
```

Now that you have done so we can continue.

#### Build packages of local repository

Now that we have cloned the CloudStack source code locally, we can build packages
by mapping `/tmp` into `/mnt/build` in the container. (Note that the container
always expects the `cloudstack` code exists in `/mnt/build` path.)

```bash
docker run \
    -v /tmp:/mnt/build \
    scclouds/cloudstack-deb-builder:ubuntu1604 [ARGS...]
```

Or if your local cloudstack folder has other name, you need to map it to
`/mnt/build/cloudstack`.

```bash
docker run \
    -v /tmp/cloudstack-custom-name:/mnt/build/cloudstack \
    scclouds/cloudstack-deb-builder:ubuntu1604 [ARGS...]
```

After the build has finished the `.deb` packages are available in
`/tmp/cloudstack/dist/debbuild/DEBS` on the host system.

### Build remote repository

Also you can build DEB packages of any remote repository without the need to
manually clone it first. You only need to specify git remote and git ref you
intend to build from.

#### Build packages of remote repository

Now let's assume we want to build packages of `HEAD` of `master` branch from
[https://github.com/apache/cloudstack] repository, we build packages by mapping
`/tmp` into `/mnt/build` in the container. The container will clone the repository
(defined by `--git-remote` flag) and check out the REF (defined by `--git-ref` flag)
in `/mnt/build/cloudstack` inside the container and can be accessed from
`/tmp/cloudstack` from the host machine.

```bash
docker run \
    -v /tmp:/mnt/build \
    scclouds/cloudstack-deb-builder:ubuntu1604 \
        --git-remote https://github.com/apache/cloudstack.git \
        --git-ref master \
        [ARGS...]
```

Note that any valid git Refspec is acceptable, such as:

- `refs/heads/<BRANCH>` to build specified Branch
- `<BRANCH>` short version of build specified Branch
- `refs/pull/<NUMBER>/head` to build specified GitHub Pull Request
- `refs/merge-requests/<NUMBER>/head` to build specified GitLab Merge Request
- `refs/tags/<NAME>` to build specified Tag

After the build has finished the `.deb` packages are available in
`/tmp/cloudstack/dist/debbuilds/DEBS` on the host system.

## Building tips

Check the following tips when using the builder:

### Maven cache

You can provide Maven cache folder (`~/.m2`) as a volume to the container to make
it run faster.

```bash
docker run \
    -v /tmp:/mnt/build \
    -v ~/.m2:/root/.m2 \
    scclouds/cloudstack-deb-builder:ubuntu1604 [ARGS...]
```

### Adjust host owner permission

Builder container in some cases (e.g. using `--use-timestamp` flag) may change
the file and directory owner shared from host to container (through volume) and
it will create `dist` directory which holds the final artifacts. You can provide
`USER_ID` (mandatory) and/or `USER_GID` (optional) from host to adjust the owner
from whitin the container.

This is specially useful if you want to use this image in Jenkins job and want
to clean up the workspace afterward. By adjusting the owner, you won't need to
give your Jenkins' user `sudo` privilege to clean up.

```bash
docker run \
    -v /tmp:/mnt/build \
    -e "USER_ID=$(id -u)" \
    -e "USER_GID=$(id -g)" \
    scclouds/cloudstack-deb-builder:ubuntu1604 [ARGS...]
```

## Builder help

To see all the available options you can pass to `docker run ...` command:

```bash
docker run \
    -v /tmp:/mnt/build \
    scclouds/cloudstack-deb-builder:ubuntu1604 --help
```

## License

Licensed under [Apache License version 2.0]. Please see the [LICENSE] file
included in the root directory of the source tree for extended license details.

[Apache License version 2.0]: http://www.apache.org/licenses/LICENSE-2.0
[LICENSE]: https://github.com/scclouds/cloudstack-deb-builder/blob/master/LICENSE
[ubuntu2004-dockerfile]: https://github.com/scclouds/cloudstack-deb-builder/blob/master/ubuntu2004/Dockerfile
[ubuntu1804-jdk11-dockerfile]: https://github.com/scclouds/cloudstack-deb-builder/blob/master/ubuntu1804-jdk11/Dockerfile
[ubuntu1804-jdk8-dockerfile]: https://github.com/scclouds/cloudstack-deb-builder/blob/master/ubuntu1804/Dockerfile
[ubuntu1604-dockerfile]: https://github.com/scclouds/cloudstack-deb-builder/blob/master/ubuntu1604/Dockerfile
[ubuntu1404-dockerfile]: https://github.com/scclouds/cloudstack-deb-builder/blob/master/ubuntu1404/Dockerfile
[https://github.com/apache/cloudstack]: https://github.com/apache/cloudstack
