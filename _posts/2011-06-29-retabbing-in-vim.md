---
layout: post
title: Retabbing files using VIM
---

# Retabbing files using VIM #

Recently I've made a mistake of using 2 spaces to indent my F# program. After
some thought I decided to stick to 4 spaces like everyone else does.

Here is how it looked:

    module foomodule
      let foo = 
        let bar = 
          get_file_names()
          |> Seq.filter (f -> f.EndsWith(".cs")
        let bar2 = do_something_with_bar bar
        bar2

And here is how I wanted it to look

    module foomodule
        let foo = 
            let bar = 
                get_file_names()
                |> Seq.filter (f -> f.EndsWith(".cs")
            let bar2 = do_something_with_bar bar
            bar2


What to do? This is not as trivial as it may look. But not if use use VIM!

Here is how to retab everything using VIM:

Step 0. (Optional) Tell VIM to show whitespace:

    :set list

Step 1. Tell VIM what the the current tabsize is:

    :set tabstop=2      " Tab size is 2 spaces

Step 2. Convert spaces to real tabs:

    :set noexpandtab    " Use real tab instead of space
    :retab!             " Replace all space-tabs with real tabs

Step 3. Convert real tabs back to spaces, using tabsize 4

    :set expandtab      " User spaces instead ot tab
    :set tabstop=4      " 4 spaces for each tab
    :retab              " Reformat using new tabbing policy


It's possible to get all this on one line:

    :set ts=2 noet | retab! | set et ts=4 | retab




