Documentation Process
=====================
This section describes the process for cloning, editing, and commiting to these GitHub pages, i.e., these web pages.

Cloning Plato Docs
------------------
Firstly, navigate to a directory where the user can clone Plato Docs.

.. code-block:: console

   git clone https://github.com/platoengine/platodocs.git

   mkdir platodocs-build

   cd platodocs-build

   git clone https://github.com/platoengine/platodocs.git html

   cd html

   git checkout -b gh-pages remotes/origin/gh-pages

The directory structure should look like this:

.. code-block:: guess

   topfolder\
   |-- platodocs\
   |   |-- docs\
   |   |   |-- source\
   |   |   |-- make.bat
   |   |   |-- Makefile
   |   |-- platodocs.pdf
   |   `-- README.md
   |
   `-- platodocs-build\
       |-- doctrees\
       `-- html\
           |-- _images\
           |-- _source\
           |-- _images\
           |-- index.html
           |-- .nojekyll
           `-- other files

