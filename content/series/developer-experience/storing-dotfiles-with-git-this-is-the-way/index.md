---
title: "Storing Dotfiles with git - This is the way"
date: 2022-02-11T18:15:45+05:30
draft: false
author: Manish Sahani
description: >
    Many nights of potential productive work, programmers have lost by procrastinating on properly managing their dotfiles. This article discusses an elegant way to manage and share dotfiles across machines using a single git repository.
tags: ["tooling", "git", "git-bare"]
categories: "development-experience"
---

It certainly takes much time to tailor a machine to fit the development needs. Moreover, this is a never-ending continous process, and the configurations evolve with experience and time. Even though configurations are an integral part of the day-to-day work of a software engineer, one may lose sight of protecting and maintaining it, and nights of potential productive work get lost due to procrastinating on properly storing these configuration files. 

There is massive debate out on the internet on which method is the best for managing dotfiles. It is easier to find more dotfile managers than the actual dotfiles. A few popular ways of managing dotfiles are - simple symlinks, stow, and a couple of other types of dotfiles managers. 

This article presents the most spartan yet elegant way to store, install and share these configuration's dotfiles using Git and nothing else.

# This is the way! Why?

There is an attempt to prove that "one of these solutions is the best" in most discussions. The question should not incline toward which is the best; instead, which solves the conditions of the developer. Perhaps, The answer is highly opinionated and changes with the developer's workflow and style. Some may lean more toward something they already know (ex - Git, stow), few want the bare minimum setup, few like added functionality of a tool, and some feel more satisfied with a tool of their own. 

Symlinking files to home from a git repository doesn't cut it and shortly falls into issues. Using a symlink manager like stow requires it to be present on the machine, thus adding installation overhead. The need to install a manager doesn't board well while installing dotfiles on a new machine. And,  In my opinion, writing scripts/tools for dotfiles management is overkill and over-engineering.

Ideally, managing these files should be dead simple, familiar to any developer, does not require anything to learn, and most importantly, does not require other software installation. I prefer setting up a new system with a single command (Read more about this - here), and the setup script is part of the dotfiles. Therefore, there goes the option to use stow, as non-dependence is an absolute must.  

Git has all the above characteristics. Developers are familiar with Git if it's already not their second nature; it comes preinstalled with macOS, i.e., non-dependency, and works remarkably well for managing dotfiles when provided a `--bare` flag while cloning. 

# Git with `--bare` for managing dotfiles

A naive way to manage dotfiles is to symlink the configurations files to $HOME from a routine git repo present somewhere on the disk. Symlinks works but soon runs into issues of managing the symlinks, dangling symlinks, and many more. Using farm managers like stow can address all the symlinks problems. But, Instead of installing stow, Why not have your files present in the $HOME directory and have a git repo track them as usual without tracking your entire home directory, only certain dotfiles.

The above method does not mean doing `cd $HOME && git init .` and creating an exhaustive .gitignore with all the home directory content in the file (this is a horrible idea). 

The trick to managing only the certain dotfiles present in $HOME and not the entire $HOME directory is by creating a bare git repository. As a first-time user of this method, make a bare repository in the $HOME directory. 

![code 1](https://carbon.now.sh/?bg=rgba%28171%2C+184%2C+195%2C+1%29&t=nord&wt=none&l=application%2Fx-sh&width=795&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=false&pv=8px&ph=8px&ln=false&fl=1&fm=Hack&fs=14px&lh=133%25&si=false&es=2x&wm=false&code=alias%2520dotfiles%253D%2522%252Fusr%252Fbin%252Fgit%2520--git-dir%253D%2524HOME%252F.dotfiles%2520--work-tree%253D%2524HOME%2522%2520)


```bash
# Notice the --bare flag; this initializes the repository as a git bare
git init --bare $HOME/.dotfiles
```

Bare repositories are special in the way that they omit the working directory. Therefore, it requires a work-tree and git-directory upon the usage. Since the dotfiles live in the $HOME directory, the `--work-tree` argument is provided with $HOME as the value. This option merely tells the git where to look for files; setting this to $HOME does not track the content of the $HOME directory. Apart from the working directory, Git also needs to know where the information about tracking lives. Therefore, `--git-dir` flag is set to the location of the bare repo, i.e., `$HOME/.dotfiles`. 

The final command to use the repository has the prefix 'git --work-dir $HOME --git-dir $HOME/.dotfiles`. Alias this mouthful command to `dotfiles` to use with ease.

```bash 
alias dotfiles="/usr/bin/git --git-dir=$HOME/.dotfiles --work-tree=$HOME" 
```

That was all for the setup, just one tiny flag, and one alias. Simple enough? There is no need to install anything, no chances of running into installation/build issues, or anything. 

# Git Prowesses for Dotfiles

Tracking dotfiles are just like tracking files in any other git repository. The regular git workflow takes over, i.e., add, commit, push. 

Example - Adding a file (.vimrc here) for tracking.

```bash
#  Notice, Add and commit are done together. Feel free to use separate commands for the same.
dotfiles commit .vimrc -m ".vimrc added."

# Push new changes to GitHub, Bitbucket, Gitlab, or anywhere else.
dotfiles push origin main
```

Example - To check the state of the dotfiles repo.

```bash
# to check the status of the tracked and untracked files
dotfiles status
```

The output of the above status command may contain all the home directory files. It is normal git behavior to show all the untracked files in the directory. This fix for this is relatively simple, by updating the git-config (for dotfiles repo)

``` 
# Hides all the untracked files in the output
dotfiles status.showUntrackedFiles no 
```

With this done, managing dotfiles is good to go. The simplicity of tracking dotfiles, versioning, and history without symlink is remarkable. 

# Beauty is in the minds of people

Besides painless management and sharing, this particular method has various other advantages like:

#### Effortless Installation

The pre-eminence of using git bare for dotfiles is in installing the dotfiles on a new machine. Just drop the `--bare` flag while cloning. 

```bash 
git clone --bare https://github.com/<username>/dotfiles.git $HOME/.dotfiles && source ~/.zshrc 
```

#### Version History 

If breaking things is a ritual. Git promotes experimenting with new configurations, eliminating the pain of finding the way back to the last working version. Basically, all the pros of using git.

```bash 
# To check the version history 
dotfiles log 

# To checkout to last working thing
dotfiles reset <commit-hash> --hard/soft
```

#### Access across multiple devices

Share the same configs to multiple devices with minimal changes using **git branches**. Create a branch for your new machine.

```bash 
# Create configurations specific to your AWS machines 
dotfiles checkout -b aws && source ~/.zshrc   
``` 

#### Environment enabled configurations Profiles

Allows the separation of configurations by creating profiles as git branches based on your environment. Create a branch, and update the configurations according to the env. 

```bash 
# Branches as profiles for - Activate  the work profile 
dotfiles checkout work && source ~/.zshrc
``` 

The list doesn't end here. These are just some examples that I have used in day-to-day activities. One can develop more practical usage of tracking dotfiles with git. 

This article's recap is that elegant solutions don't have to be fancy. It's good to use something familiar for better use of time. Always prefer less/no dependency on other software. Most importantly, try not to get overwhelmed by the things on the internet - pick the solutions that satisfy the needs. 


Feel free to reach out to me at [rec.manish.sahani@gmail.com](mailto:rec.manish.sahani@gmail.com)