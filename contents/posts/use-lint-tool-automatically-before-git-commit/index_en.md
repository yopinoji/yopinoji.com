---
title: "Detect Git commands and automatically run ESLint and Prettier to maintain code quality"

category: "Tech"
lang: "en"
date: "2020-05-10"
slug: "en/use-lint-tool-automatically-before-git-commit"
draft: false
template: "posts"
tags:
  - npm
  - ESLint
  - Prettier
  - Git
---

This article is about how to run ESLint and Prettier automatically before `git commit` in source code managed by Git.

This article uses the relatively popular ESLint and Prettier as examples, but other tools such as stylelint and textlint can be used as well.

The idea is to run these code formatters automatically when you `git commit` to keep your code clean and readable.

## Prerequisite.

Projects that meet the following requirements are eligible.

- Node.js must be running in the environment.
- A project that uses npm or yarn for package management.
- The project must be using Git to manage source code.

## About npm packages to use

### husky

This package is for hooking (detecting and catching) Git commands.

It can hook `git commit` and `git push`.

Originally, it seems to have been created to run unit tests automatically when a Git command is used.

https://www.npmjs.com/package/husky

### lint-staged

This is a package to run the Lint tool against a staged Git file.

A staged Git file is, in essence, a file that has been `git added`.  
This means that you can efficiently run the Lint tool only on files that have been modified.

https://www.npmjs.com/package/lint-staged

### eslint

It is the most famous Lint tool in the frontend world.

If you use it well, you will be able to write relatively clean source code.  
I'm not going to explain how to set it up this time.

https://www.npmjs.com/package/eslint

### prettier

This is a code formatter.  
It formats your code into a clean and readable form.

It supports not only JavaScript and TypeScript, but also JSON, CSS, YAML, and many other types of files.

https://www.npmjs.com/package/prettier

## Try to run the Lint tool automatically by detecting git commit.

This time, we will use npm.

We'll start by installing the packages we want to use.  
If you have already installed ESLint or Prettier, you can skip the installation command.

```bash
npm i husky lint-staged eslint prettier -D
```

Next, add the following code to `package.json`.  
The extension of the target file should be changed according to the target project.

```json
{
  "scripts": {
    "eslint:format": "eslint --ext .js,.ts,.vue,.jsx,.tsx --ignore-path .gitignore .",
    "prettier:format": "prettier '**/*.{js,jsx,ts,tsx,vue}' --write"
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.js": ["prettier --write", "eslint --fix"],
    "*.ts": ["prettier --write", "eslint --fix"],
    "*.jsx": ["prettier --write", "eslint --fix"],
    "*.tsx": ["prettier --write", "eslint --fix"],
    "*.vue": ["prettier --write", "eslint --fix"],
    "*.json": ["prettier --write"]
  }
}
```

In the above example, the script defined under `lint-staged` in `package.json` will run when you commit on Git.

```json
{
  "scripts": {
    "lint-fix:scripts": "eslint --fix \"src/**/*.{ts,tsx}\" \"tests/**/*{ts,tsx}\"",
    "lint-fix:style": "stylelint --fix \"src/**/*.{css,scss,sass,tsx}\""
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{ts,tsx}": ["yarn lint-fix:scripts"],
    "*.{tsx,css,scss,sass}": ["yarn lint-fix:style"]
  }
}
```

If you want to use the npm script, you can also use your own script by writing as above.  
I think this is preferable from the point of view of being able to reuse the script once it is defined and use it in common.

Now all you have to do is to try it out and see how it works.

```bash
git add .
git commit -m "Add automation lint module"
```

If husky and `lint-staged work as shown below, you are done.

```bash
husky > pre-commit (node v12.16.1)
  ✔ Preparing...
  ✔ Running tasks...
  ✔ Applying modifications...
  ✔ Cleaning up...
🔍  Finding changed files since git revision 8c2a1d5.
🎯  Found 1 changed file.
✍️  Fixing up content/2020-04-30-quick-build-graphql-server-by-hasura-with-nuxt-js/index.md.
✅  Everything is awesome!
[master 904e41a] Modify content
 1 file changed, 2 insertions(+), 2 deletions(-)
```
