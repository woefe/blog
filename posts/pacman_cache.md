---
title: Don't forget to clean your pacman cache!
date: 2017-03-26
---

Arch Linux' package manager **pacman** stores all downloaded packages in `/var/cache/pacman/pkg`.
This cache does not get cleaned automatically and pacman keeps adding new archives with every single system update.
If you forget about the pacman cache for a couple of months it takes up more and more space.
I recently freed over 12GB on my SSD by running the cleanup commands listed below.

<!--more-->

# Cleanup
To see how much space the cleanup command would save you can use `paccache -dk2 && paccache -duk0`.
And to actually clean the pacman cache use following command:

```shell
sudo paccache -vrk2 && sudo paccache -vruk0
```
The first part of the command keeps the two latest versions of each package and removes the rest.
The second part removes all versions of packages which are not installed anymore.
Note that removing all packages from the cache is usually a bad idea, because in case an updated package has a bug you can still install an older version from the cache which does not suffer from the bug.

# Pacaur
Pacaur, currently my favourite AUR helper and pacman wrapper, maintains its own cache for AUR packages in
`~/.cache/pacaur/`. This cache directory grows rapidly if you install huge packages that update often, e.g. the
Jetbrains IDEs. The following command cleans the pacaur cache

```shell
pacaur -Sac
```

# Zsh alias
I have realized that cleaning the pacman and pacaur caches should be a (semi-)regular system maintenance task,
especially on a 120GB SSD, where disk space is very limited. That's why I added the `pacclean` alias to my zshrc:

```shell
alias pacclean='sudo paccache -vrk2 && sudo paccache -vruk0 && pacaur -Sac --noconfirm'
```

# Update (2018)
I switched to pikaur, since pacaur has been deprecated for a while now.
Pikaur maintains its own cache, which can be managed with the `paccache` command.
Meanwhile I use a [small script](https://github.com/woefe/dotfiles/blob/master/scripts/.bin/pacclean) to clean the cache and remove unused packages.
