# publish docker image to multiple registries

Based on [Publishing Docker images](https://docs.github.com/actions/publishing-packages/publishing-docker-images)

## Set up Automated Builds using GitHub and Docker Hub

Continuous Integration is a software development practice that consists of constantly merging working copies into a common main development branch (up to several times a day) and performing frequent automated project builds to quickly identify potential defects and solve integration problems.
Docker Hub can automatically build images from source code in an external repository and automatically push the built image to your Docker repositories.
When you set up automated builds (also called autobuilds), you create a list of branches and tags that you want to build into Docker images. When you push code to a source code branch (for example in GitHub) for one of those listed image tags, the push uses a webhook to trigger a new build, which produces a Docker image. The built image is then pushed to the Docker Hub registry.

## Hosting some simple static content

A simple Dockerfile can be used to generate a new image that includes the necessary content

```text
FROM nginx:1.19-alpine
ADD index.html /usr/share/nginx/html
```

Place index.html in the same directory as your directory of content

## Publishing images to Docker Hub and GitHub Packages

See [publish-docker-image.yml](.github/workflows/publish-docker-image.yml) on .github

In a single workflow, you can publish your Docker image to multiple registries by using the login-action and build-push-action actions for each registry.

```text
# This workflow uses actions that are not certified by GitHub.
# Estas las proporcionan entidades terceras y las gobiernan
# condiciones de servicio, políticas de privacidad y documentación de soporte
# documentación.

# https://docs.github.com/es/actions/publishing-packages/publishing-docker-images

name: Publish Docker image

on:
  release:
    types: [published]

jobs:
  push_to_registries:
    name: Push Docker image to multiple registries
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - name: Check out the repo
        uses: actions/checkout@v2

      - name: Log in to Docker Hub
        uses: docker/login-action@f054a8b539a109f9f41c372932f1ae047eff08c9
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Log in to the Container registry
        uses: docker/login-action@f054a8b539a109f9f41c372932f1ae047eff08c9
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@98669ae865ea3cffbcbaa878cf57c20bbf1c6c38
        with:
          images: |
            ${{ github.repository }}
            ghcr.io/${{ github.repository }}

      - name: Build and push Docker images
        uses: docker/build-push-action@ad44023a93711e3deb337508980b4b5e9bcdc5dc
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

```

The above workflow checks out the GitHub repository, uses the login-action twice to log in to both registries and generates tags and labels with the metadata-action action. Then the build-push-action action builds and pushes the Docker image to Docker Hub and the Container registry.

## Testing

```text
docker run --name nginx -d -p 8080:80 afreisinger/nginx-alpine 
```
