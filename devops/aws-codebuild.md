# CodeBuild

The project is built and assembled using a build project on `CodeBuild`.

In the AWS console, choose the `CodeBuild` service.

This will show a list of the current build projects.

In the following sections an example project named `"proof-of-concept"` is used.

If the build project is already created, jump straight to [Building the project](#building-the-project).

## Creating a build project

Use the `Create build project` button.

### Project configuration

The build project name reflect the project name in some form, perhaps incorporating "SNAPSHOT" or "RELEASE" or some other scheme to distinguish different verions of the build.

_Try and keep to a consistent build project naming convention._

Description, tags and the build badge are optional and can be left unused.

The build page, if enabled, would provide a link to an image showing the current state of the build project - this link could be embedded in the project code repository README or some other place like a dashboard where it would be useful to see project build statuses.

_Build badges are **public** links, but do not contain any project-sensitive details._

### Source

For the source, use `AWS CodeCommit` and select the appropriate repository.

For this example build, building straight from master, choose the `master` branch from the dropdown list. Other builds may select some other branch, or even a specific commit hash in the repository.

The `git clone depth` should generally be `1` - this means that only the latest revision of the project artifacts will be cloned from the repository prior to the build.

### Environment

Default values are mostly fine.

This project will use the `Managed image` option, with `Ubuntu` selected for the `Operating system`.

For the build runtime choose:

 * `Runtime(s)` should be `Standard`
 * `Image` should be the one with the highest tagged version, in this case `aws/codebuild/standard:3.0`
 * `Image version` should be `Always use the latest image for this runtime version`
 * `Environment type` should be `Linux`

Since this project will be using Docker, the `Privileged` checkbox **must** also be checked.

Remaining values can be defaults, although further changes are possible to configure the provisioning of the build server.

#### Environment variables

Environment variables are used to pass configuration settings into the build process, for now these will be left blank.

### Buildspec

Mostly defaults will be used here, i.e. the project will use a `buildspec.yml` file included in the project root of the source code repository.

The buildspec contains all of the commands necessasry to complete the build, it is analagous to 'structured shell script' file.

The structure is in the form of specific named build phases, with regular shell script commands associated with each phase. The buildspec can also use normal shell script token substitution to apply any supplied environment variable values.

Reference information is [here](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html).

#### Example buildspec for a Maven project

This example `buildspec.yml` file shows a very basic build using [Maven](https://maven.apache.org):

```
version: 0.2

phases:
  install:
    runtime-versions:
      java: openjdk11
    commands:
      - apt-get update -y
      - apt-get install -y maven
  build:
    commands:
      - echo Build started on `date`
      - echo Building application...
      - mvn install
      - echo Finished building application.
  post_build:
    commands:
      - echo Build completed on `date`

artifacts:
  files:
    - proof-of-concept-0.0.1-SNAPSHOT.jar
```

The buildspec file should be located in the project root directory in the source code repository. It is committed just like any other project file.

This buildspec simply executes a `maven install` command, there is no Docker build in this case - but Docker commands _would_ be added here.

### Docker image

Building a Docker image requires a different `buildspec.yml` file, and a `Dockerfile`.

The buildspec must still include commands to build the application code, but additional commands are used to build the Docker image.

The Docker image needs to be uploaded to a container registry, in this case the Amazon Elastic Container Registry is used.

Reference documentation is [here](https://aws.amazon.com/ecr).

_It as assumed the container registry has already been provisioned._

The buildspec must contain commands to authenticate against the container registry so that the Docker image may be pushed to it.

There are various other complexities in the new buildspec, such as pulling out and parsing the current code repository commit hash and tagging the resulting Docker image appropriately.

#### Environment variables

Environment variables are used in the buildspec file.

From the build project home page, select `Edit` then `Environment`, then `Additional configuration`.

In the `Environment variables` section, add:

|Variable name     |Example              |
|------------------|---------------------|
|AWS_ACCOUNT_ID    |123412341234         |
|AWS_DEFAULT_REGION|eu-west-2            |
|IMAGE_REPO_NAME   |proof-of-concept-repo|

Commit the changes via the `Update environment` button.

#### Example Dockerfile for a Maven project

The `Dockerfile`, also in the project root directory in the source code repository, is used to prepare the image and to copy the application artifacts into the image.

```
FROM openjdk:11-jdk
LABEL maintainer="mark.lee@capricasoftware.co.uk"
ARG GROUP=poc
ARG USER=poc
RUN groupadd ${GROUP} && useradd -g ${GROUP} -s /bin/sh ${USER}
USER ${USER}:${GROUP}
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} proof-of-concept.jar
ENTRYPOINT ["java", "-jar", "proof-of-concept.jar"]
```

This buildspec is illustrative, commands can be added/changed as needed for a particular build.

#### Example buildspec for a Docker project

This example builds the application using Maven, extracts the current commit hash value and uses that to tag the Docker image when it pushes it to the container registry.

```
version: 0.2

phases:
  install:
    runtime-versions:
      java: openjdk11
    commands:
      - apt-get update -y
      - apt-get install -y maven
  pre_build:
    commands:
      - REPO_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:=latest}
      - echo Logging in to Amazon ECR...
      - $(aws ecr get-login --no-include-email --region $AWS_DEFAULT_REGION)
      - echo Logged in to ECR.
  build:
    commands:
      - echo Build started on `date`
      - echo Building application...
      - mvn install
      - echo Finished building application.
      - echo Building Docker image...
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
      - echo Finished building Docker image.
  post_build:
    commands:
      - echo Pushing the Docker image...
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
      - echo Finished pushing the Docker image.
      - echo Build completed on `date`

```

This buildspec is illustrative, commands can be added/changed as needed for a particular build.

### Artifacts

Since this project ultimately is intended to build Docker images, ensure that the selected artifact `Type` is `No artifacts`.

### Logs

Without logs, build success/failures are visible only in the AWS console and only at a high level (e.g. which phase of the build failed, rather than a specific failure for a particular command that failed).

It is therefore sensible to enable logs - for this project the `CloudWatch` logs are enabled. Other log options are left blank.

### Create build project

Finally, use the `Create build project` button to finish creating this project.

The console will refresh and show the build project home page.

From this page it is possible to edit the build project, start builds, inspect previous builds and so on.

This page also provides, if enabled earlier, a `Copy badge URL` button that copies a link to a current build status image that can be included in the project README or some other dashboard/status page.

## Building the project

If the project source code repository already contains a `buildspec.yml` file, the `Start build` button can be used to kick off a new build.

Starting the build performs all of the necessary steps to produce the build artifact, including:

 * provisioning of the build server
 * applying the Docker image with the build runtime to the server
 * executing various pre-build commands from the buildspec (such as updating software, installing packages and build tools)
 * cloning the source code repository
 * executing various build commands from the buildspec to complete the build (such as providing container registry credentials, executing `maven` commands, executing `docker` commands)
 * execiting various post-build commands from the buildspec (such as pusing a Docker image to a container registry)

During the build, the status can be followed in the console either by watching the various build phase indicators, or where logging is enabled by tailing the build log.

If the build was successful, the new Docker image will now be present and visible in the container registry.

## Conclusion

The build project is now ready.

The project described here has basic configuration, other settings can be made such as using a Virtual Private Cloud, configuring the provisioned build server and so on.
