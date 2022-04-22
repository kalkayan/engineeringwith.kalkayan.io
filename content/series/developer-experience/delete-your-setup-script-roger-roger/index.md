---
title: "Delete your setup script - Roger Roger"
date: 2022-03-20T05:49:13+05:30
draft: false
description: >
    Learn a no-code way to automate your installation setups. After this, you won't need to write scripts to automate your machine setup.
    
tags: ["tooling", "git", "git-bare"]
categories: "development-experience"

---

You may be new to using a shell and have little experience writing automation scripts, or you are a seasoned veteran with a great script to set up your machines. Either way, this piece will introduce a new way of automating the setup and other tasks. And hopefully, you will have a different perspective on this subject and significantly shorter automation scripts by the end of this article. 

# The tool of the wise 

Automation scripts are concise, highly targeted code written to reduce time-consuming procedures such as building a codebase, performing testing, setting up a development machine, and other repetitive work into a single command. A ton of developers use scripts to set up their development machines. These are so prevalent that there are thousands of similar setup scripts on GitHub only for macOS devices.

This popularity among developers stems from the fact that these scripts save time and effort while also being an excellent approach to assure installation consistency when done repeatedly.

# What seems to be the problem, officer?
<!-- # The Problem -->

You may wonder what problems can there be with scripts that claim to be so efficient? The problems are not with the scripts but are in our way of using them. We are continuously writing and hosting too much duplicative and redundant code. 

When writing your setup script, you will often notice that you directly copy a part of someone's code into your script or paraphrase it. And, If you think about it, all those thousands of scripts on GitHub virtually perform the same tasks, i.e., they install XCode, apps, set defaults, etc. You can say they are similar to an extent.

Writing more code requires more maintenance, and it may be the case that maintaining scripts takes more time than what it was supposed to save. 

# I've missed the plan part of the plan

To solve this problem, we must understand what is there in a typical system setup. Regardless of the operating system, this process is broadly installing the following: language support, building tools, a package manager, libraries, binaries, frameworks, applications, and a few customizations tailored to your needs. The ninety percent of the setup scripts accomplish the same work and only differ at the end, where it performs operations that are well suited for your needs. So, We can agree that we can remove this redundancy. 

Whenever there is repetitive work, we tend to modularize it, which is very common in the real world. And modules are the best things a language can have. In most good languages, modules come out of the box. Even C++ is introducing modules in their v20 release. But, Unfortunately, this is not the practice in the case of scripts. 

This little experiment plans to introduce package/module features to our existing scripting language. We will start with something straightforward; we will consider any single script file as a module, and we want a way to import it. 

# Work the problem people

We will start with something straightforward: we want a way to use a script as a module, and we want a way to import it to other scripts. Consider a module script that you can directly import into your script and start calling the functions. This eliminates all this redundant work we need to do.

I have written an effortless script that enables using import right out of the box. Attach this to the start of your script, and import any script on the internet.

```bash 
[ -f $HOME/.config/sh/core  ] && source $HOME/.config/sh/core \
                              || source /dev/stdin <<< "$(curl https://sh.kalkayan.io/config/sh/core)"
```
This source `core.sh` file (directly from `.config` folder or from internet) in your script. The `core.sh` exposes the import functionality to your script. Now, you can directly import any script from the internet. Example: 

```bash
...
# Import remote scripts
import https://sh.kalkayan.io/config/zsh/utils.sh
```

The below diagram helps understanding what is happending behind the scenes when you import a script.
![Workflow](flow.png)


Finally, you can start using functions and variables from `utils.sh` in your script, and the complete code looks like:
```bash
# Bring the import functionality for this scriot
[ -f $HOME/.config/sh/core  ] && source $HOME/.config/sh/core \
                              || source /dev/stdin <<< "$(curl https://sh.kalkayan.io/config/sh/core)"

# Import other scripts
import https://sh.kalkayan.io/config/zsh/utils.sh

# Rest of your script
install_xcode
```


# This is still too much work, delete the script, delete it, delete it

I got you. If you don't want to write that also, you are welcome to use my setup script. There are two ways to do this: first install everything 
```
curl -fsSL https://sh.kalkayan.io/setup | zsh -s -- --new-machine
```

If you want more control with what you are installing then (Assume you have my kind of dotfiles or you are providing --with-dotfiles)
```bash
curl -fsSL https://sh.kalkayan.io/setup | zsh -s -- --with-dotfiles      \
                                                    --install-buildtools \
                                                    --install-brew       \
                                                    --install-bins       \
                                                    --install-casks
```


<!-- # The Grievances  -->

Thanks for the read. If you have any thoughts on this, or just want to say hi, feel free to reach out to me at rec.manish.sahani@gmail.com or connect on LinkedIn.