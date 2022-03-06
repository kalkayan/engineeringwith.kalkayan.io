---
title: "Dotfiles Are Easier Than You Might Think"
date: 2022-02-11T01:52:15+05:30
draft: true
description: >
    Dotfiles are all about boosting productivity by tailoring the machine's configurations to the developer's personalized needs. This article looks into the straightforward but powerful configurations one can have and master within a few minutes.
tags: ["tooling", "bash", "zsh"]
categories: "development-experience"
---

Regardless of the surge in graphical interfaces in the market, however fancy they are, most development work still happens via a terminal. Be it a bash, zsh, or fish, developers had leaned upon these terminals long before any graphical interface existed. Terminals have proven to be productive, and using them can be extraordinarily compelling but sometimes tiresome at the same time. 

Commands in the terminal can carry out incredibly complex procedures if executed dashingly. Consider such an example: we delete all the `.DS_Store` files in the current directory and sub-directories. We can use the `find` program to perform this by running the following command. 

```bash
find . -type f -name '*.DS_Store' -ls -delete
```

The command is pretty self-explanatory, `-type f` specifies to list only files with names ending with `.DS_Store` and the `-ls -delete`  part is for deleting the files in the list. 

Without a doubt, this single command is highly convenient and much needed in regular day work, especially when we don't want these files messing up a git repository (All the allies of `git add .` say Aye!). Not only this, it can work with any other type of file: say you need to clear all the log files whose name starts with python - Easy! Replace the`*.DS_Store` with `Python*.log` and so on.

Shortly, You may realize it's not that fun writing the whole command repeatedly. This is where the magic of bash and dotfiles come into action. You can **alias** the above long command to a single word by running the following:

```bash
alias cleanup="find . -type f -name '*.DS_Store' -ls -delete"
```

Whenever you wish to remove all the `.DS_Store` files in a directory, just type `cleanup`, hit enter, and Voila no more of these files.

Cool? Isn't it? The rest of the article expands on creating such configurations in a manageable way.

# Meat and potatoes 