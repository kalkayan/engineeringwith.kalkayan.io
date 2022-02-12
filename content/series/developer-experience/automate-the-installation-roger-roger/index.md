---
title: "Automate the Installation - Roger Roger"
date: 2022-02-12T05:49:13+05:30
draft: true
description: ""
tags: []
categories: ""
---

Setting up a new development machine is a long and painful process; nobody ever remembers what they have installed in their previous device. This guide is here to help you automate everything in your development setup and lets you enjoy the exciting feeling of getting a new machine.

The guide has three parts: first is automating the installation of binaries and applications, then configuring code applications (vim, tmux, etc.) and customizing the operating system with dotfiles, and finally how to store and share them. I'll set up a MacBook Pro in all the articles, but you can always expand it to other surfaces like Linux.

<!-- > If you have any trouble sounding condescending, find a UNIX user to show you how it's done. ~ Scott Adams -->

# Getting Started

When you have a new MacBook, what is the first thing you do? (After posting its picture on LinkedIn): Download necessary apps like Spotify, Slack, VS Code, etc., right?. Visiting tens of websites and downloading dmgs was such a long, painful and tedious process until brew casks came into the picture. This feature of Homebrew gives us an excellent starting point for automating the setup process.

We can have a small bash script and call Homebrew from there. Building on this idea, in this series, we'll write a script such that it can set up your brand new machine with just one command of the following structure.

```bash
curl -fsSL https://example.com/setup.sh | bash
```

Let's create this `setup` script in the $HOME directory, which will automate our future set-up process.

```bash
touch $HOME/setup
```

# Neccessay tools and a Package Manager

We need to install XCode command-line tools, which install all the common and necessary things like macOS SDK, headers, build tools, compilers, interpreters, etc. Add the following in the previously created script:

```bash
#!/usr/bin/env bash

# Install xcode command line tool if not already installed
if ! xcode-select -p 1>/dev/null; then
    xcode-select --install
fi
```

First, the above lines check for xcode-select's path (`-p` is for printing the path of the active developer directory), and if not present, it installs using the `--install` flag.

Then we install a system package manager -- Homebrew, using the similar if-then structure.

```bash
#!/usr/bin/env bash

# Install xcode command line tool if ....

# Install brew.sh (system package manager) if already not installed
if ! brew -h 1>/dev/null; then
    echo /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
fi

# Update brew to latest available version
brew update
```

After installing brew, we run `brew update` to ensure we have the latest version installed.

# Installing Applications and libraries

In this section, We deal with installing applications and libraries, and I'll show you one of the coolest ways to manage and automate this process with just a few bash lines.

It is hard to remember what applications and libraries we had previously installed. Homebrew has `homebrew-bundle` to take care of this by dumping all the installed taps, bins, and casks in a `Brewfile` which can install all these in the future. The problem here is that `Brewfile` doesn't store versions of the bins or casks. It installs the latest version of the binaries, which can sometimes cause you trouble. For example, if you have PHP installed -- or in general software that doesn't have backward compatibility.

To solve this problem of versioning, let us start from the most straightforward thing and create a list of binaries and casks that we need (name it `bins.txt` and `casks.txt`).

```bash
touch $HOME/.config/kalkayan/bins.txt $HOME/.config/kalkayan/casks.txt
```

We list down all the brew formulas for binaries and casks with their versions that we want to have in our system in their specific file. for example --

```
hades in ~ $ cat $HOME/.config/kalkayan/bins.txt
coreutils
llvm
go
php@7.4
nvm
...
```

To automate these bins and casks' installation, we need to import these lists in our `setup` script. `mapfile` creates an array from the file, and now we have our list store in collections named `bins` and `casks`.

```bash
#!/usr/bin/env bash

# Install xcode command line tool if....

# Install brew.sh (system package manager)....

# List of binaries that needs be installed
mapfile -t bins <  <(cat $HOME/.config/kalkayan/bins.txt)

# List of casks that needs be installed
mapfile -t casks < <(cat $HOME/.config/kalkayan/casks.txt)
```

Now we iterate over these arrays and check for the library if already installed. If not, we install it using brew. In the following code, first, we split the name at `@` and take the first (example -- `php` in `php@7.4` )

```bash
# List of casks ....

# Install binaries using Homebrew, iterate over bins array and install.
for binary in "${bins[@]}"; do
    # split the brew formula into binary name and version of installation and
    # get the name of the binary (ex- php in php@7.4)
    b=`echo $binary | cut -d \@ -f 1`
    # check if the binary is already present, otherwise install
    if ! brew list --formula | grep "$b" 1>/dev/null; then
        brew install "$binary"
    fi
done
```

Similarly, we write a loop for casks but without worrying about the versions (can be extended if required)

```bash
# Install binaries using Homebrew, ....

# Install casks using Homebrew, iterate over casks array and install.
for c in "${casks[@]}"; do
    # first, check if the cask is present or not, if not install
    if ! brew list --cask | grep "$c" 1>/dev/null; then
        brew casks install "$c"
    fi
done
```

In the second part of this series, we'll create a bash function to update our bins.txt and casks.txt when a bin/cask is installed or removed.

