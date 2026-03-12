+++
title = "How to Edit Git Commits (git reset --soft)"
date = '2026-03-12T23:30:41+07:00'
draft = false
lang = "en"
slug = "git-reset-soft-head-tips"
tags = ["git", "github", "git reset", "tips"]
+++

[မြန်မာဘာသာဖြင့် ဖတ်ရှုရန်]({{< relref path="2026031101/index.my.md" lang="my" >}})

## Introduction
When using Git, you have probably run into a situation where you forgot to include a file in a commit, or made a typo in your commit message. In those cases, `git reset --soft` lets you undo that commit and fix things up — without losing any of your work.

In this post, I'll share how to use `git reset --soft HEAD^` and `git reset --soft HEAD~n` to edit your most recent commit and older commits.

## Editing the Last Commit (git reset --soft HEAD^)
If you want to undo your last commit and put your files back into the staging area, run this command:
```bash
git reset --soft HEAD^
```

This moves `HEAD` back one step to the parent commit. Because we use `--soft`, your files are not deleted, they stay in the staging area (index), ready to be re-committed. From there, you can add more files, remove files, or adjust anything you need before making a fresh commit.

## Editing Older Commits (git reset --soft HEAD~n)
If you want to undo not just the last commit but the last 2 or 3, you can use `HEAD~n`.

For example, to undo the last 3 commits and squash them all into one:
```bash
git reset --soft HEAD~3
```

This will undo those 3 commits and place all the changed files back into the staging area, letting you re-commit them as a single, clean commit.

## Other Useful Tips

### 1. Difference Between `--soft` and `--mixed`
If you use `git reset HEAD^` or `git reset --mixed HEAD^` (without `--soft`), your files will be moved out of the staging area into an unstaged (modified) state. You will need to run `git add` again before you can commit.

### 2. Be Careful with `git reset --hard`!
Using `git reset --hard HEAD^` does not just undo the commit — it also **permanently deletes** all the file changes in that commit. If you only want to undo the commit while keeping your code, make sure to use `--soft`.

### 3. Pushing to a Remote After Resetting
If you have already pushed the commits you just reset, a normal `git push` will be rejected because you have rewritten history. You will need to force push:
```bash
git push --force-with-lease origin main
```

*`--force-with-lease` is safer than `--force` because it prevents accidentally overwriting commits that someone else may have pushed.*

## Conclusion
`git reset --soft` is one of the most useful tools in Git — whether you want to clean up your commit history, combine multiple commits into one, or simply fix a mistake without losing your work. I hope these tips make your day-to-day coding a little smoother.
