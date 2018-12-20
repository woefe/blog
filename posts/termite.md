---
title: Install Termite's terminfo
date: 2017-11-04
---

On many systems Termite's terminfo is not available by default. That's a problem since you cannot start ncurses apps,
when the terminfo is missing. I usually run into this problem when I try to ssh into a newly installed machine. Here is
a one-liner to fix the issue (you have to run this command on the remote, not you local machine!).

<!--more-->

```bash
curl https://raw.githubusercontent.com/thestinger/termite/master/termite.terminfo | sudo tic -x -
```

The command above adds a terminfo entry to the system-wide database. If you don't have root access on the remote
machine, simply remove the `sudo` from the command, which will make the terminfo available only to the current user.
