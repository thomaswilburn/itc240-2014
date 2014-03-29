Getting Started
===============

For this class, you'll need to turn in source code for each of your homework assignments. To do so, we'll be committing code from Zephir to GitHub, so that you don't need to copy and paste. The following steps will connect your Zephir directory to your GitHub account, and teach you the commands required to update them.

First things first, you'll need a [GitHub account](https://github.com) if you don't already have one. An account is free for public repos, which is fine for our purposes. Once you've got that, log into Zephir using PuTTY or another SSH terminal. The username and password [should be similar](http://www.seattlecentral.edu/dmartin/moin/Zephir#How_Accounts_are_Created) to the ones used in the labs.

Once you're logged into Zephir, you should be able to generate the SSH key to connect to GitHub, as in [this article](https://help.github.com/articles/generating-ssh-keys). The SSH key is an encrypted form of ID used to authenticate instead of a password. Type the following command to begin and press enter:

```sh
ssh-keygen -t rsa -C "your-email-here@example.com"
```

The generation program will ask a series of questions, such as the passphrase to protect your key. The defaults are fine, so just press enter until it returns you to the prompt. To display your key, type the following:

```sh
cat ~/.ssh/id_rsa.pub
```

If the process works, you should see a block of text starting with `ssh-rsa` and ending with your e-mail address. That block is the encryption key. If you select it with the mouse, it'll be copied to your computer's clipboard.

Now that we have a key, you just need to tell Github about it. Use your browser to visit [the SSH section](https://github.com/settings/ssh) of your account settings on GitHub, and press the "Add SSH Key" button. In the form, type "Zephir" in the title box, and paste the key into the second box. Press the green "Add key" button, and you should see a success message.

Finally, let's test to see if it worked. In the Zephir terminal window, use the following command.

```sh
ssh -T git@github.com
```

If it worked, you should see a message that reads "Hi user! You've successfully authenticated, but GitHub does not provide shell access." 

Now, let's clone the class repo so you can get started. In your browser, visit the class source repo [here](https://github.com/thomaswilburn/itc240-2014). If you're logged in, there should be a button in the upper-right of the page marked "Fork." Click that button to make a copy of the class repo in your own account. It may take a few minutes before it shows up, but once it does, it should contain an exact copy of whatever was inside the original, including this file.

Back in your Zephir command line, type the following lines, pressing enter after each:

```sh
cd public_html
git clone git@github.com:your-username-here/itc240-2014.git
```

Using `git clone` will create a second copy of the repo, this one linked to your account. Changes made in that directory can be committed and published up to the site, which is where I'll look at your source code for grading.
