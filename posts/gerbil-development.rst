:Author: Luis Osa
:Date: Fri Dec  4 18:24:35 2020
:tags: scheme, emacs

=====================================================================
Setting up a development environment for Gerbil Scheme and Doom Emacs
=====================================================================


There is an `installation guide <https://cons.io/guide/>`_ and a `page <https://cons.io/guide/emacs.html>`_ about development with Emacs in the
Gerbil Scheme website, but I felt it was centered on plain Emacs and not on
variants like Doom Emacs -- which is fine, of course, I just happen to prefer Doom
because it has a focus on Vim keybindings and I am much more used to them than
to plain Emacs chords.

Install Gerbil
--------------

This one is easy: use the Hombrew formula. It also pulls in the underlying
Gambit Scheme dependency.

.. code:: console

    $ brew install gerbil-scheme

Install ``gerbil-mode`` and ``gambit-mode``
-------------------------------------------

Both Gerbil and Gambit offer Emacs support as single Elisp files that live
inside their respective project directory tree. They intend these files to be
copied or symlinked into ``~/.emacs.d`` and required in ``.emacs``.

I found it easier to install them as “Github recipes” with Doom’s ``package!``
macro.

Within ``.doom.d/packages.el``, add:

.. code:: elisp

    (package! gerbil-mode
      :recipe (:host github :repo "vyzo/gerbil"
               :files ("etc/gerbil-mode.el")))

    (package! gambit-mode
      :recipe (:host github :repo "gambit/gambit"
               :files ("misc/gambit.el")))

Configure the new modes
-----------------------

There is a full example of a ``use-package`` configuration in the page linked
above from Gerbil’s documentation. I have adapted it to configure separately
both modes, only taking the minimal required setups.

Within ``.doom.d/config.el``, add:

.. code:: elisp

    (use-package gambit-mode
      :defer t
      :config
      (add-hook 'inferior-scheme-mode-hook 'gambit-inferior-mode))

    (use-package gerbil-mode
      :defer t
      :config
      (defvar gerbil-program-name
        ;; depending on your env setup, this can probably be just "gxi"
        (expand-file-name "/usr/local/bin/gxi"))
      (setq scheme-program-name gerbil-program-name))

Redefine key bindings
---------------------

Most of the useful key bindings that Gerbil-mode and Gambit-mode expose are
behind a “C-c” prefix. In Doom, hitting that key chord with a Schme file open
(file extension “.scm” or “.ss”) will show all available key bindings on a
``which-key`` minibuffer. I have redfined some of them to be also available under
the local leader key, which is defined elsewhere in my config to ``,``.

Within ``.doom.d/config.el``, add:

.. code:: elisp

    (after! gerbil-mode
      (map! :map gerbil-mode-map
            :localleader :desc "Run Scheme shell"
            "'" (cmd! (pop-to-buffer "*scheme*")
                      (run-scheme scheme-program-name))
            "," #'scheme-send-region
            "e" #'scheme-send-definition
            "E" #'scheme-send-definition-and-go
            "l" #'scheme-load-file))
