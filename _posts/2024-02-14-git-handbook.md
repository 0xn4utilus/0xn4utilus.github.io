---
title: Git Handbook
author: nautilus
date: 2024-02-14 08:30:30 +0530
categories: [Git,Development]
tags: [git,github]
---
Trying to save time I spend revisiting the same git stackoverflow answers.

## Git?

It is [git](https://git-scm.com/book/en/v2).

## Branching 101

Read more from [here](https://www.atlassian.com/git/tutorials/using-branches).

### Working locally

```bash
# list all the local branches
git branch

# list all the remote branches
git branch -a

# list all branches verbose
git branch -v -a

# just make new branch
git branch <branch>

# switch to branch
git switch <branch>

# delete branch
git branch -d <branch>

# force delete branch
git branch -D <branch>

# rename current branch
git branch -m <branch>

# make a new branch, switch current working directory
git checkout -b <branch>

# if branch already exists
git checkout <branch>

```

### Working with Remote

```bash
# list remote 
git remote

# list remote with links
git remote -v

# add new remote
git remote add <name> <url>

# add remove remote
git remote rm <name>

# rename remote
git remote rename <old_name> <new_name>

```

Reset local branch to match remote branch

```bash
git fetch origin
git branch -v -a
git reset --hard <remote>/<branch>
```

How do I check out a remote Git branch?

```bash
# create a new branch of name test
git fetch origin
git branch -v -a
git switch -c test origin/test
```

Squash last `n` commits into single commit

```bash
# squash last 2 commits into single commit
git reset --soft HEAD~2
git commit
```
