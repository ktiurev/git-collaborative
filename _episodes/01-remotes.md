---
layout: episode
title: Working with remotes
teaching: 20
exercises: 10
questions:
  - How can we keep repositories in sync?
  - How can we share repositories with others?
objectives:
  - Understand the difference between local branch, origin/branch, and remote branch.
keypoints:
  - "`git clone` copies everything and sets some pointers to remember where the clone came from."
  - You communicate commits with `git fetch`/`git pull` and `git push`.
  - All other Git operations are offline - you can work on a plane while your coworker is on vacation in North Korea.
  - "`origin` refers to where you cloned from (but you can relocate it)."
  - "`origin/foo` is a read-only pointer to branch `foo` on origin."
  - "`origin` pointers only move when you `git fetch`/`git pull` or `git push`."
---

## From local repositories to remote repositories

- In contrast to other version control tools we do not contribute to a repository
  through a lightweight working copy.
- In Git we often work within a clone.
- Contributing to a repository often starts by cloning the entire repository.

---

## Cloning repositories

```shell
$ git clone https://host.com/user/project.git project
```

- A clone is a full-fledged repository.
- Think of `git clone` as a `scp -r` "plus".
- We will see what the "plus" means.
- By cloning we clone all commits, all branches and tags, **entire history**.

---

## When push comes to pull

- We need a mechanism to communicate changes between the repositories.
- We will **pull** or **fetch** updates **from** remote repositories.
- There is a difference between pull and fetch and we will soon discuss what the difference is.
- We will **push** updates **to** remote repositories.

---

## Working with others

- We collaborate with other people through clones by pulling/fetching and pushing changes.
- Everybody typically works on own clones.
- Sometimes one person works on several clones (typically on different machines).

---

## Cloning repositories

This is a representation of what happens when you clone:

*remote*: ![]({{ site.baseurl }}/img/distributed/remote-02-remote.svg)

*local*: ![]({{ site.baseurl }}/img/distributed/remote-03-local.svg)

- We clone the entire history, all branches, all commits (in this example two branches, four commits).
- `git clone` creates pointers `origin/master` and `origin/dev`.
- `origin` refers to where we cloned from, try: `git remote -v`.
- `origin` is a shortcut for the full URL.
- `origin/master` and `origin/dev` are read-only pointers.
- They only move during `git pull` or `git fetch` or `git push`.
- Only `git pull` or `git fetch` or `git push` require network.
- All other operations are local operations.

---

## Fetching updates

Let us imagine that the remote repository receives a new commit:

*remote*: ![]({{ site.baseurl }}/img/distributed/remote-06-remote.svg)

*local*: ![]({{ site.baseurl }}/img/distributed/remote-03-local.svg)

Pointer is moved (pushed) by someone else (could be you).

Nothing changes on the local repository until we pull/fetch these changes over the network:

```shell
$ git fetch origin
```

`git fetch origin` updates `origin/master` and `origin/dev`:

*remote*: ![]({{ site.baseurl }}/img/distributed/remote-06-remote.svg)

*local*: ![]({{ site.baseurl }}/img/distributed/remote-04-local.svg)

In a second step we merge `origin/master` (in this case fast-forward; further below we explain what fast-forward means):

```shell
$ git merge origin/master
```
This updates the local branches:

*remote*: ![]({{ site.baseurl }}/img/distributed/remote-06-remote.svg)

*local*: ![]({{ site.baseurl }}/img/distributed/remote-05-local.svg)

---

## Fetch vs. pull

What is the difference between `git fetch` and `git pull`?

So far we have fetched and merged in two steps:

```shell
$ git fetch origin
$ git merge origin/master
```

This is equivalent to:

```shell
$ git pull origin master
```

- `git pull` consists of two operations: a `git fetch` followed by a `git merge`.
- Summary: `git pull origin master` fetches `master` from `origin` and merges it.
- There is always a `git merge` "hidden" in `git pull`.
- Many people will simply `git pull`, very careful people first `git fetch` and inspect the commits before merging them.
- With Git you typically merge several times a day without even noticing.

---

## Publishing local commits

We can commit locally and
these commits are not visible to others until we `git push`.

Observe that `master`, `origin/master`, and `master` on remote repository are 3 different pointers:

*remote*: ![]({{ site.baseurl }}/img/distributed/remote-06-remote.svg)

*local*: ![]({{ site.baseurl }}/img/distributed/remote-06-local.svg)

Now let us push the local commits upstream:

```shell
$ git push origin master
```

Only now the remote `master` as well as `origin/master` move:

*remote*: ![]({{ site.baseurl }}/img/distributed/remote-07-remote.svg)

*local*: ![]({{ site.baseurl }}/img/distributed/remote-07-local.svg)

We commit `d8` (the color is to signal that we differ, we are still on
`master`)
and in the meantime remote receives commit `c8` from someone else:

*remote*: ![]({{ site.baseurl }}/img/distributed/remote-12-remote.svg)

*local*: ![]({{ site.baseurl }}/img/distributed/remote-08-local.svg)

What happens if we try to `git push origin master`?

`git push` is rejected:

```shell
$ git push
To https://github.com/user/repo.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'https://github.com/user/repo.git'
To prevent you from losing history, non-fast-forward updates were rejected
Merge the remote changes (e.g. 'git pull') before pushing again.  See the
'Note about fast-forwards' section of 'git push --help' for details.
```

The natural reflex is now to `git pull` first but
what happens if we `git pull origin master`?

*remote*: ![]({{ site.baseurl }}/img/distributed/remote-12-remote.svg)

*local*: ![]({{ site.baseurl }}/img/distributed/remote-09-local.svg)

```shell
$ git pull origin master
```

Explanation:

- First we fetched (`origin/master` moved),
  then we merged `origin/master` to `master` (`master` moved).
- Local master and remote master are two different branches.
- If they diverge, Git will merge them during `git pull`.

You can avoid the merge commits using `--rebase`:

```shell
$ git pull --rebase origin master
```

This will replay your unpublished local master commits at the end of `origin/master`:

*remote*: ![]({{ site.baseurl }}/img/distributed/remote-12-remote.svg)

*local*: ![]({{ site.baseurl }}/img/distributed/remote-10-local.svg)

Note how `d8` changed to `d8*`.

---

## When is a good moment to pull?

- Local master and remote master are two different branches
- Local feature branch and remote feature branch are two different branches
- `git pull` fetches and merges
- If you never pull then the branches may diverge
- `git pull` often to stay in sync with upstream development
- `git push` whenever you want other people to know about your changes
- If you never `git push` others will not see your changes
- Nontrivial changes should not be done on master

---

## When is a good moment to fetch?

Whenever, you can always decide whether or not to merge.

---

## Tracking branches

We want to make changes to `origin/dev`:

*remote*: ![]({{ site.baseurl }}/img/distributed/remote-12-remote.svg)

*local*: ![]({{ site.baseurl }}/img/distributed/remote-10-local.svg)

For this we create and switch to local branch `dev` tracking `origin/dev`:

```shell
$ git checkout -b dev origin/dev
```

*remote*: ![]({{ site.baseurl }}/img/distributed/remote-12-remote.svg)

*local*: ![]({{ site.baseurl }}/img/distributed/remote-11-local.svg)


### Shortcut to checkout tracking branches

If there is no local branch `dev` and there is a remote branch `origin/dev`, then both are equivalent:

```shell
$ git checkout -b dev origin/dev
```

```shell
$ git checkout dev
```

As we commit to `dev` the pointer moves while `origin/dev` does not:

*remote*: ![]({{ site.baseurl }}/img/distributed/remote-12-remote.svg)

*local*: ![]({{ site.baseurl }}/img/distributed/remote-12-local.svg)

We can now push `origin/dev` "forward":

```shell
$ git push origin dev
```

With the result:

*remote*: ![]({{ site.baseurl }}/img/distributed/remote-13-remote.svg)

*local*: ![]({{ site.baseurl }}/img/distributed/remote-13-local.svg)

What if you have created a local branch and want to make it public? Push it upstream:

```shell
$ git checkout -b cool-branch     # create and switch to cool-branch
$ git commit                      # work and commit
$ git push -u origin cool-branch  # push to origin and set as upstream
```

We can also delete remote branches:

```shell
$ git push origin --delete cool-branch
```

---

## Two types of repositories

### Non-bare repository

- A non-bare repository contains `.git/` as well as a snapshot of your tracked files that you can directly edit called **the working tree**.
- This is where we work.

### Bare repository

- A bare repository contains only the `.git` part.
- By convention the names of bare repositories end with `.git` to emphasize this.
- We only push to bare repositories.
- We never do actual work inside a bare repository.

---

## Exercise: practice working with remote repository

Objectives:

- Practice cloning, pulling and pushing
- Get used to working with remotes

This exercise involves setting up a remote repository on your local machine. In the 
next episode, we look at more realistic scenarios involving remote repositories on GitHub.

1. Set up a "remote" repository for a [guacamole recipe](https://github.com/coderefinery/guacamole) on your local machine:
   - Clone a *mirror* using the `git clone --mirror` command.
   - This creates a fully-fledged "remote" repository on your machine.
   - Note the `.git` part of the directory name. This is how bare repositories are usually named.
   - Inspect the contents of the `guacamole.git` directory. Is it possible to work there?
2. After mirror cloning, try to clone in the normal way with `git clone /your/local/path/guacamole.git`.
   - Enter the (normally) cloned repository, and inspect the remotes using the `git remote -v` command.
3. Make a commit in your cloned repository, and push it to your (local) remote.

![]({{ site.baseurl }}/img/distributed/mirroring-1.svg)

### EXTRA: 

If you have time and want to practice resolving conflicts you can experiment with the following steps:
7. Clone your remote (`/your/local/path/guacamole.git`) to another location on your machine.
8. Make a commit in this clone, and push it.
9. Go back to your first clone, make a different commit, and try pushing. Your push will be rejected since the remote repository has commits that you haven't pulled yet.
10. Do a `git pull` instead. If you get merge conflicts, try to resolve them, commit again and push. 
Notice how you get a merge commit in your history.

![]({{ site.baseurl }}/img/distributed/mirroring-2.svg)

---

