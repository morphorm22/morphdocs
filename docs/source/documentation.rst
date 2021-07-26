Documentation For Plato
=======================
This section describes the process for cloning, editing, and commiting to these GitHub pages, i.e., these web pages. These pages are created using reStructuredText in `Sphinx <https://www.sphinx-doc.org/>`_.

Cloning Plato Docs
------------------
The user should navigate to a directory where :code:`/platodocs/` and :code:`/platodocs-build/` can be stored.

.. code-block:: console

   git clone https://github.com/platoengine/platodocs.git

   mkdir platodocs-build

   cd platodocs-build

   git clone https://github.com/platoengine/platodocs.git html

   cd html

   git checkout -b gh-pages remotes/origin/gh-pages

The directory structure should look something like this:

.. code-block:: guess

   topfolder\
   |-- platodocs\              <-- release branch
   |   |-- docs\
   |   |   |-- source\
   |   |   |   |-- images\
   |   |   |   |-- _static\ 
   |   |   |   |-- _templates\
   |   |   |   |-- conf.py     <-- configuration information to sphinx (extensions, etc.)
   |   |   |   |-- index.rst   <-- index source file
   |   |   |   `-- other files
   |   |   |-- make.bat
   |   |   `-- Makefile
   |   |-- platodocs.pdf
   |   `-- README.md
   |
   `-- platodocs-build\
       |-- doctrees\
       `-- html\           <-- gh-pages branch
           |-- _images\
           |-- _source\
           |-- _images\
           |-- index.html  <-- main html file (can open with browser)
           |-- .nojekyll   <-- tells github to not use a jekyll theme
           |-- README.md
           `-- other files

The existing webpages can be viewed by opening :code:`/platodocs-build/html/index.html` in a web browser or they can be viewed in the pdf version from :code:`/platodocs/platodocs.pdf`.

Editing Plato Docs
---------------------
**Editing Existing Pages:** 

Documentation can be edited from the source folder (:code:`/platodocs/docs/source/`). The files in this directory are written using reStructuredText (.rst). These files can be opened in any text editor and edited using syntax found `here <https://docutils.sourceforge.io/rst.html>`_.

**Adding New Pages:** 

When adding a new page to the html web pages, the user can create another .rst file in the source directory (:code:`/platodocs/docs/source/`). This file is added to the website by adding the file name to the contents in :code:`/platodocs/docs/source/index.rst` or another file if the user does not want the file to show up on the home page.

For example, this page is built based on the :code:`documentation.rst` and was added to the web pages by using "toctree" directive in :code:`index.rst`. An example of this is shown below.

:code:`/platodocs/docs/source/index.rst`

.. code-block:: console

   .. toctree::
   :maxdepth: 2

      description
      githubrepositories
      documentation        <-- adding documentation.rst

**Updating HTML and PDF in Ubuntu:** 

The web pages (:code:`.html` files) can be updated to represent the changes in the :code:`.rst` files.

First, the user should navigate to :code:`/platodocs/docs/`. Then they can run the following command in a virtual environment (.venv) from the terminal. For directions to get into a virtual environment, go to ( **TODO:** add reference to virtual environment).

.. code-block:: console

   (.venv) make html

Changes can be viewed by opening :code:`/platodocs-build/html/index.html` in a web browser. 

The user can update the pdf version of the documentation by running the following command at from :code:`/platodocs/docs/` from the terminal.

.. code-block:: console

   (.venv) make latexpdf

The changes can be viewed by opening :code:`/platodocs/platodocs.pdf`.

Publishing Documentation
------------------------


Pushing and Pulling
-------------------
The process for pushing to and pulling from the :code:`release` branch of Plato Docs (:code:`/platodocs/`) is standard. This is the branch used for the source files of Plato Docs.

.. code-block:: console

   git push

or

.. code-block:: console

   git pull

The user will have to specify the branch when pushing to or pulling from the :code:`gh-pages` branch of Plato Docs (:code:`/platodocs-build/`). This is the branch used for the build files of Plato Docs.

Navigate to be within the :code:`/platodocs-build/html/` directory.

.. code-block:: console

   git push origin gh-pages

or 

.. code-block:: console

   git pull origin gh-pages

Packages for Editing Documentation
----------------------------------

