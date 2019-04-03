# git-push-fork-to-upstream-branch

This is a tool primarily intended for being able to trigger integration tests
to run on code submitted from forks.

Sometimes, your automated testing needs to call out to external services
or access external data, requiring credentials. Most Continuous Integration (CI)
providers have facilities for uploading such credentials as secrets that
are only exposed to trusted builds, that is builds triggered from pushes
to the main repository where only project committers have write access.

If you want to allow pull requests (PRs) from outside contributors
who don't have commit access to your main repository on GitHub, you
either have to be content for only a subset of tests to run that don't
require credentials or you need to check out the forked PR locally and
run your tests manually.

This tool presents another option where a committer checks over a PR
to ensure none of the changes will expose secrets in CI logs, then runs
a one-line command locally that pushes the exact commits from the fork
to a branch of the main repository. This is an indirect way of the committer
expressing their trust of the forked code and it has the effect of
triggering trusted builds on the CI provider which GitHub then associates
with the forked PR.

This workflow has been tested to work with both CircleCI and TravisCI,
but GitHub is the only supported Git host.

## Installation

The tool is a simple bash script that makes various calls to `git` for
configuring remotes, fetching commits, and republishing them to the
upstream repo. The only prerequisites are having `bash` and `git` installed
on your system and installation involves simply copying the script into
a location on your `$PATH`:

```bash
# Feel free to choose a different destination directory on your $PATH;
# /usr/local/bin used here is on the path by default on Linux and Mac systems.
GPF_INSTALL_LOCATION=/usr/local/bin/git-push-fork-to-upstream-branch

GPF_URL=https://raw.githubusercontent.com/jklukas/git-push-fork-to-upstream-branch/master/git-push-fork-to-upstream-branch

# These last commands may require sudo depending on your user's privileges.
curl -sL $GPF_URL > $GPF_INSTALL_LOCATION
chmod 755 $GPF_INSTALL_LOCATION
```

## Configuration

The tool respects a few configuration options set via environment variables.

To push to an upstream branch other than `trigger-integration`, set:

    export GPF_UPSTREAM_BRANCH=my-custom-branch-name

To push code to the upstream branch via ssh rather than the default of https, set:

    export GPF_USE_SSH=true

## Usage

First, we assume that you have the target Git project cloned to your local
machine and that you have a remote configured that points to the upstream
project (the one that contributors would be issuing PRs to).

When a contributor submits a PR from a fork, the main page for the PR on GitHub
will include text similar to the following at the top:

> jklukas wants to merge 1 commit into `myorg:master` from `jklukas:new-feature`

First, do an initial review of the PR to check for any code that may spill
secrets to logs of your CI system (whether intentional or accidental);
remember that this process relies on you the reviewer to act as gatekeeper
for what code is allowed to run in CI. Once you're comfortable that the
code is safe to send to CI, navigate to the root of your local checkout 
of the project and invoke:

```bash
git-push-fork-to-upstream-branch upstream jklukas:new-feature
```

That invocation assumes that your remote is named `upstream` and it copies in
the `username:branch` from the PR; note that GitHub includes a handy button
right next to that text for copying to the clipboard and pasting to this command.
The code has now been pushed to the `trigger-integration`and because this is
exactly the same commit (with the same hash) as on the forked repository, most
CI systems will trigger a build that GitHub then associates with the forked PR.
You should see status change in the PR to indicate that a new build is running.
