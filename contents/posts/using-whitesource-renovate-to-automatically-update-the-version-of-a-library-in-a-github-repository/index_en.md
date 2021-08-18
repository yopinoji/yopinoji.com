---
title: "Using WhiteSource Renovate to automatically update the version of a library in a GitHub repository"

category: "Tech"
lang: "en"
date: "2021-01-07"
slug: "en/using-whitesource-renovate-to-automatically-update-the-version-of-a-library-in-a-github-repository"
draft: false
template: "posts"
tags:
  - GitHub
---

In the field of software development, it is now common to use publicly available libraries such as OSS.

Since such libraries can be vulnerable, it is essential to update their versions on a regular basis.  
However, it is troublesome to manually update the version of the library on a regular basis.

WhiteSource Renovate is for you.

## WhiteSource Renovate

![WhiteSource Renovate](WhiteSource_Renovate.png)

I've just taken an excerpt from the official site, but it allows you to do the following

- Software updates with dependencies such as libraries can be automated
- Fully customizable with settings to suit any workflow

For more details, please read the following.

https://www.whitesourcesoftware.com/free-developer-tools/renovate

## Automatically update the version of a library in a GitHub repository.

If you want to use it on GitHub, you can install it from

https://github.com/marketplace/renovate

Once you install WhiteSource Renovate on GitHub and decide which repositories you want to enable your app for, a pull request will be created as follows.

![Configure Renovate](Configure_Renovate.png)

You can change the behavior of WhiteSource Renovate by changing the settings that are added at this time.

The available options can be seen in detail below.

https://docs.renovatebot.com/configuration-options/

![Pin dependecies](Pin_dependecies.png)

After a while, WhiteSource Renovate will create a pull request to update the library.

![bot make PR](renovate_bot_make_PR_1.png)

![bot make PR](renovate_bot_make_PR_2.png)

It is possible that the library update will cause your app to stop working, so you might want to check that it works just in case.  
[It is safe to use GitHub Actions to check the behavior of each pull request.](/use-github-actions-to-check-build-is-passed-for-each-pr)

In some cases, you may have conflicts in Git that prevent you from merging.

![bot make PR](renovate_bot_make_PR_2.png)

In such cases, WhiteSource Renovate will detect the conflict and perform a `git rebase` to resolve the conflict.

How convenience!
