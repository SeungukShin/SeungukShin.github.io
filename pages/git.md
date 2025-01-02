---
title: Git
---

# Git
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

## Commands

Create a repository
```sh
$ mkdir <project>
$ cd <project>
$ git init --bare --shared
$ cd ..
$ sudo chown -R <git user>.<git group> <project>
$ sudo chmod -R g+w <project>
```

Change a repository to shared one
```sh
$ cd <project>
$ git config core.sharedrepository true
$ sudo chown -R <git user>.<git group> <project>
$ sudo chmod -R g+w
```

Archive a repository
```sh
$ git archive <branch/tag> | bzip2 > <output.tar.bz2>
```

Create a patch
```sh
$ git format-patch -1
```

Push a branch
```sh
$ git push <remote> <local branch>:<remote branch>
# or
$ git push <remote> <local branch>
```

### Change a Commit Message

Rebase tree to the commit right before the commit to change
```sh
$ git rebase -i <the commit right before>
```

Change the line includes the commit to change from pick to edit

Change the commit message
```sh
$ git commit --amend --author="author <author@server.com>"
```

Commit the change
```sh
$ git rebase --continue
```

Push the change
```sh
$ git push -f
```

## References

* [Pro Git](https://git-scm.com/book/en/v2)
