Committing Your First Files
===========================

Files located in the `public_html` folder of your Zephir home directory can be viewed via a browser, but we can't see their source code. Using the following commands, we can synchronize the contents of your `itc240-2014` folder that you created earlier to GitHub.

Using whatever method you prefer, such as editing through WinSCP or uploading from your local machine, create a new file called `index.php` inside of the `assignment-0` folder, with the text "This space intentionally left blank" inside. Then log into Zephir using an SSH program like PuTTY. The following commands will change your location to the repository folder.

```sh
cd public_html
cd itc240-2014
```

The `cd` command moves you down a directory. If you ever want to go up, use the `cd ..` command to move up one folder level. You can see your current location by typing `pwd` (for "present working directory"). 

Once you're inside a Git repository, such as the `itc240-2014` folder that we cloned from GitHub, you can use the `git` command to perform various repo operations. For example, let's take a look at the current state of the repo. Type the following command to see:

```
git status
```

You should see a listing onscreen, with your new `index.php` file under "untracked files." That means that Git can see your file, but isn't actively storing backups of its contents. We want to add it to our list of tracked files. Use the `add` command to put it in the "staging area":

```sh
git add assignment-0/index.php
```

Running `git status` now will show the new file under "Changes to be committed." Storing changes in Git is a two-stage process: first we stage the changes, using `git add`, then we commit them as a backup snapshot. Type the following command to make your first commit.

```sh
git commit -m "First commit!"
```

The text after the `-m` flag is the text of the commit message, so you can keep track of which changes were made with each snapshot. You can see the history of all commits made in a repo by using `git log`.

Once you have a commit of your own, let's push it up to GitHub. Send the commit states up to GitHub by using the `push` command.

```sh
git push origin master
```

For this command, `origin` is the destination for the push (in this case, where it was cloned from), and `master` is the branch (we will typically only use the default, "master" branch). When the push completes, you should be able to visit your repo on the GitHub website and see the files there. Other people will also be able to pull from your repos &mdash; which is how I'll see your homework for grading. It's important not to forget to push your changes to GitHub when you finish homework, since it will be late if the source code is not committed.

Git is an extremely useful service, but it's not perfect. If your repo becomes stuck, or you find yourself unable to commit files, please feel free to send me a message, and we'll get it sorted out.
