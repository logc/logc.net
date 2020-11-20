:Author: Luis Osa
:Date: Sun Nov 8 22:04:59 2020
:tags: meta

==========
Blog Setup
==========


Summary
-------

I have just started to rewrite my blog using `Sphinx <https://www.sphinx-doc.org/en/master/>`_ and its extension `Ablog <https://ablog.readthedocs.io/>`_. As
an extra, I have setup a part of the blog as a Zettelkasten -- which is more or
less a glorified wiki --, and also I have configured Emacs to write posts in Org
mode and export them to Restructured Text, which is what Sphinx expects.

Initial setup
-------------

.. code:: shell

    mkdir blog && cd blog
    python -m venv venv && source venv/bin/activate
    pip install ablog
    ablog start

This will create an initial structure that looks like this:

::

    .
    ├── _static
    ├── _templates
    ├── about.rst
    ├── conf.py
    ├── first-post.rst
    └── index.rst

The only customization that I have made is to have all posts in a dedicated
subfolder. You need to specify a pattern for the post files in ``conf.py``.

.. code:: python

    blog_post_pattern = "posts/*.rst"

Now the directory structure can be something like this:

::

    .
    ├── _static
    ├── _templates
    ├── about.rst
    ├── conf.py
    ├── index.rst
    └── posts
        ├── blog-setup.org
        └── blog-setup.rst

Adding pages that are not posts
-------------------------------

I also wanted to have a part of this site that does not really need to be
structured as a blog, i.e. pages that do not need a date, are not ordered
chronologically and are related only via internal links.

The main advantage of using Sphinx for this website is specifically for those
pages, so that they get indexed and are searchable.

The ``ablog start`` command already created an ``index.rst`` file with a section for
“Recent posts”. In order to have more pages, you just need to create a
subfolder, add ``.rst`` files there, and **list at least one of them in the index page** with the ``doc`` directive.

.. code:: rst

    Zettelkasten
    ------------

    - :doc:`Start page </zettelkasten/start>`

With a folder structure like this:

.. code:: shell

    .
    ├── _static
    ├── _templates
    ├── about.rst
    ├── conf.py
    ├── index.rst
    └── zettelkasten
        └── start.rst

Adding more zetteln
~~~~~~~~~~~~~~~~~~~

In order to have a Zettelkasten structure, pages inside the folder have to be able to reference each other and the start page. You can get into a recursion error if you list in a ``toctree`` directive a page that already references the current page in another ``toctree`` directive. Therefore, remember to use the ``doc`` directive in order to create links back to the start.

For instance, with a first post that links to a second post, and both of them linking back to the start page, one would have these contents:

.. code:: shell

    └── zettelkasten
        ├── first-zettel.rst
        ├── second-zettel.rst
        └── start.rst

The start page is:

.. code:: rst

    My own Zettelkasten
    ===================

    Which is just a fancy way of saying "my own wiki".

    .. toctree::
        first-zettel

The first zettel is:

.. code:: rst

    A first zettel
    ==============

    This page is referenced from the Start page of the Zettelkasten.

    .. toctree::
        second-zettel

And the second zettel has this contents:

.. code:: rst

    A second zettel
    ===============

    This second zettel references the start. Try not to reference the first one,
    or you will get into an error of recursion, since that zettel references this
    one.

    - :doc:`start`

Writing posts with Org-mode
---------------------------

This is probably only relevant if you are an Emacs user.

Since the blog post pattern of ABlog (and in general, Sphinx) only picks up
``.rst`` files, I can write ``.org`` files in the same folders and export them to
``.rst`` with `ox-rst <https://github.com/msnoigrs/ox-rst>`_, a package availble in MELPA.

The header of those files has to be something like this:

.. code::

    #+TITLE: Blog Setup
    #+AUTHOR: Luis Osa
    #+DATE: Sun Nov 8 22:04:59 2020
    #+OPTIONS: toc:nil num:nil
    :tags: meta

Notice that the ``:tags:`` line is actually raw RST. If you use a ``#+TAGS`` header
line in Org, it will not be exported to the ``.rst`` file. Also, notice that the
``#+DATE`` is a full timestamp and not an Org datetime; those do not get correctly
picked up by the Sphinx engine.

After exporting the Org document to RST, the ``:Author:``, ``:Date:`` and ``:tags:``
lines are underneath the main title. Since ABlog expects to find them as front
matter, you need to manually move them to the first lines of the file. There
seems to be no way to define front matter with ``ox-rst``. I hope this can be
fixed in later releases.
