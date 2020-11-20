:Author: Luis Osa
:Date: Fri Nov 20 22:11:51 2020
:tags: meta

=========================
How is this blog deployed
=========================


Summary
-------

This blog is deployed to Github Pages from `this Github repository <https://github.com/logc/logc.net>`_, and then
served under a sub-domain (a ``www`` sub-domain) of a domain that I host at `Amazon Route 53 <https://aws.amazon.com/route53/>`_.

Step by step
------------

Prepare Github repo
~~~~~~~~~~~~~~~~~~~

Create a new repository on Github. It does not need to be ``<user>.github.io``.

Push the contents of your recently created Sphinx blog. Minor note: later it
is assumed that you have created a Python virtual environment, or otherwise
can list all dependencies needed in a ``requirements.txt`` file at the root of
this repository.

Go to repository settings and activate Github Pages. Leave for now the branch
to be served as ``main``.

Set ``custom domain`` to ``www.yourdomain.yourtld``

By doing that, a CNAME file will be created on the main branch. Pull it to
your local clone of the repository.

Deployment via Github Action
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create a directory ``.github/workflows`` in the root folder of your ``main``
branch.

.. code:: shell

    mkdir -p .github/workflows/

Add the following contents to a file called ``deploy.yml`` (or any other name,
as long as it is a YAML file and it is under that folder):

.. code:: yaml

    name: deploy-website

    # Only run this when the main branch changes
    on:
    push:
        branches:
        - main

    # This job installs dependencies, build the book, and pushes it to `gh-pages`
    jobs:
    deploy-book:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v2

        # Install dependencies
        - name: Set up Python 3.8
        uses: actions/setup-python@v2
        with:
            python-version: 3.8.6

        - name: Install dependencies
        run: |
            pip install -r requirements.txt
        # Build the book
        - name: Build the site
        run: |
            ablog build
        # Push the book's HTML to github-pages
        - name: GitHub Pages action
        uses: peaceiris/actions-gh-pages@v3.5.9
        with:
            github_token: ${{ secrets.GITHUB_TOKEN }}
            publish_dir: ./_website
            cname: www.yourdomain.yourtld

I think the contents are pretty self-explanatory, but for completeness sake:
what this workflow does is to check out the repository every time that there
is a push on ``main``, on a VM running the latest Ubuntu version, and then
proceeds to install Python 3.8 and all dependencies listed on the
repository’s ``requirements.txt`` (which should include ABlog and Sphinx).
Then it builds the blog using the command ``ablog build``, which is the same
command used locally to build everything, and then copies the contents of
the results directory, ``_website``, as if they were the contents of a **new branch** called ``gh-pages`` back to the same Github repository. It also
includes a file called CNAME, apart from the results of building with ABlog,
that contains the custom domain we want to answer from.

After running this workflow for the first time (any push on ``main`` will be
enough, e.g. a change in the README) then a new branch ``gh-pages`` should be
available.

Change the Github repo settings to serve ``gh-pages`` and ``/root``. Keep the same
custom domain.

You may now remove the CNAME file from the ``main`` branch. It is not needed.

Changes in DNS
~~~~~~~~~~~~~~

Now you need to head over to your DNS provider and create a CNAME record on
their side that maps ``www.yourdomain.yourtld`` to ``<user>.github.io``. This is
weird but that is what the official Github documentation says. The specific
repository under your user account is chosen based on the CNAME file that is
served on their side.

The changes in DNS may take from one hour to one day to propagate. In my
experience, waiting for several minutes was enough.

Once the changes are propagated, you will be able to open a browser and type ``http://www.yourdomain.yourtld`` to see the contents of your new blog.

Add HTTPS
~~~~~~~~~

In order to add TLS (note that the previous address uses ``http``, not
``https``!), you need to navigate again to the Github repository’s settings
and click on the setting “Enforce HTTPS” under Github Pages. Github will
take care of certificates and renewing them.

And that is all! Enjoy your new blog.
