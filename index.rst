
.. logc.net index file, created by `ablog start` on Sat Nov  7 21:56:14 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Telling rocks what to think
===========================

This is the work website of :ref:`Luis Osa Gómez del Campo <about>`. The
opinions expressed here are my own, not my employer's. And of course, all errors
and inaccuracies are mine. Thanks to anyone who takes an interest in these notes!


Recent blog posts
-----------------

.. postlist:: 5
   :excerpts:


Zettelkasten
------------

- :doc:`Start page </zettelkasten/start>`

.. `toctree` directive, below, contains list of non-post `.rst` files.
   This is how they appear in Navigation sidebar. Note that directive
   also contains `:hidden:` option so that it is not included inside the page.

   Posts are excluded from this directive so that they aren't double listed
   in the sidebar both under Navigation and Recent Posts.

.. toctree::
   :glob:
   :hidden:

   about
   zettelkasten/*

