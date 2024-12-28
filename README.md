# GitLab CI/CD with a Python App

## Introduction and Overview

Welcome to this repository! The goal here is to learn the basics of GitLab CI/CD while working with a Python Flask application. You will learn how to:

- Run tests
- Build a Docker image
- Push the image to a private repository
- Deploy the application to a server

This guide explains each step in detail, so let's get started.

## What is GitLab CI/CD?

GitLab CI/CD is an integrated part of GitLab, a DevOps platform that provides a continuous integration (CI) and continuous delivery (CD) pipeline. It allows developers to:

- Automate testing of code changes.
- Build and deploy applications quickly and efficiently.
- Monitor deployments for any issues.

## What is CI/CD in Simple Words?

CI/CD stands for Continuous Integration and Continuous Delivery/Deployment. It is a practice of automatically building, testing, and deploying applications whenever changes are made to the code. This ensures faster delivery and better quality in software development.

In simple words:

- CI: Continuously integrate code changes into a shared repository.
- CD: Continuously release these changes to the end environment.

## GitLab in Comparison to Other CI/CD Platforms

While Jenkins is still one of the most widely used CI/CD platforms in the industry, GitLab CI/CD is gaining popularity due to its simplicity, integration, and out-of-the-box features. GitLab CI/CD does not require setting up separate servers for pipelines, making it an excellent choice for teams looking for a streamlined setup.

## GitLab Architecture - How GitLab Works

With GitLab CI/CD, you don't need to set up or configure servers manually. Instead, GitLab uses Runners to execute pipeline jobs. A runner is an application that processes builds and sends the results back to GitLab. These runners can be shared or specific to your project.

## Overview of the Demo App (Run Locally)

This repository contains a Python Flask application. To run the application locally:

1. Install the dependencies listed in `requirements.txt`.

2. Use the Makefile to execute commands like:

   - `make test`: Runs the test suite.
   - `make run`: Starts the application.

## Pipeline Configuration File (.gitlab-ci.yml)

### What is .gitlab-ci.yml?

GitLab CI/CD pipelines are defined using a `.gitlab-ci.yml` file. This file contains all the instructions for running CI/CD jobs, and it must be placed in the root directory of your project.

### Structure of `.gitlab-ci.yml`

The pipeline in this repository has three stages:

1. Test
2. Build
3. Deploy

Each stage consists of jobs, and these jobs are executed according to the order defined in the pipeline.

## Run Tests

The first job in the pipeline is to run tests. Here's how it's defined:

```yaml
run_tests:
  stage: test
  image: python:3.9-slim-buster
  before_script:
    - apt-get update && apt-get install -y make
  script:
    - make test
```

This job:

- Uses the Python 3.9 slim image.
- Installs make to run the tests.
- Executes the make test command to validate the application.

## Build and Push Docker Image

The second stage builds the Docker image and pushes it to a private repository.

**Job Configuration:**

```yaml
build_image:
  stage: build
  image: docker:20.10.16
  services:
    - docker:20.10.16-dind
  variables:
    DOCKER_TLS_CERTDIR: '/certs'
  before_script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login -u "$CI_REGISTRY_USER" --password-stdin
  script:
    - docker build -f build/Dockerfile . -t $IMAGE_NAME:$IMAGE_TAG
    - docker push $IMAGE_NAME:$IMAGE_TAG
```

**Key Points:**

1. **Docker-in-Docker**: This setup allows running Docker commands inside a Docker container.

2. **Variables for Login Credentials**: Credentials are stored in GitLab CI/CD variables (CI_REGISTRY_USER and CI_REGISTRY_PASSWORD).

3. **Docker Build and Push**:

   - Builds the Docker image using the Dockerfile.
   - Pushes the image to a private Docker repository.

## Define Stages

GitLab CI/CD stages ensure that jobs are executed in a specific order. In this pipeline:

- The test stage runs first.
- The build stage runs after successful tests.
- The deploy stage runs last.

**Configuration Example:**

```yaml
stages:
  - test
  - build
  - deploy
```

## Prepare Deployment Server

The deployment process involves deploying the Docker container to an Ubuntu server.

**Connect GitLab to the Server**
To connect to the server, you need an SSH private key. This key is stored as a secret variable in GitLab CI/CD settings (SSH_KEY).

**Deployment Job Configuration:**

```yaml
deploy:
  stage: deploy
  before_script:
    - chmod 400 "$SSH_KEY"
  script:
    - |
      ssh -o StrictHostKeyChecking=no -i "$SSH_KEY" ubuntu@13.229.124.173 "
      docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" &&
      CONTAINERS=$(docker ps -aq) &&
      if [ -n "$CONTAINERS" ]; then
        docker stop $CONTAINERS &&
        docker rm $CONTAINERS;
      fi &&
      docker run -d -p 5000:5000 $IMAGE_NAME:$IMAGE_TAG
      "
```

**Steps:**

1. **SSH Login**: Connect to the server using the private key.

2. **Stop and Remove Old Containers**: Ensures a clean slate before deploying the new container.

3. **Run Docker Container**: Deploys the new application version.

## Validate Application Runs Successfully

Once deployed, validate that the application is running by accessing it at `http://<server-ip>:5000`.

## Execute Pipeline

After committing changes, GitLab automatically triggers the pipeline. You can monitor pipeline execution in the CI/**CD > Pipelines** section of your GitLab project.
