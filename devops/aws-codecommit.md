# CodeCommit Repository

Project source code and other artifacts are stored in a [git](https://git-scm.com) repository.

In the AWS console, choose the `CodeCommit` service.

This will show a list of the current repositories.

_There is one repository per project, this is somewhat different from the situation with Subversion where there is often one giant repository with many projects within it._

In the following sections an example project named `"proof-of-concept"` is used.

If the repository is already created, jump straight to [Cloning the repository](#cloning-the-repository).

Reference documentation is [here](https://docs.aws.amazon.com/codecommit/latest/userguide/welcome.html).

## Creating a repository

Use the `Create repository` button.

The repository name should be the project name in some form.

_Try and keep to a consistent repository naming convention._

Description and tags are optional and may be left unused.

The repository page of the newly created project is shown, and contains instructions for a client to clone the repository using SSH (preferred) or HTTPS.

If the repository had a README file in its root directory, the contents of that file would also be shown on this page.

## Initialising the repository

It is **strongly** advised to add a README file to the repository as soon as possible, even if the file just has placeholder content, so that developers will not be cloning an empty repository.

In can cause difficulties for developers when cloning an empty repository, i.e. a repository without a "root" commit. Those issues are not so difficult to resolve, but they are easily avoidable.

_Markdown is very commonly used for the README file as it provides lightweight formatting to regular text files with uncluttered mark-up. See [here](https://commonmark.org) for Markdown reference documentation._

Be careful adding files directly via the AWS console as the author details must be manually entered (ordinarily these details come from git configuration on the client).

To add a basic project README file, from the project repository page choose `Add file` then `Create file` from the popup menu.

Example Markdown content for the new file:

```
# proof-of-concept

An example project.
```

For the file name, use `README.md`.

Add the appropriate author name and email address, take care to use the same details as would ordinarily have been provided by the git client configuration.

Finally, add a commit message (or leave blank to use an auto-generated message) and confirm via the `Commit changes` button.

_Artifacts in the repository are not normally edited directly in the repository, this is a special case to bootstrap the repository so it is not empty when a developer clones it._

Now the repository page will show the contents of this newly added README file. It is a good place to add important project information for developers.

It may also be useful to create a standard `.gitignore` file to prevent IDE artifacts, logs, build output directories and so on from being committed to the repository.

### Branch-level permissions

Depending on the git workflow being adopted by the project team, it may be desirable to set branch-level permissions.

For example, in the well-known [git-flow](https://nvie.com/posts/a-successful-git-branching-model) workflow, regular developers are not generally permitted to commit their changes directly to `master`. Instead, feature branches are used and when a feature is complete the developer's work is reviewed by one or more "approvers" and only then are the developer's changes merged to master (perhaps as part of a release).

See [here](https://aws.amazon.com/blogs/devops/refining-access-to-branches-in-aws-codecommit) for more information.

## Cloning the repository

_It is assumed that the developer has already created and properly configured an SSH key-pair to access this repository._

### Clone URL

The newly created repository is now ready to be cloned by developers.

The repository URL is available on the repository page, a `Clone URL` menu is provided from which the SSH or HTTPS clone URL can be retrieved.

Choosing one or other clone URL option copies the URL to the clipboard ready to be used by the client.

This menu also has an option to show the repository connection steps again if needed.

### Git clone

For general `git` usage the command-line is recommended, although IDE integrations are an option,

In a terminal, change directory to the location that will contain the cloned project.

For example `workspaces` will be used:

```
mark@serenity:~$ cd ~/workspaces
mark@serenity:~/workspaces$
```

To clone the project repository, copy the clone URL from the repository page (as described in the preceding section) and paste it into a clone command:

```
git clone ssh://git-codecommit.eu-west-2.amazonaws.com/v1/repos/proof-of-concept
```

An example terminal session:

```
mark@serenity:~/workspaces$ git clone ssh://git-codecommit.eu-west-2.amazonaws.com/v1/repos/proof-of-concept
Cloning into 'proof-of-concept'...
The authenticity of host 'git-codecommit.eu-west-2.amazonaws.com (52.94.52.153)' can't be established.
RSA key fingerprint is SHA256:zBsKeiw3krlWS34/eQEnCNDsaERZFewWoLsMEXAMPLE.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'git-codecommit.eu-west-2.amazonaws.com,52.94.52.153' (RSA) to the list of known hosts.
remote: Counting objects: 3, done.
Receiving objects: 100% (3/3), 257 bytes | 257.00 KiB/s, done.
mark@serenity:~/workspaces$ cd proof-of-concept
mark@serenity:~/workspaces/proof-of-concept$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

nothing to commit, working tree clean
mark@serenity:~/workspaces/proof-of-concept$
```

In this example, a `proof-of-concept` directory is created underneath `workspaces`, and from there all the usual git commands can be used.
