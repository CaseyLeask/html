# HTML Standard Team Instructions

This document contains info used by the team maintaining the standard. Mostly boring infrastructure stuff.

## Handling pull requests

The green button shall not be pushed. Each change needs to result in a single commit on the master branch, with no merge commits.

For optimal merges, the following instructions may be helpful:

### Fetching pull requests from forks

To be able to more easily review pull requests from forks, you can optionally configure your clone such that:

* alongside the fetched branches you already have for all normal named branches local to this repo, you'll also have fetched branches for all existing PRs
* each time you run `git fetch` or `git pull` and it finds a PR branch *NNN* you don’t have in your clone, you `git` will automatically fetch that branch into your clone, giving it a branch name of the form `pr/NNN`
* you can `git checkout` any PR branch you want to build/review, and use `git merge` to pull any updates to it

To do all that, use these steps:

1. Make the following addition to your `html/.git/config` to enable automatic fetch of branches in PRs from forks:

  ```diff
    [remote "origin"]
           url = git@github.com:whatwg/html.git
           fetch = +refs/heads/*:refs/remotes/origin/*
   +        fetch = +refs/pull/*/head:refs/remotes/origin/pr/*
  ```
  (Omit the `+` sign; it’s just `diff` syntax to get the markdown viewer to highlight the line.)

2.  Run `git fetch` or `git pull` to do the initial fetch of all branches for current PRs.

3.  Run `git checkout pr/NNN` to check out a particular PR branch.

4.  If a contributor subsequently pushes changes to the corresponding branch for that PR in their fork (for example, in response to your review comments), then while you're on the checked-out `pr/NNN` branch locally you can pull those changes  by doing the following:

  ```bash
  git fetch
  git merge origin/pr/NNN
  ```

### Merging pull requests from forks

Pull requests from external contributors come from their forks. Here is a Bash function that you can add to your `.bash_profile` or similar that makes it easy to merge such PRs:

```bash
pr () {
  git fetch origin refs/pull/$1/head:refs/remotes/origin/pr/$1 --force
  git checkout -b pr/$1 origin/pr/$1
  git rebase master
  git checkout master
  git merge pr/$1 --ff-only
}

$ pr 123
```

It will pull down the PR into a local branch, using [the special refs GitHub provides](https://help.github.com/articles/checking-out-pull-requests-locally/). Then it will rebase the PR's commits on top of `master`, and do a fast-forward only merge into `master`. You should then amend the commit message with a final line containing `PR https://github.com/whatwg/html/pull/XYZ`, so that we can easily see a link back to the pull request's discussion. Finally, you can do `git push origin master` to push the changes, and comment on the pull request with something like "Merged as 123deadb33f" before closing.

If you would like to review a pull request's changes locally, you can manually do the first few steps of that script:

```bash
$ git fetch origin refs/pull/$1/head:refs/remotes/origin/pr/$1 --force
$ git checkout -b pr/$1 origin/pr/$1
$ git rebase master
```

(substituting the pull request number for `$1`). This will leave you on a branch named `pr/$1` that is rebased on top of `master`, which you can then e.g. build to check for errors or review the output.

### Merging pull requests from branches

Pull requests from other editors or members of the WHATWG GitHub organization may come from branches within this repository. Here is a function that you can use to merge such PRs:

```bash
mypr () {
  git checkout $1
  git rebase master
  git push origin $1 --force
  git checkout master
  git merge $1 --ff-only
}

$ mypr branch-name
```

It will rebase the PR on top of `master`, then force-push it to the appropriate branch, thus updating the PR. Then it will do the fast-forward only merge into `master`. At this point you can do a `git push origin master` to push the changes, which will _automatically_ close the PR and mark it as merged, since you managed to update the commits contained there.

To review the changes locally, you can do

```bash
$ git checkout $1
$ git rebase master
```

to get a local checkout of the branch, rebased on `master`.

## Bugs

There are three sources of bugs we should be managing:

- [This repository's GitHub issue tracker](https://github.com/whatwg/html/issues)
- [The remaining bugs from the W3C's WHATWG/HTML Bugzilla component](https://www.w3.org/Bugs/Public/buglist.cgi?bug_status=UNCONFIRMED&bug_status=NEW&bug_status=ASSIGNED&bug_status=REOPENED&component=HTML&list_id=59457&product=WHATWG&query_format=advanced&resolution=---)
- [Some bugs from the W3C's HTML WG/HTML5 spec Bugzilla component](https://www.w3.org/Bugs/Public/buglist.cgi?bug_status=UNCONFIRMED&bug_status=NEW&bug_status=ASSIGNED&bug_status=REOPENED&component=HTML5%20spec&f1=status_whiteboard&list_id=61030&o1=notequals&product=HTML%20WG&query_format=advanced&v1=whatwg-resolved)

### Handling bugs in W3C Bugzilla

Bugs in the WHATWG/HTML component should be RESOLVED MOVED when we create a GitHub issue or a pull request for it, while adding a comment linking to the new issue or pull request URL.

Some bugs that are not filed in that component might still be relevant for us; these are mostly captured by the above search, although it's possible there are other components where people are filing relevant bugs. When we fix such bugs, or if you notice such a bug that is already fixed or doesn't apply, add `whatwg-resolved` to the bug's whiteboard field, which will ensure that it disappears from the above search and does not show up in the margin of the generated spec.
