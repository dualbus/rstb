rstb
====

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

I want to have a blog. But I don't like:

- To use a crappy HTML embedded editor. I prefer to use my beloved vim.
- To do raw HTML in vim. I want to write in a simple and concise
  format, and let another tool do the job of producing the HTML.
- That my blog depends completely of an external provider. If XYZ provider
  dies, my posts die with it. I guess I could backup, in the case of SQL
  databases, but ¿really?. I prefer to have my blog locally, and just push the
  posts to the external provider.
- To copy-n-paste the produced content to the rich-text editor of the XYZ
  provider. I prefer to mail the post to it, or push it using another tool.

And thus, rstb was born. From "reStructuredText Blog" (I'm pretty sure the rstb
acronym isn't new, since I read it somewhere, but it doesn't seem to be in use,
so I took it, it's cool).

Things you can do with rstb:

- Use you favorite text editor to write your posts.
- Write in an intermediate format, suitable for humans (like reStructuredText),
  and produce HTML from it.
- Manage a blog in your local computer. Your blog is in your computer, and you
  push the changes to your publisher.
- Publish your blog. If you do want to let the world know about your ramblings,
  you can extend rstb to make it publish your documents to external hosts, via
  mail or any other mean.

Configuration
+++++++++++++

Configuration files:

- /etc/rstbrc
- /usr/local/etc/rstbrc
- ~/.rstbrc

The configuration file will be know from now on as rstbrc. These files are
sourced as bash scripts, so be careful. Any code you put in there will be run
as normal code.

Some tips:

- No whitespace around the ``=`` sign in variable assignments. Sorry, bash
  doesn't accept it. So, ``a=b`` is right, ``a = b`` is wrong.

- Quoting rules are more complex than in most programming languages. Again,
  sorry, but blame Bourne. If you're having trouble quoting:

  * http://mywiki.wooledge.org/Quotes
  * http://wiki.bash-hackers.org/syntax/quoting

- Funcions are defined as ``<name>(){ <list-of-commands>; }``. You can read
  more about them:

  * http://mywiki.wooledge.org/BashGuide/CompoundCommands#Functions

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
    Compilation format extension. From "Built extension". Yeah, I'm not good
    with the names. The default value is ``bext=html``. It controls the
    extension used for the compiled post.

New
+++

To create a new post, just type:

    rstb new <name>

where <name> is the name of the post, for example:

    rstb new what-a-wonderful-world

The previous command will create a post in ``posts_directory``. If you don't
commit the changes you do in your editor, the entry will not be saved. See the
section on Edit_ for more details on the editing. 

The following directory structure will be used:

.. code:: bash

    $posts_directory/$year/$month/$day/$index/what-a-wonderful-world.$sext

Where ``$sext`` is the expanded value of the intermediate format extension. If
you're using something different to reStructuredText for your documents, you
should modify it to match that format.

Edit
++++

You might want to edit an already created post, so that's just:

    rstb edit <file>

where <file> is the whole path to the file created.

I know, typing the whole path is boring. rstb is supposed to help, not bother.
Well, I did a bash completion script to ease the typing:

- https://github.com/dualbus/bashcomp/

You can create a bash function in the rstbrc file to override the editor. The
file will be the first argument to the function. For example, this one will
open the file in ``gedit``:

.. code:: bash

   # We don't want gedit to mess with our terminal.
   editor() { gedit "$1"    </dev/null >&0 2>&1; }

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

<file> is the path to the file in the intermediate format. Again, bash
completion is suggested to reduce the amount of tedious typing.

You can override the ``builder`` function to provide a different compiler. For
example, instead of the default:

.. code:: bash

   builder(){ rst2html "$1"; }

you could provide

.. code:: bash

   builder(){ rst2pdf "$1"; }

Or whatever tool you desire. You can even handle Markdown or other intermediate
formats.

There is one variable to control the extension of the built file, ``bext``
(from build extension). You can set ``bext=pdf``, for example, to use it with
the ``rst2pdf`` builder.

View
++++

.. code:: bash

   viewer() { firefox "$1"  </dev/null >&0 2>&1; }

Publish
+++++++

PROBLEMS
--------

- Alpha. 

SEE ALSO
--------

- http://docutils.sourceforge.net/ 
- https://github.com/dualbus/bashcomp/ 
