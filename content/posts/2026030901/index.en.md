+++
title = "What to Do If GitHub Contributions Are Missing (gitmailmap)"
date = '2026-03-09T22:00:40+07:00'
draft = false
lang = "en"
slug = "github-contributions-missing-gitmailmap"
tags = ["git", "github", "gitmailmap", "filter-repo"]
+++

[မြန်မာဘာသာဖြင့် ဖတ်ရှုရန်]({{< relref path="2026030901/index.my.md" lang="my" >}})

## Introduction

Everyone who uses GitHub wants to see those green contribution graphs on their profile. But sometimes, you might be writing code and pushing every day, yet your contributions don't show up on GitHub.

The main reason for this is that the email configured in your local Git config is different from the email associated with your GitHub account.

In this post, I will share why this happens and how you can update your past commits using [`gitmailmap`](https://git-scm.com/docs/gitmailmap) and `git filter-repo`.

## Why are Commits Missing?
### 1. Check Local and GitHub Emails
First, let's check why your commits are not appearing.

Run the following command in your terminal to check the email set up in your local Git config:

```bash
git config user.email
```

The email that appears there must match the email you added in GitHub under Settings -> Emails. That email also needs to be "Verified".

### 2. Check if Commits are Pushed to GitHub

If you only commit locally and haven't run `git push`, it won't appear in your contributions. Also, if you commit to a branch that isn't the default branch (usually `main` or `master`) and haven't merged it, it might not show up. Furthermore, in private repositories, contributions might be hidden if the setting to show private contributions on your profile is turned off.

## What if the Emails are Different?
If you've confirmed that the emails are different, you first need to update to the correct email for your future commits.

```bash
git config --global user.name "Your Name"
git config --global user.email "your-real-email@example.com"
```

Once you change this, all your new commits will automatically appear as contributions on GitHub. If you want to keep this specific to the current project, use it without `--global`.

## How to Edit Past Commits?
The problem lies with the old commits made before you changed the email. They already contain the incorrect email. If you want to replace them with the correct email and name without changing the date-time, you need to use `.mailmap` together with `git filter-repo`.

### 1. Install `git filter-repo`

`git filter-repo` is a tool that is faster and safer than Git's built-in filter-branch.

- On macOS: `brew install git-filter-repo`
- On Linux or Windows: `pip install git-filter-repo`

*Before doing anything, I highly recommend making a backup of your repository locally.*

### 2. Create a .mailmap File

Create a file named `.mailmap` in the root directory of your project. Insert the following format into that file:

```text
Kyaw Kyaw Myo <your-real-email@example.com> <user@github.com>
```

Here:
- Replace `Kyaw Kyaw Myo` with your real name
- Replace `your-real-email@example.com` with your verified email on GitHub
- Replace `user@github.com` with the incorrect email you previously used.

### 3. Rewrite History

Once you have saved the `.mailmap` file, run the following command:

```bash
git filter-repo --mailmap .mailmap
```

This command will read through your commit history and replace the incorrect emails with the correct ones based on the information in `.mailmap`. The original commit dates will remain unchanged.

## Force Push
Because you have rewritten the commit history, a simple `git push` will no longer work. You need to do a force push.

```bash
git push --force-with-lease --all origin
git push --force-with-lease --tags origin
```

*Since this overwrites the repo's history, you should only do this for personal repos where you are the only contributor. If this is a team project, beware that it can impact other team members who have cloned the project.*

## Conclusion
After doing a force push and waiting a moment, if you check your GitHub profile, you will see the missing green blocks reappear. I hope this post is helpful for developers who are frustrated by their missing contribution graphs.
