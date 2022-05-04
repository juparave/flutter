# Ruby tips

You don't have write permissions for the /Library/Ruby/Gems/2.3.0 directory. (mac user)

    $ brew install chruby ruby-install
  
ref: https://stackoverflow.com/questions/51126403/you-dont-have-write-permissions-for-the-library-ruby-gems-2-3-0-directory-ma

Update:

You are correct that macOS won't let you change anything with the Ruby version that comes installed with your Mac. However, it's possible to install gems like bundler using a separate version of Ruby that doesn't interfere with the one provided by Apple.

Using sudo to install gems, or changing permissions of system files and directories is strongly discouraged, even if you know what you are doing. Can we please stop providing this bad advice? I wrote a detailed article that shows why you should never use sudo to install gems.

The solution involves two main steps:

* Install a separate version of Ruby that does not interfere with the one that came with your Mac.
* Update your PATH such that the location of the new Ruby version is first in the PATH. Some tools do this automatically for you. If you're not familiar with the PATH and how it works, it's one of the basics that you should learn, and you'll understand why you sometimes get "command not found" errors and how to fix them.

There are several ways to install Ruby on a Mac. The best way that I recommend, and that I wish was more prevalent in the various installation instructions out there, is to use an automated script like Ruby on Mac that will set up a proper Ruby environment for you.

The main reason is that it saves each person a ton of time. Time is our most limited and valuable resource. Why make people do things manually when they can be automated with a perfect result every time?

Another reason is that it drastically reduces the chance of human error, or errors due to incomplete instructions.

If you want to do things manually, keep on reading. First, you will want to install Homebrew, which installs the prerequisite command line tools, and makes it easy to install other necessary tools.

Then, the two easiest ways to install a separate version of Ruby are:
If you would like the flexibility of easily switching between many Ruby versions [RECOMMENDED]

Choose one of these four options:

    chruby and ruby-install - my personal recommendations and the ones that are automatically installed by the Ruby on Mac script. These can be installed with Homebrew:

brew install chruby ruby-install

    rbenv - can be installed with Homebrew

    RVM

    asdf

If you chose chruby and ruby-install, you can then install the latest Ruby like this:

ruby-install ruby

Once you've installed everything and configured your .zshrc or .bash_profile according to the instructions from the tools above, quit and restart Terminal, then switch to the version of Ruby that you want. In the case of chruby, it would be something like this:

chruby 3.1.0

Whether you need to configure .zshrc or .bash_profile depends on which shell you're using.
