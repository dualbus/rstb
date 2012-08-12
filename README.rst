rstb
====

:author: Eduardo A. Bustamante López (dualbus)


NAME
----

    rstb - manage a plain text blog

SYNOPSYS
--------

::

    rstb new <name>
    rstb edit <file>
    rstb build <file>
    rstb view <file>
    rstb publish <file>
    rstb cmd ...

DESCRIPTION
-----------

In a nutshell
+++++++++++++

I want to have a blog. But I don't like/want:

- To use a "rich-text editor". They're all different, some support some
  features, the interfaces are all different. And if I care about my post, I'll
  end up doing it in a text file locally, since some don't recover your post in
  the remote case that an error occurs.
  
- If I'm not using the rich-text editor, then I'm probably doing it in a text
  editor locally. I don't do HTML directly unless I have to, but it's a
  pain-in-the-ass. 

- Depending entirely on an external provider. If XYZ provider
  dies, my posts die with it. I guess I could *export* the blog, but, really?
  I'm not going to have my blog in SQL+HTML.

- To copy n' paste the post from my local editor to the rich-text editor of the XYZ
  provider. I prefer to mail the post to it, or push it using another tool.

To solve all these issues: rstb. From "reStructuredText Blog" (I'm pretty sure
the rstb name isn't new, since I read it somewhere, but it doesn't seem to be
in use, so I stole^H^H^H^Hused it; it's cool). It's implemented in ``bash``,
a powerful shell, superset of POSIX sh. And no, I'm not doing it in POSIX sh, I
don't have that **much** free time.

The things you can do with rstb:

- Use your favorite text editor to write your posts. Yeah, you can use vim,
  emacs, ed, nano, gedit, geany, ...

- Write in a cool intermediate format. I picked reStructuredText as the default
  because it's flexible, complete, and I like it. It's really human-readable.
  Also, it has tools to produce HTML from it (rst2html). But you could pick
  Markdown or any other format you like. Heck, even LaTeX :).

- Manage your blog locally. Your blog is stored in your machine. That's where
  your **main** copy of the blog resides. If you want to make it publicly
  available, just "publish" it.

- Publish your blog. If you do want to let the world know about your ramblings,
  you can extend rstb to make it publish your documents to external hosts, via
  mail or any other mean. By default it assumes your using a mail2blog
  interface, like Blogger's. Just setup your blogger account to accepts posts
  via email, setup the appropiate ``mail_to`` variable in the configuration
  file, and you're good to go.

Installation
++++++++++++

- See INSTALL_ (INSTALL.rst)

.. _INSTALL: https://github.com/dualbus/rstb/blob/master/INSTALL.rst

Configuration
+++++++++++++

I think advanced users have advanced needs. Instead of wasting my precious
seconds using the newbie-friendly-defaults, I prefer to invest some time
configuring my tools. But not hours. Also, I don't want to lock-up other
possible users by doing an inflexible tool that is married to XYZ editor, or
ABC publisher. Hence, configuration files.

By default, rstb will try to read from these locations, in that specific order.

- ``/etc/rstbrc``
- ``/usr/local/etc/rstbrc``
- ``~/.rstbrc``

Yep. Personal settings override the global settings. I will call the
configuration file(s) from now on as ``rstbrc``. By the way, I'm bundling a
sample ``rstbrc`` with some variations on the defaults. It should be
self-documenting (I hope).

``rstbrc`` is sourced as a bash script. This means:

- It will read variables from it.
- It will read function definitions from it.
- Other cool stuff.

But it also means that:

- Any code will run as the calling user. Make sure you don't put stupid things
  in there, like ``rm``'s. 
  
- Also, don't trust that ``rstbrc`` file you found in the XYZ blog from some
  random dude in the Internet.

- If you're putting sensitive information in there, make sure you, and only you
  can read that file. Unix is a multi-user environment. Don't use permissions
  like 0777 in these files. A simple ``chmod a=,u+r ~/.rstbrc`` should do it.

And regarding the actual content of the configuration script (bash), some tips:

- No whitespace around the ``=`` symbol in variable assignments. Sorry, bash
  doesn't accept it. So, ``a=b`` is right, ``a = b`` is wrong.

- Quoting rules are more complex than in most programming languages. Again,
  sorry, but blame Stephen Bourne. If you're having trouble quoting:

  * http://mywiki.wooledge.org/Quotes
  * http://wiki.bash-hackers.org/syntax/quoting

- Funcions are defined as ``<name>(){ <list-of-commands>; }``. You can read
  more about them:

  * http://mywiki.wooledge.org/BashGuide/CompoundCommands#Functions

- If you have any question regarding bash, you can reach me via IRC. I hang
  around in Freenode. #bash is the channel you should be consulting regarding
  **bash** questions. If you have rstb-specific questions, they will be
  off-topic there. Try to contact me via private messages.

Variables 
~~~~~~~~~

``posts_directory``:
    Root of the blog. This directory will serve as the storage point for all
    the posts. Make sure you have write permissions.
``sext``:
    Intermediate format extension. From "Source extension". The default value
    is ``sext=rst``, but you can change it if you want to use another
    intermediate format.
``bext``:
    Compiled format extension. From "Built extension". Yeah, I'm not good
    with the names. The default value is ``bext=html``. It controls the
    extension used for the compiled post.

New
+++

The first step you will want to do is to create a new blog post. Just type:

    rstb new <name>

where <name> is the name of the post, for example:

    rstb new what-a-wonderful-world

Don't use characters that you wouldn't use in file names. And also remember to
quote properly if you're using spaces or shell meta-characters. The previous
command will create a post in ``posts_directory``. If you don't commit the
changes you do in your editor, the entry will not be saved. See the section on
Edit_ for more details on the editing. 

The following directory structure will be created:

.. code:: bash

    $posts_directory/$year/$month/$day/$index/what-a-wonderful-world.$sext

    # ~/blog/2012/08/12/1/what-a-wonderful-world.rst

Where ``$sext`` is the expanded value of the intermediate format extension. If
you're using something different to reStructuredText for your documents, you
should modify it to match that format.

Edit
++++

You might want to edit an already created post, so that's just:

    rstb edit <file>

where ``<file>`` is the path to the file created.

.. note::

    I know, typing the whole path is boring. rstb is supposed to help, not bother.
    Well, I did a bash completion script to ease the typing:

    1. https://github.com/dualbus/bashcomp/

You can create a bash function in the rstbrc file to override the editor. The
file will be the first argument to the function. For example, this one will
open the file in ``gedit``:

.. code:: bash

   # We don't want gedit to mess with our terminal.
   editor() { gedit "$1" </dev/null >&0 2>&0; }

Or just set the EDITOR environment variable, since rstb will try to use your
default editor (or fall-back to vi). If you're having trouble setting that
variable:

* http://mywiki.wooledge.org/DotFiles

Build
+++++

The building process transforms the intermediate format to the final publishing
format. By default, the intermediate format is reStructuredText and the
publishing format is HTML.

To build an existing post:

    rstb build <file>

``<file>`` is the path to the file in the intermediate format. Again, bash
completion is suggested to reduce the amount of tedious typing.

You can override the ``builder`` function to provide a different compiler. For
example, instead of the default:

.. code:: bash

   builder(){ rst2html "$1"; }

you could provide

.. code:: bash

   builder(){ rst2pdf "$1"; }

There is one variable to control the extension of the built file, ``bext``
(from build extension). You can set ``bext=pdf``, for example, to use it with
the ``rst2pdf`` builder.

View
++++

Now you've edited your post, and built it. How does it look? Does it look Ok?
Well, that's what the view command is about.

    rstb view <file>

Will try to preview ``<file>``. Use the path to the source file, not the built
file; let's try to keep the interface simple. Make sure it's already built too!

For example, to use Chromium as the viewer:

.. code:: bash

   viewer() { firefox "$1" </dev/null >&0 2>&0; }

.. note::

   Arch Linux bundles Chromium as ``chromium`` (extra/chromium). If I recall
   correctly, some bundle it as ``chromium-browser``. Make sure you confirm the
   name of the executable.

By default it calls ``xdg-open`` on the file. But that fails miserably if you
don't have it well setup (like me :( ). And since I'm lazy, I prefer to specify
it in ``rstbrc`` than figure out how to reconfigure XDG.

::

    $ xdg-settings get default-web-browser
    xdg-settings: unknown desktop environment

To be honest, I'm not sure if it's my fault or XDG's. I blame XDG though. It
tries to open Internet Explorer via Wine, or something like that. Bah.

Publish
+++++++

Everything until now has been pure happiness. But now it's time to publish your
note. Things start to get ugly. I use blogger, so I'll talk about it. Blogger
has a mail interface. That's nice. The problem is that it doesn't allow me to
edit already posted notes via email. Bad.

I had to setup blogger to provide me with a magical address:

- http://support.google.com/blogger/bin/answer.py?hl=en&answer=41452

After they provided me with the address, I just set it up as a variable in
``rstbrc``:

.. code:: bash

   mail_to='dualbus.SECRET@blogger.com'

Wordpress seems to have that kind of support too:

- http://codex.wordpress.org/Post_to_your_blog_using_email

There's a method to post to blogger via Google's API, I put a sample command in
the ``rstbrc`` bundled.

After all these boring steps:

    rstb publish <file>

Again, ``<file>`` is the path of the file. And voilá. Your post should now be
public.

PROBLEMS
--------

- Alpha. 
- See TODO_ for desirable features that are not implemented.
- Completion is case-sensitive. Bad.
- ``rstb cmd`` isn't documented.

.. _TODO: https://github.com/dualbus/rstb/blob/master/TODO.rst

SEE ALSO
--------

- rstb bash completion: https://github.com/dualbus/bashcomp/ 

- docutils & reStructuredText

  + http://docutils.sourceforge.net/ 
  + http://docutils.sourceforge.net/rst.html
  + http://docutils.sourceforge.net/docs/ref/rst/introduction.html
  + http://docutils.sourceforge.net/docs/ref/rst/restructuredtext.html

- riv.vim: https://github.com/Rykka/riv.vim

- mailx: http://heirloom.sourceforge.net/mailx.html
