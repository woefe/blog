---
title: Bookmarks with ownCloud and Vimperator
date: 2016-05-23
---

I recently added the Bookmarks app to my ownCloud installation. This app keeps a simple list of bookmarks synced across
my computers. The Bookmarks app provides a [bookmarklet](https://en.wikipedia.org/wiki/Bookmarklet) to add new bookmarks
easily. But there is a problem with using the bookmarklet: I have to click on it! And therefore I have to display the
bookmarks bar or search for the bookmarklet in the bookmarks menu. I don't like both approaches for following reasons:
The bookmark bar eats too much space on my small notebook screen and searching for the bookmarklet in the menu takes too
long.

<!--more-->

The solution to this problem involves the Firefox plugin "Vimperator", another great piece of software. Vimperator
rearranges or removes some UI components of Firefox, like the address bar and makes your Browser behave like the
[Vim](http://www.vim.org) text editor. In Vimperator one can execute JavaScript code with the `:js` command. This makes
it possible to execute the JavaScript of the bookmarklet directly in Vimperator. So let's define a function that adds
a new bookmark to the Bookmarks app and map this function to a keypress in Vimperator.

This snippet of my `.vimperatorrc` creates a new key binding (**A** in normal mode), which overrides Vimperator's
default key binding for adding a new bookmark.

    :::javascript
    noremap A :js bmark_owncloud()<CR>

    :js << EOF
    function bmark_owncloud(){
        var bmark_title = encodeURIComponent(content.document.title);
        var bmark_url = encodeURIComponent(content.document.location);
        var window_features = 'left=' + ((window.screenX||window.screen.left) + 10) + ',top=' + ((window.screenY||window.screen.top) + 10) + ',height=500px,width=550px,resizable=1,alwaysRaised=1';
        var window_name = 'bkmk_popup';
        var owncloud_url = 'https://owncloud.example.com/index.php/apps/bookmarks/bookmarklet?output=popup&url=' + bmark_url + '&title=' + bmark_title;
        var new_window = window.open(owncloud_url, window_name, window_features);

        window.setTimeout(
            function(){
                new_window.focus()
            },
            300
        );
    };
    EOF

Now the only thing that is left to do, is to change the `https://owncloud.example.com` to your actual ownCloud
installation and you are good to go!
