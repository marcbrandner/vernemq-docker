Docker Images for VerneMQ
===

This repository houses Dockerfiles for VerneMQ including the build process of VerneMQ binaries. As it is not using the VerneMQ binaries built by OctavoLabs, the [VerneMQ End User License Agreement](https://vernemq.com/blog/2019/11/26/vernemq-end-user-license-agreement.html) does not apply. Hence you are free to use it for any use case without the obligation of a subscription. It also allows you to build the Docker image with newer versions of Erlang or the Alpine base image if you need to update them for security reasons. On the other hand, this is just a "standard" build without any optimizations applied during the build process by the experts from Octavo Labs. It's your choice.

<!-- TOC -->

- [1. Quickstart](#1-quickstart)
- [2. Easy Build](#2-easy-build)
- [3. Build a Specific Version](#3-build-a-specific-version)
- [4. Use Image](#4-use-image)
- [5. Troubleshooting](#5-troubleshooting)
    - [5.1. Errors Fetching Plugins](#51-errors-fetching-plugins)
    - [5.2. Build Failures](#52-build-failures)
- [6. Roadmap](#6-roadmap)
- [7. Maintenance](#7-maintenance)
    - [7.1. Push Images to GitHub Container Registry](#71-push-images-to-github-container-registry)

<!-- /TOC -->

# 1. Quickstart

Use an existing image from `ghcr.io`:
```
docker run --name vmq1 -p 8888:8888 -p 1883:1883 -d ghcr.io/marcbrandner/vernemq:1.12.3-build.1-alpine
```

# 2. Easy Build

Clone the repository, i.e. like this:
```
git clone https://github.com/marcbrandner/vernemq-docker.git -b 1.12.3-build.1
```

(The version number (`1.12.3-build.1`) may be replaced with one of the [available tags](https://github.com/marcbrandner/vernemq-docker/releases).)

Build the image based on the `alpine` base image:
```
docker build -t vernemq:1.12.3-build.1-alpine vernemq-docker/alpine
```

Or build the image based on the `debian-slim` base image:
```
docker build -t vernemq:1.12.3-build.1-debian-slim vernemq-docker/debian-slim
```

# 3. Build a Specific Version

If the VerneMQ version you require is not available as a tag in this repository, you can set a valid combination of component versions using Docker Build Arguments.

Example:
```
git clone https://github.com/marcbrandner/vernemq-docker.git -b main

# alpine
docker build -t vernemq:foobar-alpine \
    --build-arg VMQ_VERSION=1.12.3 \
    --build-arg ERLANG_VERSION=23.3.2.0 \
    --build-arg ALPINE_VERSION=3.13.6 \
    vernemq-docker/alpine

# debian-slim
docker build -t vernemq:foobar-debian-slim \
    --build-arg VMQ_VERSION=1.12.3 \
    --build-arg ERLANG_VERSION=23.3.2.0 \
    --build-arg DEBIAN_SLIM_VERSION=stable-20211011-slim \
    vernemq-docker/debian-slim
```
* `BUILD_VERSION`: Name of the tag/release of this repository. You can pick whatever fits your demands as this version string will only be used to tag the image.
* `VMQ_VERSION`: Valid version of the VerneMQ source code to use for the build from the [VerneMQ repo](https://github.com/vernemq/vernemq).
* `ERLANG_VERSION`: Valid Erlang version used for the build stage compatible with `VMQ_VERSION` (see [Release Notes in VerneMQ repo](https://github.com/vernemq/vernemq/releases).
* `ALPINE_VERSION`: Valid Alpine base image version used for the run stage (see [VerneMQ `alpine` Dockerfile](https://github.com/vernemq/docker-vernemq/blob/master/Dockerfile.alpine) or [DockerHub](https://hub.docker.com/_/alpine)).
* `DEBIAN_SLIM_VERSION`: Valid Debian base image version used for the run stage (see [VerneMQ `debian` Dockerfile](https://github.com/vernemq/docker-vernemq/blob/master/Dockerfile) or [DockerHub](https://hub.docker.com/_/debian)).

# 4. Use Image

If you don't want to build the images yourself, already built images can be found on [Github's container registry at `ghcr.io/marcbrandner/vernemq`](https://github.com/users/marcbrandner/packages/container/package/vernemq).

Images are not regularly built for newlypublished [VerneMQ releases](https://github.com/vernemq/vernemq/releases). If you want a specific version that is not available in this repo's tags, [build it yourself](#build-a-specific-version).

Run a VerneMQ container __using an image from `ghcr.io`__:
```
# alpine
docker run --name vmq1 -p 8888:8888 -p 1883:1883 -d ghcr.io/marcbrandner/vernemq:1.12.3-build.1-alpine

# debian
docker run --name vmq1 -p 8888:8888 -p 1883:1883 -d ghcr.io/marcbrandner/vernemq:1.12.3-build.1-debian-slim
```
Run a VerneMQ container __using your own image__:
```
docker run --name vmq1 -p 8888:8888 -p 1883:1883 -d vernemq:<your-tag>
```
Check the logs:
```
docker logs vmq1 -f
```
Check the VerneMQ instance's status:
```
docker exec vmq1 vmq-admin cluster show
```
Use `curl` to test the status page:
```
curl localhost:8888/status --verbose
```
Enter a container a running VerneMQ instance:
```
docker exec -it vmq1  /bin/sh
```
Run a container __without__ starting a VerneMQ instance and get an interactive shell (i.e for troubleshooting):
```
# alpine
docker run -it --rm vernemq:1.12.3-build.1-alpine /bin/sh

# debian-slim
docker run -it --rm vernemq:1.12.3-build.1-debian-slim /bin/sh
```

# 5. Troubleshooting

## 5.1. Errors Fetching Plugins

__Error:__
```
===> Fetching rebar3_cuttlefish (from {git,"git://github.com/vernemq/rebar3_cuttlefish",
                             {ref,"4cf5e1c8133bb54be7badbd8aa57aa5f79763452"}})
===> Errors loading plugin {rebar3_cuttlefish,
                               {git,
                                   "git://github.com/vernemq/rebar3_cuttlefish",
                                   {ref,
                                       "4cf5e1c8133bb54be7badbd8aa57aa5f79763452"}}}. Run rebar3 with DEBUG=1 set to see errors.
```
__Possible Cause:__ Most likely your network does not allow you to clone from GitHub using `git://` which uses SSH.

__Possible Solution:__ Changing to `https://` in the `rebar.config` does not cover all plugins, which is why so far there is no other way than poking a hole into your firewall for SSH on port 22 or using a another way to access the Internet.

## 5.2. Build Failures

__Error:__ Obscure build failures after all plugins are fetched.

__Possible Cause:__ If a build fails, the Docker build leaves behind some residual layers which are picked up by subsequent builds (with repository `<none>` and tag `<none>`).

__Possible Solution:__ Delete those dangling layers to clean up:
```
for image_id in $(docker images | awk '{print $1 " " $3}' | grep '<none>' | awk '{print $2}'); do
    docker rmi -f $image_id
done
```

# 6. Roadmap

* Reduce image size by removing `nano` from all images
* Automation with GitHub Actions:
    - Automate Docker image build
    - Trigger builds on each new VerneMQ release

# 7. Maintenance

A few things useful for maintenance of this repository:

## 7.1. Push Images to GitHub Container Registry

* Authenticate in browser with GitHub account 
* Generate a new Personal Access Token ([see GitHub documentation](https://docs.github.com/en/packages/working-with-a-github-packages-registry/))
* Login to `ghcr.io`:
```
export CR_PAT=YOUR_TOKEN
echo $CR_PAT | docker login ghcr.io -u USERNAME --password-stdin
```
* Tag and push the images:
```
# alpine
docker tag vernemq:1.12.3-build.1-alpine ghcr.io/marcbrandner/vernemq:1.12.3-build.1-alpine
docker push ghcr.io/marcbrandner/vernemq:1.12.3-build.1-alpine

# debian-slim
docker tag vernemq:1.12.3-build.1-debian-slim ghcr.io/marcbrandner/vernemq:1.12.3-build.1-debian-slim
docker push ghcr.io/marcbrandner/vernemq:1.12.3-build.1-debian-slim
```