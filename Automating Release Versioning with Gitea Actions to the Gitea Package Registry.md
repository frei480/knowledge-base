---
tags:
  - cicd
link: https://about.gitea.com/resources/tutorials/automating-release-versioning-with-gitea-actions-to-the-gitea-package-registry
---
# Automating Release Versioning with Gitea Actions to the Gitea Package Registry
## Introduction

Automated release versioning plays a crucial role in contemporary CI/CD pipelines. There are situations when it's preferable to publish build artifacts to a private, self-hosted package registry instead of a public one. Yet, managing codebase, CI/CD tools, and a private registry can be quite a challenge. Thankfully, Gitea offers a solution by combining all these functions into one platform. By setting up a Gitea instance, you can manage the entire workflow. This tutorial demonstrates how to automatically build a Docker image and upload it to the Gitea Docker registry.

## TL;DR

Paste the workflow file under your `.gitea/workflows` folder, replace the local IP with your own, add the relevant secret for your workflow, and you're all set.

```yaml
name: release-tag

on:
  push

jobs:
  release-image:
    runs-on: ubuntu-latest
    container:
      image: catthehacker/ubuntu:act-latest
    env:
      DOCKER_ORG: teacup
      DOCKER_LATEST: nightly
      RUNNER_TOOL_CACHE: /toolcache
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v2

      - name: Set up Docker BuildX
        uses: docker/setup-buildx-action@v2
        with: # replace it with your local IP
          config-inline: |
            [registry."192.168.8.30:3000"] 
              http = true
              insecure = true            

      - name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          registry: 192.168.8.30:3000 # replace it with your local IP
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
          
      - name: Get Meta
        id: meta
        run: |
          echo REPO_NAME=$(echo ${GITHUB_REPOSITORY} | awk -F"/" '{print $2}') >> $GITHUB_OUTPUT
          echo REPO_VERSION=$(git describe --tags --always | sed 's/^v//') >> $GITHUB_OUTPUT                

      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          file: ./Dockerfile
          platforms: |
            linux/amd64
            linux/arm64                        
          push: true
          tags: | # replace it with your local IP and tags
            192.168.8.30:3000/${{ env.DOCKER_ORG }}/${{ steps.meta.outputs.REPO_NAME }}:${{ steps.meta.outputs.REPO_VERSION }}
            192.168.8.30:3000/${{ env.DOCKER_ORG }}/${{ steps.meta.outputs.REPO_NAME }}:${{ env.DOCKER_LATEST }}
```

![docker package](https://about.gitea.com/img/tutorials/automating-release-versioning-with-gitea-actions-to-the-gitea-package-registry/docker_package.png)

## Explanation

Let's dissect the different components of this workflow file. It consists of three parts: setting up the Docker environment, logging in to the Docker registry, and building and uploading the image.

### Set up the Docker Environment

```yaml
steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v2

      - name: Set up Docker BuildX
        uses: docker/setup-buildx-action@v2
        with: # replace it with your local IP
          config-inline: |
            [registry."192.168.8.30:3000"] 
              http = true
              insecure = true
```

These three steps are designed to set up the Docker environment.

As this tutorial does not cover HTTPS setup in Gitea, we are using HTTP for our local Gitea instance. Since the Docker action defaults to HTTPS, we need to adjust the configuration accordingly.

**Note:** If your Gitea instance is open to the public network, please use HTTPS. Follow [the official Gitea guide](https://docs.gitea.com/administration/https-setup) to set up HTTPS for your Gitea instance and [this guide](https://github.com/docker/login-action/issues/295) for setting up HTTPS certification for your Docker.

### Login to the Docker Registry

```
      - name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          registry: http://192.168.8.30:3000
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Get Meta
        id: meta
        run: |
          echo REPO_NAME=$(echo ${GITHUB_REPOSITORY} | awk -F"/" '{print $2}') >> $GITHUB_OUTPUT
          echo REPO_VERSION=$(git describe --tags --always | sed 's/^v//') >> $GITHUB_OUTPUT
```

Ensure to set up the secrets beforehand.

![docker secrets](https://about.gitea.com/img/tutorials/automating-release-versioning-with-gitea-actions-to-the-gitea-package-registry/docker_secrets.png)

The `Get Meta` step is used for obtaining metadata for the Docker tag. You can customize the Docker tag according to your preference.

### Build and Upload the Image

```
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          file: ./Dockerfile
          platforms: |
            linux/amd64
            linux/arm64                        
          push: true
          tags: | # replace it with your local IP and tags
            192.168.8.30:3000/${{ env.DOCKER_ORG }}/${{ steps.meta.outputs.REPO_NAME }}:${{ steps.meta.outputs.REPO_VERSION }}
            192.168.8.30:3000/${{ env.DOCKER_ORG }}/${{ steps.meta.outputs.REPO_NAME }}:${{ env.DOCKER_LATEST }}
```

The tag format is `{package registry address}/{owner}/{image}:{tag}`, which should be replaced with your own tag.

In the example above, we use the `meta` data obtained from the `Login to the Docker registry` step.