# Git Basics

This guide provides an introduction to some of the most basic and essential git commands.

It is inteded to give a brief overview of the most common commands, it does not cover all the commands in great detail.

_It is assumed that the necessary git repository credentials have already been configured._

Full reference documentation is [here](https://git-scm.com/docs).

## IDE integration

All modern IDEs will have git repository and git command integration.

Nevertheless, the command-line is often superior in terms of flexibility and efficiency.

_It is strongly recommended to become familiar with git on the command-line._

## Basic concepts

Many developers will be transitioning from Subversion to Git.

Whilst it may be appealing to draw parallels between Subversion and Git concepts, it is not always helpful. It is better to come to git with a clean slate and leave most Subversion habits behind.

Having said that, broadly speaking some concepts do have parallels.

### Getting the code

Cloning a repository copies the repository to a working copy on your local PC.

Cloning is analagous to a Subversion checkout.

## Cloning a repository

Repositories can be accessed either using `SSH` or `HTTPS`.

Using ssh is recommended, mainly because a username/password request will not be prompted on every change to the repository.

Generally:

```
git clone repository-url
```

The `repository-url` above varies depending on the type of remote repository.

### Github and GitLab

For [GitHub](https://github.com) and [GitLab](https://gitlab.com), using ssh:

```
git clone git@github.com:YOUR-USERNAME/YOUR-REPOSITORY.git
```

Or HTTPS:

```
git clone https://github.com/YOUR-USERNAME/YOUR-REPOSITORY.git
```

### CodeCommit

For AWS [CodeCommit](https://aws.amazon.com/codecommit), using ssh:

```
git clone ssh://git-codecommit.YOUR-AWS-REGION.amazonaws.com/v1/repos/YOUR-REPOSITORY
```

Or HTTPS:

```
https://git-codecommit.YOUR-AWS_REGION.amazonaws.com/v1/repos/YOUR_REPOSITORY
```

## Working copy status

The `git status` command reports on the status of the current repository, showing the current branch and all of the resources that have been made to the working copy.

```
git status
```

This command also gives useful prompts with relevant command syntax depending on the state of the working copy.

## Differences

To see all of the changes in the working copy in standard `diff` format, use `git diff`:

```
git diff
```

A file specification can optionally be supplied to limit the scope of the diff:

```
git diff src/test
git diff pom.xml
git diff "*.md"
```

## Staging changes

Before changes can be commited, they must first be "staged".

Staging is used to add files to the changeset, remove staged files from the changeset, and just generally prepare for a subsequent commit.

File are staged with `git add` and a file specification, some examples:

Add the current directory, and all sub-directories recursively, to the staging area:
```
git add .
```

Add a specific directory and all of its contents:

```
git add src/main/java
```

Add a single file:

```
git add pom.xml
```

A file will not be staged if it has not changed, so wildcards can be used for file specificatons but only the changed files will be staged.

Artifacts can be un-staged with `git reset HEAD` and a file specification, for example:

```
get reset HEAD pom.xml
```

_If a staged file is subsequently changed, it would need be to staged **again** to have the new changes included in the changeset._

## Commiting changes

To commit the staged files, use `git commit`.

```
git commit
```

All staged changes will be committed to the current branch, and removed from the staging area.

An important distinction when transitioning from Subversion to Git is that when changes are commited to a working copy they are sent to the original repository (termed a "remote") until changes are expressly pushed.

You are free to create and delete whatever branches you want locally, commit whatever changes you want, undo it all, and none of it will touch the original repository unless and until those changes are pushed.

A push is effectively publishing your changes.

```
git push
```

## Updating the working copy

To update the local working copy with the latest changes from the remote repository, use `git pull`:

```
git pull
```

There is also `git fetch`, which can be used to fetch all of the changes from the remote repository, but without merging them to the local working copy.

A `pull` is effectively a `fetch` then a `merge`.

## Resolving merge conflicts

Pulling from the remote repository may introduced merge conflicts.

Conflicts are resolved using `git mergetool`.

## Branching and merging

Branching is strongly encouraged when using git. Branches are very easy and very quick to create, switch between, merge and delete.

### Creating a branch

To create a branch:

```
git checkout -b my-cool-feature-branch
```

The branch will be created and switched to near instantaneously.

You can easily diff between branches, e.g. to show the difference between the current branch and `master`:

```
git diff master
```

### Listing branches

To list the branches:

```
git branch
```

The current branch will be indicated with an asterisk '*'.

### Merging branches

To merge a feature branch back to master:

```
git checkout master
git merge my-cool-feature-branch
```

See also [git client configuration](git-client-config.md) for some comments on the different types of merge.

### Deleting branches

To delete a branch, if the branch has been merged back:

```
git branch -d my-cool-feature-branch
```

The previous command will fail if the branch has not been merged.

To delete a branch that has not been merged:

```
git branch -D my-cool-feature-branch
```

## Updating a feature branch

A feature branch can be updated by rebasing it against some other branch:

```
git rebase master
```

Or by merging changes from some other branch:

```
git merge master
```

## Stashing changes

Git provides a temporary stash where local changes can be kept, usually while some other change is being made.

An example of this could be stashing some unfinished changes while a merge is performed, then after the merge completes restoring those other changed files back from the stash.

To stash current changes:

```
git stash
```

To list the stash (there may be multiple sets of changes stashed):

```
git stash list
```

To apply the most recently stashed changes:

```
git stash apply
```

## Summary

This guide has introduced some basic, and most frequently used, git commands.

There are many more commands and options available.

For example, it is possible to push local feature branches to the remote repository (e.g. to prepare a pull request for review, or to share a branch with teammates).

It is possible to add multiple remote repositories and push changes to one or more of them.

Patches can be created and applied.

The working copy commit history can be changed by using `rebase`.

And much, much more...
