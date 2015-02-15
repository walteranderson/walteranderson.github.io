---
layout: post
title: Easy Vim Environment
---

I thought I would share a quick way to get started using vim and take advantage
of the large number of plugins that have been created for it using the runtime
manager [Pathogen](https://github.com/tpope/vim-pathogen). This will allow you to
install plugins and runtime files in their own directories, which is extremely convenient
when you combine it with git submodules.

To start, clone the repository into the folder ~/.vim:

{% highlight bash %}
git clone https://github.com/walteranderson/vim-config.git ~/.vim
{% endhighlight %}

The vimrc file is where you include all of your configuration options. In order for
vim to see it, we need to create a symlink, or symbolic link, to the file from within
the home directory. You can think of a symlink as just another type of file that actually
just references a file elsewhere on your system. Like a placeholder sign that points
the computer to the correct location.

{% highlight bash %}
ln -s ~/.vim/vimrc ~/.vimrc
{% endhighlight %}

[Git submodules](http://git-scm.com/book/en/v2/Git-Tools-Submodules) sound like
exactly what they are: existing git repositories within your git repository. In this
case our project is the vim configuration and the other projects are the
plugins.

To install the existing plugins, we need to first initialize the repository

{% highlight bash %}
git submodule init
{% endhighlight %}

Then run an update:

{% highlight bash %}
git submodule update
{% endhighlight %}

This will go through .gitmodules and install each dependency. To start, this
won't actually do anything. That's because the repository I have set up
does not have any plugins. But in the future when you have some plugins installed
and you decide to switch machines, this command will automatically set everything
up for you. To add a new submodule run the submodule add command passing in the
repository of the plugin. Be sure to clone it into the bundle/ folder and name
it something similar to the name of the plugin.

{% highlight bash %}
git submodule add http://github.com/user/name-of-plugin.git bundle/name-of-plugin
{% endhighlight %}

That's essentially it! There is a directory called "colors" in the repo that contains
a color scheme to start you out. You can continue to copy the color scheme file
and paste it in this directory, or you can install the color scheme the same way
as any other plugin using git submodules. At that point, simply open the vimrc
file and change the line starting with "colorscheme" to whatever you want.

{% highlight bash %}
colorscheme Tomorrow-Night
{% endhighlight %}

One last thought, a great place to find vim plugins is [Vim Awesome](http://vimawesome.com/).
There are all sorts of great filters, and you can even naviagate the site using
vim key commands!
