# Amazon Elastic Container Registry (ECR)

A container registry, `ECR`, is used to store Docker images created by the build process.

In the AWS console, choose the `ECR` service.

Reference documentation is [here](https://aws.amazon.com/ecr).

## Creating a repository

If this is the first repository, use the `Get started` button.

The repository name may be related to a specific project, or may be more general - it depends if there is a need to keep various different images separately or not.

For this repository, `"proof-of-concept-repo"` will be used.

Other values can be left at defaults.

Use the 'Create repository` button.

When the repository is created, the page will refresh and show the list of repositories.

In future, the `Create repository` button will be used rather than the `Get started` button mentioned previously.

## Additional steps

### Docker privileges

The IAM service role used for `CodeBuild` **must** be changed to enable pushing Docker images to the container registry.

It is likely this service role was created automatically when setting up the code repository (it is possible to set it up manually instead).

In the AWS console, choose the `IAM` service, then `Roles`.

Select the role used for `CodeCommit`, in this example `codebuild-proof-of-concept-snapshot-service-role` is used.

There should be a pre-existing policy under the `Permissions` tab, in this example it is `CodeBuildBasePolicy-proof-of-concept-snapshot-eu-west-2`.

Select the policy, then use the `Edit policy` button and switch to the `JSON` tab.

The new `Action` block shown below **must** be inserted.

```
{
    "Version": "2012-10-17",
    "Statement": [
        ## start of insert ##
        {
           "Action": [
                "ecr:BatchCheckLayerAvailability",
                "ecr:CompleteLayerUpload",
                "ecr:GetAuthorizationToken",
                "ecr:InitiateLayerUpload",
                "ecr:PutImage",
                "ecr:UploadLayerPart"
            ],
            "Resource": "*",
            "Effect": "Allow"
        },
        ## end of insert ##
        {
            "Effect": "Allow",
            ...
        }
    ]
```

Obviously "## start of insert ##" and "## end of insert ##" text should **not** be included, it is used here only to indicate the extent of the necessary changes.

When the edits have been made, use `Review policy` then `Save changes` to commit the changes.

If this is not done, when building the project the following error message will be encountered and the build will fail:

```
[Container] 2020/02/29 09:30:18 Running command $(aws ecr get-login --no-include-email --region $AWS_DEFAULT_REGION)

An error occurred (AccessDeniedException) when calling the GetAuthorizationToken operation: User: arn:aws:sts::847463738381:assumed-role/codebuild-proof-of-concept-snapshot-service-role/AWSCodeBuild-4002c38e-ef97-2d1e-34aa-fc23a6cb4d6f is not authorized to perform: ecr:GetAuthorizationToken on resource: *
```

There is an unfortunate consequence of manually editing this policy file - if further changes are made to the `CodeCommit` build project `Environment` it will complain when saving.

The easiest solution is to uncheck the `Allow CodeBuild to modify this service role so it can be used with this build project` checkbox before saving the changes.

Further information is [here](https://docs.aws.amazon.com/codebuild/latest/userguide/troubleshooting.html#enhanced-zero-click-role-creation).

This is required for `CodeBuild` only when building Docker images.

### User account

AWS recommend that an AWS root account is **not** used for the code repository to access the container registry, further information is in the sample documentation.

Further information is [here](https://docs.aws.amazon.com/codebuild/latest/userguide/sample-docker.html).

## Pulling images

Use the AWS command-line interface.

_It is assumed the AWS cli is properly installed and configured._

In the examples that follow, these values are illustrative only and need to be replaced with whatever is appropriate:

 * `123412341234` is the AWS account ID
 * `eu-west-2` is the AWS region
 * `proof-of-concept-repo` is the repository name
 * `fcd3f42` is the image tag

It may first be necessary to authenticate the AWS command-line session to the container registry, for example:

```
aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin 123412341234.dkr.ecr.eu-west-2.amazonaws.com
```

To list the repositories:

```
aws ecr describe-repositories
```

This will return a JSON response containing the repository identifiers, find the relevant repository.

To list the images in the repository, in this example `proof-of-concept-repo` is used:

```
aws ecr describe-images --repository-name proof-of-concept-repo
```

From the list of images, find the relevant tag and then use Docker to pull the image from ECR to the local repository.

To pull the image:

```
docker pull 123412341234.dkr.ecr.eu-west-2.amazonaws.com/proof-of-concept-repo:fcd3f42
```

To check the image is in the local repository:

```
docker images
```

Finally, to run the image locally:

```
docker run -p80:8080 123412341234.dkr.ecr.eu-west-2.amazonaws.com/proof-of-concept-repo:fcd3f42
```

In this example run command `-p80:8080` is used to map the exposed port `8080` from the image and publish it to the local port `80`. Again, these are just illustrative values, use whatever is appropriate.
