+++
title = "How to Update Past Commits"
date = '2026-03-06T22:53:52+07:00'
draft = false
lang = "en"
slug = "update-past-commits"
tags = ["git", "github", "version-control"]
+++

## Introduction
When pushing code, writing clear commit messages is very important. But sometimes, you might make a typo in the commit message or want to change it to something clearer.

In this post, I will share how you can update your past commits using `git commit --amend` and `git rebase`.

## Updating the Latest Commit?
If the commit you want to edit is the latest one, it's very easy.

You can simply use the `git commit --amend` command.

```bash
git commit --amend
```

Running this command will open a text editor in your terminal. There, you can write the new commit message and save it.

> **Pro Tip:** If you forgot to include some files in the last commit and just want to add them without changing the commit message, you can easily do so by running `git add .` followed by `git commit --amend --no-edit`.

## Updating Older Commits?
If the commits you want to edit are not the latest ones, you'll need to use `git rebase`. For example, if you want to edit the last 3 commits, you can use **interactive rebase** as follows:

```bash
git rebase -i HEAD~3
```

When you run this command, an editor (maybe Vim) will open, showing a list of commits.

Example:
```text
pick e3a1b35 Commit message 1
pick 7ac9a67 Commit message 2
pick 4db8c21 Commit message 3
```

You need to change the word `pick` to `reword` in front of the commit you want to edit. In this example, `reword` is used for every commit.

```text
reword e3a1b35 Commit message 1
reword 7ac9a67 Commit message 2
reword 4db8c21 Commit message 3
```

After saving and exiting, the editor will reopen sequentially for each commit marked with `reword`. At each step, simply update the commit message to your liking and save.

***Pro Tip:*** In interactive rebase, besides `reword`, there are other useful commands. For example:
- Use `drop` (or `d`) if you want to delete an unnecessary commit
- Use `squash` (or `s`) if you want to merge two commits into one
- Use `edit` (or `e`) if you want to modify not just the commit message, but the files as well
- Use `git rebase --abort` if you make a mistake during the rebase and want to stop and return to the original state.

## Safety Net
Don't worry if you make a mistake or accidentally delete something while editing your commit history during a rebase. There is a command called `git reflog`.

```bash
git reflog
```

Through this command, you can view the history of all the actions you've taken, along with their commit hashes. From there, you can easily go back to anywhere you want using `git reset --hard <commit-hash>`.

## Push to Remote
Once you have finished editing your commits, you need to push them back to the remote repository. However, since the commit history has changed, a simple `git push` will no longer work.

In this situation, you need to do a **force push**. To avoid affecting other people's code and for safety, you should always use `--force-with-lease`.

```bash
git push --force-with-lease
```

## Conclusion
These commands are incredibly useful in an everyday development workflow. Being able to easily fix mistakenly written commits before code reviews helps keep the git history clean and clear.
