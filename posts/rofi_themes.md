---
title: Convert legacy Rofi themes
date: 2017-10-24
---

[Rofi](https://github.com/DaveDavenport/rofi) is a window switcher, application launcher and dmenu replacement, which
integrates nicely with tiling window managers. In versions 1.3.x and before themes were read from [X
resources](https://wiki.archlinux.org/index.php/X_resources). The recent upgrade to version 1.4.2 broke my custom
[Arc-Dark](https://github.com/horst3180/Arc-theme) inspired theme. Here is how to migrate legacy themes to the new
format:

```shell
rofi -config /path/to/old/themefile -dump-theme > theme.rasi
```

<!--more-->

In case you are interested in the Arc-Dark theme I am using, check my
[dotfiles](https://github.com/woefe/dotfiles/blob/master/rofi/.config/rofi/ArcDark.rasi). Alternatively you might be
able to get it from the [Rofi Themes Repo](https://github.com/DaveDavenport/rofi-themes), if they accept my pull
request. The theme was first contributed to the theme repo by [Faruk Ünver](https://github.com/leofa), but got lost when
the themes were restructured. 

It took me quite some time to figure out the command above, because at a first glance it seems like there is a different
method to convert legacy themes. There is a
[script](https://github.com/DaveDavenport/rofi/blob/next/script/rofi-convert-theme.sh) that is supposed to convert
legacy themes to the new format. However, this script generated a theme file, which could not be parsed by Rofi. After
digging around some more in Rofi's project files, I found the `convert_old_theme_test.sh` test script, which contains
the command from above. 
