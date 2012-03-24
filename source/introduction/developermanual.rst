====================================================================
 How to update and maintain collective.developermanual
====================================================================

.. admonition:: Description

        This document explains how collective.developermanual
        package is maintained and how changes are pushed.

.. contents :: :local:

.. highlight:: console

Introduction
==============

This document concerns those who:

* wish to generate a HTML version of Plone Community Developer manual

* need to edit templates or styles of Plone Community Developer manual, or
  otherwise customize Sphinx process

* Need to deal with readthedocs.org integration

* wish to upload Sphinx documentation to a Plone site (deprecated)

collective.developermanual
==========================

collective.developermanual_ is open-for-anyone-to-edit documentation for
Plone developers in Sphinx format, living in 
`Plone's collective Github version control system`_ (anyone can get commit
access, or, even easier, submit pull requests from their own github fork of
the repository).

.. _collective.developermanual: https://github.com/collective/collective.developermanual 
.. _Plone's collective Github version control system: https://github.com/collective

Sphinx_ is a tool that makes it easy to create intelligent and beautiful
documentation, written by Georg Brandl and licensed under the BSD license.

.. _Sphinx: http://sphinx.pocoo.org/

The ``collective.developermanual`` checkout contains 
:doc:`buildout.cfg recipes </tutorials/buildout/index>` to:

* install Sphinx;
* compile the manual to HTML;
* upload manual to a Plone site (using the ``bin/toplone`` script and
  ``transmogrify`` source packages).

Upload does not need any special support from the Plone site to accept
Sphinx documentation. If the `Plone Help Center`_ add-on package is
installed, it will use the *Reference Manual* content types, but in its
absence the pipeline can be configured to use plain *Folders* and *Pages*
instead.

.. _Plone Help Center: http://plone.org/products/plonehelpcenter

Setting up software for manual compilation and upload
=======================================================

First you need to install Git for your operating system to be able to
retrieve the necessary source code::

    sudo apt-get install git-core # Debian-based Linux
         
or::

    sudo port install git-core # Mac, using MacPorts

.. note::

    You must not have Sphinx installed in your Python environment (this will
    be the case if you installed it using ``easy_install``, for example).
    Remove it, as it will clash with the version created by buildout.  Use
    ``virtualenv`` if you need to have Sphinx around for other projects.

Then clone ``collective.developermanual`` from GitHub::

    git clone git://github.com/collective/collective.developermanual.git

Run buildout to install Sphinx and the necessary packages for uploading.
First step: bootstrap::

    python2.6 bootstrap.py
    ./bin/buildout

This will always report an error, but the ``bin/`` folder is created and
populated with the required scripts.  Now you need to checkout all the
source code using the *mr.developer* tool::

    ./bin/develop co ""

Run buildout again::

    ./bin/buildout

readthedocs.org
-----------------

The documentation is automatically synced to 
`readthedocs.org <http://collective-docs.readthedocs.org/>`_
by the readthedocs.org bot.

Pushing changes to GitHub is enough to publish the changes.        

Analytics
---------

http://readthedocs.org pages have the Google Analytics script installed.
Please ask on the #plone.org IRC channel for data access.

Building static HTML with Sphinx
=================================

This creates the ``docs/html`` folder from the source documents in the
``source`` folder, by compiling all the ``collective.developermanual``
pages, using the ``sphinx-build`` command from buildout::

    ./bin/sphinx-build source build

If you want to build everything from the scratch, to see all warnings::

    rm -rf build
    ./bin/sphinx-build                                     

.. What about the Makefile? The above commands could also be e.g. 
   ``make html``. Is the Makefile being deprecated?

Editing CSS styles
---------------------

When ``sphinx-build`` is run it copies stylesheets from *sources* to
*build*.

For live editing of CSS styles you might want to do::

    cp source/_static/plone.css build/_static

Then copy back::

    cp build/_static/plone.css source/_static    

.. note ::

    Firefox does not follow symlinks on file:// protocol, and cannot load
    CSS files from them.

More info

* http://sphinx.pocoo.org/templating.html

* https://bitbucket.org/birkenfeld/sphinx/src/65e4c29a24e4/sphinx/themes/basic

Uploading documentation to a Plone site
------------------------------------------

.. warning :: 

    This part is deprecated; docs are no longer hosted on http://plone.org,
    but at http://collective-docs.readthedocs.org instead.

The ``collective.developer`` manual contains a buildout which defines a
pipeline for 
`collective.transmogrify <http://pypi.python.org/pypi/collective.transmogrifier/>`_
to upload the Sphinx-generated HTML files to a Plone site as
`Plone Help Center <http://plone.org/products/plonehelpcenter>`_
*Reference Manual* content.

``collective.transmogrify`` was originally developed to provide pipelines to
import, manipulate and export content. It allows you to use a plug-in
architecture to provide necessary steps, called *blueprints*, to pass
content in a pipeline from one filter to another.  The idea is very similar
to video and audio codec architectures.

Blueprints can be mix-and-matched using a ``pipeline.cfg`` configuration
file.  ``collective.developermanual`` defines pipelines for crawling the
generated Sphinx HTML files, breaking down the HTML to fields (*title*,
*description*, *body*) and then uploading it to a Plone site using Zope's
XML-RPC API and URL functions exposed by Plone.  As Zope provides the
necessary XML-RPC facilities, no specific code for the uploader is needed on
the server running the Plone site. 

Setting up a Plone site for receiving Sphinx documentation
-----------------------------------------------------------

First install the *Plone Help Center* add-on and create a *Reference Manual* 
item at the remote site.

The documentation can be uploaded to:

* any Plone site 
* with
  `Plone Help Center add-on <http://plone.org/products/plonehelpcenter>`_
  installed.
* Sphinx-specific CSS styles can be installed (optional) for source code
  colorization, warnings, notes and other mark-up in the documentation.
  These are included as ``sphinx.css`` and it is easiest to drop them to
  your ``portal_skins`` layer manually.

Compiling the HTML manual
--------------------------

Use the Sphinx makefile::

    make html

.. Should this be changed? To the following:
    ./bin/sphinx-build source build

Running upload
--------------

Buildout generates the ``bin/toplone`` command-line script.

Example how to upload to a local Plone instance::

    make clean html
    ./bin/toplone bin/toplone --ploneupload:target=http://admin:admin@localhost:8080/Plone/documentation/manual/plone-community-developer-documentation

... or in the case of Plone.org::

    ./bin/toplone --ploneupload:target=http://username:password@manage.plone.org/documentation/manual/plone-community-developer-documentation    

(Substitute your username and password in the URL.)

The upload script:

* is generated when you run buildout (see ``buildout.cfg``);
* reconfigures the transmogrifier pipeline to use the remote site and 
  *HTTP Basic Auth* credentials provided on the command line;
* crawls the Sphinx documentation and extracts the title, description and
  body text of each page (in theory you could crawl any remote web site for
  this. The ``toplone`` script is created to crawl local documentation
  only.);
* create *Plone Help Center* content items at the remote Plone instance
  using Zope's XML-RPC functionality;
* update remote Plone items to reflect documentation changes, using Zope's
  XML-RPC functionality;
* Publish items, and hide unnecessary items from the navigation (e.g. image
  source folders).

.. note ::

    The upload script does not currently purge the existing uploaded
    documentation.  If pages have been renamed or moved, you need to delete
    the documentation on the target site before performing the upload. Just
    go to the *Reference Manual Contents* tab, select all, and hit delete.

Setting up CSS for http://plone.org
-----------------------------------

An example ``sphinx.css`` is provided with ``collective.developermanual``.

* It sets up CSS for default Sphinx styles (notices, warning, other
  admonition).  
* It sets up CSS for syntax highlighting.  
* It resolves some CSS class conflicts between Sphinx and the plone.org
  theme.

``sphinx.css`` assumes that a special Sphinx ``page.html`` template is used.
This template is modified to wrap everything which Sphinx outputs in the
``sphinx-content`` CSS class, so we can nicely separate them from standard
Plone styles.

``page.html`` can be found at ``sources/_templates/page.html``.


