---
title: "Storing dotfiles with Git - This is the way"
date: 2022-02-11T18:15:45+05:30
draft: false
author: Manish Sahani
description: > 
    Learn an easy yet elegant way of storing dotfiles using just a Git repository - no symlinks, no extra tooling, and no scripts required.
tags: ["tooling", "git", "git-bare"]
categories: "development-experience"
---

Shaping dotfiles takes time, patience, and consistent improvement. Undoubtedly, no one wants to lose all that hard work, and there is no argument against Git repositories being the best way to preserve and protect any source code files. Ultimately, any method for storing dotfiles uses a Git repository.

You may be asking, why are we making such a big fuss about it if we use Git in the end? Because these dotfiles are highly volatile, they change more than you expect them to, so the process of keeping them in sync with the backup repository needs to be efficient and one with the least effort. In this article, we discuss various ways to sync these dotfiles with the backup repository. 

# This is the way! Why?

One of the ways we could track dotfiles is by copying them to a backup repository everytime we make a change. But, this neither feels effective nor productive, as there is a redundant copy for each dotfile, and you need to move them around (from home to git repo) to keep them in sync. We can eliminate the extra copy if we symlink the files in the Git repo to the home directory. This brings us to the second way we could track these dotfiles.

Symlinking dotfiles is effective, and you don't need to move them around again and again. In fact, this is the most popular way of tracking them as it's only natural to do so. However, it shortly falls into issues, doesn't scale up and is a no for sure. Dangling symlink is one such issue, but using a symlink manager like GNU Stow solves all the problems with symlinking. 

Stow is prevalent in the Linux community, and you will see many developers preferring this method. It works, but it needs to be physically present on the machine, thus adding installation overhead. This dependency on an external tool (stow) doesn't board well while installing dotfiles on a new machine. A neat way of setting up a new system is with a single command (Read more about this), that is independent of any other softwares. So, there goes the option to use stow.  

You may also find some scripts/tools explicitly written to manage these dotfiles. But, this is an overkill, in my opinion.

Ideally, managing dotfiles should be quite simple, familiar to any developer, does not require anything to learn, and most importantly, does not depend on other softwares. You may be asking, then what could be used to sync them? And the answer is *nothing* - yes, you don't need anything other than good old Git to keep them in sync. 

Git has everything built in to do this work, and we don't need to have an extra copy, symlink, or use stow to keep the files in sync. Almost everyone is familiar with Git, if it isn't already their second language. It comes preinstalled with macOS, and works remarkably well for managing dotfiles when provided a `--bare` flag. 

# Git's hidden gem: `--bare`

Bare repositories are special in the way that they omit the working directory. So, you can track files outside your Git repository, which is what we wanted all along. This does not mean doing `cd $HOME && git init .` and creating an exhaustive `.gitignore` with all the home directory content excluded from tracking (this is a horrible idea). 

The trick to managing only a certain dotfile present in the `$HOME` directory as opposed to the entire directory itself is by creating a bare Git repository. As a first-time user of this method, provide `--bare` flag while creating your backup repository.

```bash
# Notice the --bare flag; this initializes the repository as a git bare
git init --bare $HOME/.dotfiles
```

Bare repositories require a work-tree and git-directory on usage. Since the dotfiles live in the `$HOME` directory, the `--work-tree` argument is provided with `$HOME` as the value. This option merely tells Git where to look for the files; setting this to `$HOME` does not track its content. Apart from the working directory, Git also needs to know where the information about tracking lives. Therefore, `--git-dir` flag is set to the location of the bare repo, i.e., `$HOME/.dotfiles`. 

So, the backup repository can be operated with `git --git-dir=$HOME/.dotfiles --work-tree=$HOME` in the starting. For example, to check its status, run:

```bash
git --git-dir=$HOME/.dotfiles --work-tree=$HOME status
```

Alias this mouthful command to `dotfiles` to use with ease.

```bash 
alias dotfiles="/usr/bin/git --git-dir=$HOME/.dotfiles --work-tree=$HOME" 
```

That was all for the setup, just one tiny flag, and one alias. Simple enough? There is no need to install anything, no chances of running into installation/build issues, or any other issues.

# Git prowesses for dotfiles

Tracking dotfiles are just like tracking files in any other Git repository. The regular Git workflow takes over, i.e., add, commit, push. 

Example - Adding a file (`.vimrc` in this example) for tracking.

```bash
#  Notice, add and commit are done together. Feel free to use separate commands for the same.
dotfiles commit .vimrc -m ".vimrc added."

# Push new changes to GitHub, Bitbucket, Gitlab, or anywhere else.
dotfiles push origin main
```

Example - To check the state of the dotfiles repo.

```bash
# to check the status of the tracked and untracked files
dotfiles status
```

The output of the above status command may contain all the home directory files. It is normal Git behavior to show all the untracked files in the directory. The fix for this is relatively simple, by updating the `git-config` for the dotfiles repo.

``` 
# Hides all the untracked files in the output
dotfiles status.showUntrackedFiles no 
```

With this done, managing dotfiles is good to go. The simplicity of tracking dotfiles, versioning, and managing is remarkable with Git. 

# Beauty is in the minds of people

Besides painless management and sharing, this particular method has various other advantages like:

#### Effortless Installation

The pre-eminence of using a bare repository for dotfiles is in installing them on a new machine. Just drop the `--bare` flag while cloning. 

```bash 
git clone --bare https://github.com/<username>/dotfiles.git $HOME/.dotfiles && source ~/.zshrc 
```

#### Version History 

If breaking things is a ritual, Git promotes experimenting with new configurations, eliminating the pain of finding the way back to the last working version. Basically, all the pros of using Git.

```bash 
# To check the version history 
dotfiles log 

# To checkout to last working version
dotfiles reset <commit-hash> --hard/soft
```

#### Access across multiple devices

Share the same configs to multiple devices with minimal changes using **git branches**. Create a branch for your new machine.

```bash 
# Create configurations specific to your AWS machines 
dotfiles checkout -b aws && source ~/.zshrc   
``` 

#### Environment enabled configurations profile

Allows the separation of configurations by creating profiles as **git branches** based on your environment. Create a branch, and update the configurations according to the env. 

```bash 
# Checkout to different profiles as per the environment using branches
dotfiles checkout work && source ~/.zshrc
``` 

The list doesn't end here. These are just some examples that I use daily. One can develop more practical usage of tracking dotfiles with Git. 


To recap, we have discussed a few popular ways to track and manage dotfiles. You can find chatters on the internet about *which of these solutions is the best*. A better question than this would be which solution serves your needs? Perhaps, the answer for this question is biased on the developer's working style: some may lean more toward something they already know (ex - stow), few want the bare minimum setup (ex - git), even fewer are more satisfied with the added functionality of their tool. Ultimately, elegant solutions don't have to be fancy. It's good to use something familiar for better use of time. Always prefer less/no dependency on other softwares. Most importantly, try not to get overwhelmed by the things on the internet - pick the solutions that satisfy your needs. 

Anyway, thanks for the read. If you have any thoughts on this, or just want to say hi, feel free to reach out to me at [rec.manish.sahani@gmail.com](mailto:rec.manish.sahani@gmail.com) or connect on [LinkedIn](https://www.linkedin.com/in/manishsahani/).