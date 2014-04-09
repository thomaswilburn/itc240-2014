Command Line Basics
===================

When we log into Zephir over SSH, we're immediately confronted with a wall of text and a prompt. If you've primarily used graphical environments like Windows or OS X in the past, this can be baffling and a little scary. But it doesn't take a lot of commands to perform the essential web development tasks. In this document, we'll review any commands that have been covered in class so far.

Where Am I, and Why Am I in This Handbasket?
--------------------------------------------

Our first task is just to figure out where we are, and move to another folder. We always start out in our home folder on the server. Our prompt should show this as a `~` character instead of spelling out the full path. For example, when I log in, my prompt looks like this:

```sh
[twilburn@zephir ~]$
```

The `twilburn@zephir` tells me which user I am logged in as, and `~` says I'm still in my home folder. Prompts for regular users end in a `$` character, while admins end in a `#` (so you'll probably never see a prompt ending in `#` on Zephir, unless something goes wrong).

To see the complete path of my current location, I can type the `pwd` command, which stands for "present working directory".

```sh
[twilburn@zephir ~]$ pwd
/home/twilburn
```

This result means that on the server's hard drive, I'm located in a folder named after my username, inside another folder named "home." For most students, your home folder will be inside `/home/classes/<username>`. By running `ls`, we can see the contents of this folder.

```sh
[twilburn@zephir ~]$ ls
public_html
```

To change folders, we use the `cd` command, which stands for "change directory." For example, inside your home folder, you should have a `public_html` folder, as the `ls` command showed above. Folders will be colored differently from regular files, and on Zephir they're bolded. I navigate inside the `public_html` folder with the following command:

```sh
[twilburn@zephir ~]$ cd public_html
[twilburn@zephir public_html]$ ls
head.inc  index.php  second.php
```

As you can see, I started in home, or `~`. Then I typed `cd public_html` to say that I want to change into that subfolder. I can see that the directory changed, because my prompt now lists a new location between the brackets. Finally, I used the `ls` command to show the files that are inside my new folder.

If you want to go up in the folder heirarchy, each directory has a special "subdirectory" that's actually a link to its parent folder, named `..` (it also has a link to itself, named `.`). So to move back up into my home folder from its `public_html` subdirectory, I type `cd ..` like so:

```sh
[twilburn@zephir public_html]$ cd ..
[twilburn@zephir ~]$
```

Using Nano to Edit Files
------------------------

To create and edit files on the server, we can use the built-in notepad application, which is called Nano. We won't be using this for the entire class--it's far too simplistic--but it will do for now. To start Nano by itself, we use the `nano` command. Most of the time, we want to create or edit a file. So we'll follow that command with the name of the file we want to edit or create. For example, I might type:

```sh
nano index.html
```

If `index.html` exists, Nano will open that file and show it to us for editing. If it doesn't exist, Nano will create it for us, which is handy. 

Once you've got a file open in Nano, you'll notice that it has a double row of helpful tips at the bottom. There may also be a message above that, letting you know how many lines are in the current file or other temporary status information.

```sh
^G Get Help     ^O WriteOut     ^R Read File    ^Y Prev Page    ^K Cut Text     ^C Cur Pos
^X Exit         ^J Justify      ^W Where Is     ^V Next Page    ^U UnCut Text   ^T To Spell
```

These lines are there to remind you of the key combinations for various actions in Nano--after all, it doesn't exactly have a menu you can use with the mouse. In each listing, the `^` means that you should hold the control key while pressing the specified letter. You may see these listings change from time to time, depending on the context. When you're saving a file, for example, it will show you that `^C` (meaning Ctrl-C) will cancel the save operation.

I'm going to write out a file. I type a little bit into the `index.html` file that Nano created when I ran the `nano index.html` command above, then I want to save it. In Nano, saving a file is called "write out", because it writes the content of the file out to the hard disk. You should see that the keystroke for "WriteOut" is `^O`, so I'll press Ctrl-O. 

Nano will ask what filename I want to use, with the current file name already filled in. Most of the time, I'll just hit enter to save with the same filename. If you're trying to type on the Nano screen, make sure that it isn't asking you a question on the line above the command listings. When in doubt, look for the bright, solid cursor onscreen.

Now that I've created the file, I'll press Ctrl-X (or, from the listings, `^X`) to exit. Like most graphical editors, if there are unmodified changes in the file, Nano will ask if I want to save them. You can press `y` or `n` to choose whether or not you want to save. This means you can also use `^X` to exit and save simultaneously, instead of first pressing `^O` to save and then `^X` to quit.

These are the simplest keystrokes available in Nano, but it can actually perform a lot of different operations. Press the `^G` (Get Help) shortcut to find out all the various built-in functions Nano has available. With a little work, Nano can be used as a passable multi-file editor, including syntax highlighting and powerful search capabilities, although it will never be as good as a real, dedicated code editor.

Moving, Copying, and Removing Files
-----------------------------------

Back out of Nano, let's say that you fat-fingered the original command, and now you've saved your file as `index.htmlf`. We want to rename it to `index.html`. In Unix environments like Linux, a "rename" is the same as a "move": you're taking it from one place, and putting it in another. So to rename our file, we use the `mv` command to "move" it to its new filename:

```sh
mv index.htmlf index.html
```

Much better! What if we want to move it between directories? Say, for example, that we created our file in `~`, and not in `~/public_html` where it will be visible to visitors on the web. We need to move it into the subdirectory. That's easy enough: `mv` is smart enough to relocate the file if the second parameter is the name of a directory:

```sh
mv index.html public_html
```

If you also want to change the name of the file when you move it, you can specify that as well. This command will move our file into public_html, but it will also change its name to `index.php`.

```sh
mv index.html public_html/index.php
```

By now, if you've been thinking about the ways that we move around using `cd`, you may have also realized that you can use the `..` directory to move files up a level as well. If we are inside `~/public_html` and want to move a file back up into the home directory, we can do this:

```sh
mv file.txt ..
```

This moves the file into the special `..` directory, which is a link to the parent of the current directory. Basically, you've moved it up a level.

Copying files works the same way as moving them, except obviously the original file will still exist. Use the `cp` command to copy a file from one place to another. In class, when we duplicated the `index.php` file into a file named `second.php`, we ran this command:

```sh
cp index.php second.php
```

Deleting files on the server is risky, since there is no Trash or Recycle Bin. Once you delete something, it's gone. That's part of why we use Git--having a copy of our files in source control means that bad deletions are always recoverable. So make sure you don't really need that file before you delete it.

If you definitely want to delete something, the `rm` command will "remove" it. To get rid of the `second.php` file, we'll type this:

```sh
rm second.php
```

And voila! It's gone for good. Hope you didn't need anything in there.