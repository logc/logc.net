:Author: Luis Osa
:Date: Tue Nov 24 16:35:41 2020
:tags: scala, sbt, build

====================
Creating SBT plugins
====================


Summary
-------

How to create a minimal working SBT plugin. There is an example repository on
Github `here <https://github.com/logc/sbt-hello>`_.

Create a new Scala project
--------------------------

.. code:: console

    $ sbt new scala/scala-seed.g8

Pick a name that starts with ``sbt-``, following the `plugin author best practices <https://www.scala-sbt.org/1.x/docs/Plugins-Best-Practices.html#Artifact+naming+convention>`_.

Modify ``build.sbt``
--------------------

.. caution::

    Remove the ``scalaVersion`` from ``build.sbt``. In `this issue <https://github.com/sbt/sbt/issues/5032>`_ of SBT the authors
    explain that there is no version of SBT for Scala 2.13. However, since that is
    the current stable version of Scala, the ``scala-seed`` project template selects
    it by default, leading to a compilation error.

The only other change to ``build.sbt`` is to add a line ``enablePlugins`` to the
main project. Enabling the ``SbtPlugin`` is what marks this project as an SBT
plugin at all.

.. code:: scala

    lazy val root = (project in file("."))
      .enablePlugins(SbtPlugin)
      .settings(
        name := "sbt-hello"
      )

Extend ``AutoPlugin``
---------------------

I am not sure whether there is another interface to create SBT plugins but this
one is convenient: create an object that extends ``AutoPlugin``.

Inside that object, the most important things that you should define are a
``Task`` for your plugin, and whatever ``Settings`` it takes to get configured.
There is some discussion in the documentation about the differences between a
Task and a Command, but the quick explanation is that the Task is whatever
command the user of your plugin is going to execute on the SBT shell, while a
Setting is something to be customized in the user’s ``build.sbt``.

There are **functions** called ``taskKey[T]`` and ``settingKey[T]`` that allow you to
produce the **types** ``TaskKey`` and ``SettingKey`` (note the different
capitalization).

A simple example taken from the reference documentation is this plugin:

.. code:: scala

    object SbtHelloPlugin extends AutoPlugin {
      object autoImport {
        val greeting = settingKey[String]("greeting")
        val hello = taskKey[Unit]("say hello")
      }
      import autoImport._
      override def trigger = allRequirements
      override lazy val buildSettings =
        Seq(greeting := "Hi!", hello := helloTask.value)
      lazy val helloTask =
        Def.task {
          println(greeting.value)
        }
    }

where ``greeting`` is a value that can be overridden by users in their
``build.sbt``, and ``hello`` is the command name they can use in the SBT shell to
perform what the plugin has to offer. The return type of that task is ``Unit``
because it just prints a string to the console, and the type of the setting is a
``String`` because that is what gets printed.

Usually, you would create another object, in the same package or even in the
same file, that implements an ``apply`` method that is really doing the work you
intend to do. That is what you would have to call in the ``Def.task`` part of the
Plugin object.

.. code:: scala

    object Hello {
      def apply(): Unit = println("Hi!")
    }

.. code:: scala

    object SbtHelloPlugin {
      // ...
      lazy val helloTask =
        Def.task {
          Hello()
        }
    }
