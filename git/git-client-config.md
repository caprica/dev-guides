# Git Client Configuration

Some useful client configuration settings when getting started with `git`.

Reference documentation is [here](https://git-scm.com/docs/git-config).

## Local vs global configuration

Git configuration commands are executed by invoking `git config`. In this way the configuration can be applied for the current repository only (local).

To set global values instead, use `git config --global`. Global settings are applied on all repositories.

## Essentials

It is highly recommended to set consistent author name and email settigs for all repositories. The values for these settings should follow some organisational standard.

```
git config --global user.name "Mark Lee"
git config --global user.email "mark.lee@capricasoftware.co.uk"
```

Checking the configuration:

```
git config --global --list
```

An example terminal session:

```
mark@serenity:~$ git config --global --list
user.email=mark.lee@capricasoftware.co.uk
user.name=Mark Lee
mark@serenity:~$
```

These are the values that will be incorporated into and displayed in the repository commit log.

## Editor

It is possible to change the editor used, for example when editing commit messages:

```
git config --global core.editor "nano"
```

Typical options are "vi", "vim", "nano" (in increasing order of ease of use).

It is possible to set any command, even something like VSCode or Notepad++.

## Merges

By default git uses 'fast forward' to perform merges.

Some teams do not like this as it can lose context about what was actually changed. Using the git option for `--no-ff` switches to a different type of merging.

```
git config --global merge.commit no
git config --global merge.ff no
```

These config settings only switch the default behaviour, it is possible to pass `--ff` or `--no-ff` to individual commands to override that default behaviour.

For example, it is sometimes recommended to not use fast-forward merges for branch merges, but to allow it when rebasing in a developer's private feature branch (to keep the branch history clean).

## Other options

This guide has covered some of the more common options, consult the reference documentation for information on the myriad other options.
