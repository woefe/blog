---
title: How to bootstrap your Zsh config with zplug
date: 2019-07-22
---

Recently, I spent some (probably too much) time tweaking my [Zsh](https://en.wikipedia.org/wiki/Z_shell) configuration.
The results of my efforts are the three Zsh plugins [wbase.zsh](https://github.com/woefe/wbase.zsh), [git-prompt.zsh](https://github.com/woefe/git-prompt.zsh), [vi-mode.zsh](https://github.com/woefe/vi-mode.zsh) that are now available for anyone.
In this post I will explain how you can use the plugin manager [zplug](https://github.com/zplug/zplug) together with my plugins to bootstrap your own config.
One note before we start: for the best experience, check that your terminal supports 256 colors or more.
You can do that with the `tput colors` command.

<!--more-->

# Why not Oh My Zsh or Prezto?
There are several ways to manage a Zsh configuration.
The probably most popular approach is to use a configuration framework like [Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh) or [Prezto](https://github.com/sorin-ionescu/prezto).
A configuration framework is a collection of themes and plugins that are carefully selected by developers to ensure good compatibility between plugins.
And all included themes and plugins are pre-configured with settings that are suitable for a majority of users.

Configuration frameworks are great and I can still recommend [Prezto](https://github.com/sorin-ionescu/prezto) together with the powerlevel10k prompt.
However, the frameworks interfere too much with my own ideas of how to configure my shell, and ultimately I decided that the frameworks are not for me.
[Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh), for example, is missing essential plugins like [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) or [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions).
Of course, you could install plugins manually, but that defeats the purpose of a configuration framework, because you would now fiddle with your config manually.
Prezto has a better selection of plugins, but I don't like its `zstyle` configuration mechanism.
It is cumbersome to discover which `zstlye` options are actually available.

I think plugin managers are a better alternative to configuration frameworks.
They can give you a similar user experience, but provide more flexibility.
Next, we will look at the plugin manager [zplug](https://github.com/zplug/zplug) and use it to build a new Zsh config.

# Installing zplug
[zplug](https://github.com/zplug/zplug) stands out among other plugin managers, because it handles not only plugins, but also binary releases from GitHub and more.
It can also manage plugins from Oh My Zsh or Prezto.
The full documentation of zplug and its installation procedure is available in the [README.md](https://github.com/zplug/zplug) of the zplug repo, but here is a short rundown.

1. Install zplug with following Git command:

    ```bash
    git clone https://github.com/zplug/zplug ~/.zplug
    ```
2. Add following template to your `.zshrc`:

    ```bash
    # Initialize plugins
    source ~/.zplug/init.zsh
    # Register your plugins here. E.g:
    #zplug "woefe/wbase.zsh"
    #zplug "zsh-users/zsh-completions"
    #zplug "zsh-users/zsh-autosuggestions"
    zplug load
    ```
3. Restart your shell and execute `zplug install`. You need to run this command every time you add new plugins to your `.zshrc`.
4. Optionally, execute `touch $ZPLUG_LOADFILE` to reduce the startup time of zplug.

# A complete .zshrc
At this point you should have a working zplug installation and be able to execute the `zplug` command.
Now you probably ask yourself, "How exactly do I build my ultimate `.zshrc` with zplug, and where can I find more plugins?"
The recipe for your ultimate `.zshrc` is below and a list of plugins is available at the [awesome-zsh-plugins](https://github.com/unixorn/awesome-zsh-plugins) repo.
Make a backup of your existing `.zshrc` and then replace it with following content, then restart your shell and execute `zplug install`:

```bash
# Aliases
alias la='ls -lah --color=auto'
alias lh='ls -lh --color=auto'
alias ls='ls --color=auto'
alias l='ls --color=auto'
alias grep='grep --color=auto'

# Use the Emacs-like keybindings
bindkey -e

# Keybindings for substring search plugin. Maps up and down arrows.
bindkey -M main '^[OA' history-substring-search-up
bindkey -M main '^[OB' history-substring-search-down
bindkey -M main '^[[A' history-substring-search-up
bindkey -M main '^[[B' history-substring-search-up

# Keybindings for autosuggestions plugin
bindkey '^ ' autosuggest-accept
bindkey '^f' autosuggest-accept

# Gray color for autosuggestions
ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=247'

# fzf settings. Uses sharkdp/fd for a faster alternative to `find`.
FZF_CTRL_T_COMMAND='fd --type f --hidden --exclude .git --exclude .cache'
FZF_ALT_C_COMMAND='fd --type d --hidden --exclude .git'

# Load plugins
source ~/.zplug/init.zsh
zplug "woefe/wbase.zsh"
zplug "woefe/git-prompt.zsh", use:"{git-prompt.zsh,examples/wprompt.zsh}"
zplug "junegunn/fzf", use:"shell/*.zsh"
zplug "junegunn/fzf-bin", from:gh-r, as:command, rename-to:fzf, use:"*linux*amd64*"
zplug "sharkdp/fd", from:gh-r, as:command, rename-to:fd, use:"*x86_64-unknown-linux-gnu.tar.gz"
zplug "zsh-users/zsh-completions"
zplug "zsh-users/zsh-autosuggestions"
zplug "zsh-users/zsh-syntax-highlighting", defer:2
zplug "zsh-users/zsh-history-substring-search", defer:3
zplug load
```

The configuration above installs following plugins. Browse through their READMEs to learn what they do!

* [woefe/wbase.zsh](https://github.com/woefe/wbase.zsh)
* [woefe/git-prompt.zsh](https://github.com/woefe/git-prompt.zsh)
* [zsh-users/zsh-completions](https://github.com/zsh-users/zsh-completions)
* [zsh-users/zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)
* [zsh-users/zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)
* [zsh-users/zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search)

The most notable feature of the `.zshrc` from above are not the plugins, but the commands that get installed.
It installs the [fzf](https://github.com/junegunn/fzf) and [fd](https://github.com/sharkdp/fd) commands and integrates them in your shell.
The fzf integration makes the `<ctrl+r>` hotkey fancier and installs two new hotkeys.
The `<alt+c>` hotkey starts a fuzzy-find `cd` command, and the `<ctrl+t>` hotkey finds a file and inserts its path into the current command line.
Try, for example, typing `nano <ctrl+t>` in the current command line. A window will pop up. Then search for a file and press enter.

The command integration will only work on 64bit x86 CPUs (all modern AMD and Intel CPUs).
If your CPU has a different architecture, for example ARM on a  Raspberry Pi, you need to adjust the `use:` parameter of all `zplug .. as:command ..` lines.

# Without zplug
Unfortunately, zplug adds noticeable delay to the startup of your shell (roughly 100ms on a modern laptop with SSD).
Nevertheless, zplug and especially its command integration of binaries from GitHub provides enough value to justify it in most cases.
If you are not happy with the performance and think of switching to another plugin manager, don't!
Either use zplug or no plugin manager at all.
I think no other plugin manager provides enough benefits over installing and sourcing plugins manually.
You can see how you would manage your plugins manually (with Git submodules) in my [dotfiles](https://github.com/woefe/dotfiles/blob/master/zsh/.zshrc).
