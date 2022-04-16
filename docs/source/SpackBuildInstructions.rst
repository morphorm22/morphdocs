Build Instructions
==================
This section goes over the steps to build `PLATO Engine <https://github.com/platoengine/>`_ with `PLATO Analyze <https://github.com/platoengine/platoanalyze/>`_ using the `Spack <https://spack.io/>`_ package management tool. The instructions have been tested on Ubuntu 18.04 and make use of the apt package manager that ships with Debian derived GNU/Linux distributions.

Spack Setup
-----------
Before beginning the Plato Engine build process, we need to set up Spack. First, install these dependencies if you do not already have them:

.. code-block:: console

   $ sudo apt install build-essential gfortran git curl python
   
Next, clone the Spack repository specifically tailored to install Plato Engine and checkout the :code:`update` branch:

.. code-block:: console

   $ git clone https://github.com/platoengine/spack.git
   $ cd spack
   $ git checkout update
   
To use Spack, you first need to set up your environment. Fortunately, Spack provides a script to do this for you. Source this script in your current shell.

.. code-block:: console

   $ source ~/spack/share/spack/setup-env.sh
   
**Hint:** If you are working with multiple spack areas or different spack branches, sometimes spack can find itself in a bad state. We have found the command :code:`spack clean -a` to be very helpful in such situations. If you get spacked, try running this command.

Plato Engine Build Instructions
-------------------------------
.. Note::

   If you only wish to build Plato Engine (without Plato Analyze), proceed with these instructions, otherwise, skip to :ref:`Plato Analyze Build Instructions <PlatoAnalyzeBuildInstructions>`.

Plato Engine can be installed with the following command:

.. code-block:: console

   $ spack install platoengine

Running this command will install the default configuration of platoengine along with all of its dependencies. :numref:`PlatoengineVariants` shows the spack 'variants' for building platoengine.

.. _PlatoengineVariants:

.. csv-table:: 'Variants' for building platoengine
   :header: "Variant", "Default Value", "Description"
   :widths: 5, 5, 15
   :align: center

   "platomain", "True", "Compile PlatoMain"
   "platostatics", "True", "Compile PlatoStatics"
   "regression", "True", "Add regression tests"
   "unit_testing", "True", "Add unit testing"
   "albany_tests", "False", "Configure Albany tests"
   "analyze_tests", "False", "Configure Analyze tests"
   "cuda", "False", "Configure with cuda"
   "esp", "False", "Configure with esp"
   "expy", "False", "Compile exodus/python API"
   "geometry", "False", "Turn on Plato Geometry"
   "iso", "False", "Turn on iso extraction"
   "platoproxy", "False", "Compile PlatoProxy"
   "prune", "False", "Compile turn on use of prune and refine"
   "rol", "False", "Turn on use of rol"
   "stk", "False", "Turn on use of stk"
   "tpetra_tests", "False", "Configure Tpetra tests"

These can be turned on or off by appending the variant to the spack spec using the :code:`+` or :code:`~` operators. For example, platoengine can be built with the 'rol' module enabled but without support for unit testing by using

.. code-block:: console

   $ spack install platoengine+rol~unit_testing

For more details on forming specs, see the `Spack Documentation <https://spack.readthedocs.io/en/latest/>`_


.. _PlatoAnalyzeBuildInstructions:

Plato Analyze Build Instructions
--------------------------------
To build and run Plato Analyze, you must have a machine with an nVidia GPU with a minimum compute capability of 3.0. To determine the compute capability of your GPU, go to the `nVidia website <https://developer.nvidia.com/cuda-gpus>`_. If you have a GPU that meets the minimum criteria, then you will need to install CUDA version 9.2 or greater.

.. Note::

   Please ensure that the drivers installed on your system as well as the CUDA version that you are running support the compute capability of your GPU. For more information please see `CUDA compatibility <https://docs.nvidia.com/deploy/cuda-compatibility/index.html>`_.

If both Spack and CUDA are properly installed, you can now begin the build process by installing Plato Analyze and all of it's dependencies with a single command. In the following command set the :code:`$COMPUTE_CAPABILITY` flag to the compute capability of your GPU (with no decimal).

.. code-block:: console
   
   $ spack install platoanalyze+cuda ^trilinos cuda_arch=$COMPUTE_CAPABILITY ^amgx cuda_arch=$COMPUTE_CAPABILITY

For instance, if you have a Tesla V100 GPU, the compute capability is 7.0. Thus, the :code:`$COMPUTE_CAPABILITY` flag should be set as follows:

.. code-block:: console

   $ spack install platoanalyze+cuda ^trilinos cuda_arch=70 ^amgx cuda_arch=70

**This step can take several hours** since it will install many Plato Engine, Plato Analyze, and all of its dependencies in the spack/opt directory.

:numref:`PlatoAnalyzeVariants` lists the variants of Plato Analyze

.. _PlatoAnalyzeVariants:

.. csv-table:: 'Variants' for Plato Analyze
   :header: "Variant", "Default Value", "Description"
   :widths: 5, 5, 15
   :align: center

   "amgx", "True", "Compile with AMGX solver"
   "cuda", "True", "Compile with cuda"
   "meshmap", "True", "Compile with MeshMap capability"
   "mpmd", "True", "Compile with mpmd"
   "esp", "False", "Compile with ESP"
   "geometry", "False", "Compile with MLS geometry"
   "openmp", "False", "Compile with OpenMP for CPU solvers"
   "python", "False", "Compile with python"
   "rocket", "False", "Builds ROCKET and ROCKET_MPMD"
   "tpetra", "False", "Compile with Tpetra solvers"

Building for Different Platforms
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The following command will build Plato Analyze for CPU with the Tpetra stack in Trilinos and OpenMP enabled on the CPU.

.. code-block:: console
 
   $ spack install platoanalyze~cuda~amgx+openmp+tpetra

Alternatively you could disable OpenMP with :code:`~openmp`, or build with Epetra with :code:`~tpetra`. We recommend Tpetra with OpenMP for best performance on CPU builds.

It is also possible to use Tpetra solvers on the GPU. This can be done with:

.. code-block:: console

   $ spack install platoanalyze+cuda~amgx~openmp+tpetra ^trilinos cuda_arch=70

Development Builds
------------------
The Plato team relies on the git submodule feature to reduce large file content from the main source code repository. In order to create a development build of Plato Analyze with regression tests enabled, you need to run the following set of commands from inside you Plato Analyze source directory before creating a development build.

.. code-block:: console

   $ git submodule init
   $ git submodule update

In order to create a development build of platoengine or platoanalyze, rather than using :code:`spack install`, simply run :code:`spack dev-build` from your source directory. For example, to create a development build of platoanalyze, you can run.

.. code-block:: console

   $ git clone https://github.com/platoengine/platoanalyze.git
   $ cd platoanalyze
   $ spack dev-build platoanalyze+cuda ^trilinos cuda_arch=$COMPUTE_CAPABILITY ^amgx cuda_arch=$COMPUTE_CAPABILITY

Spack Modules
-------------
The next **optional** steps install the dependencies needed to enable the Spack module functionality if your system does not already have environment modules installed.

.. code-block:: console

   spack bootstrap
   source ./spack/share/spack/setup-env.sh
   
Testing Plato Engine
--------------------
To ensure that everything built correctly, from your build directory run the unit tester:

.. code-block:: console

   $ ./apps/services/unittest/PlatoMainUnitTester

Finally we want to test an example run of the Plato Engine. Before doing so, make sure to load the MPI implementation and version of cmake installed by Spack:

.. code-block:: console

   $ spack load cmake
   $ spack load platoanalyze
   $ spack load platoengine
   $ spack load openmpi

Then change to the 2Load_OC directory and run ctest:

.. code-block:: console

   $ cd examples/2Load_OC
   $ ctest -VV

This should go through a full example. At the end you should see "100% tests passed, 0 tests failed out of 1".

Testing Plato Analyze
---------------------
If you ran

.. code-block:: console

   $ git submodule init
   $ git submodule update

before running the :code:`spack dev-build` command to build platoanalyze, then there should be a :code:`tests` directory in your build directory. You can run all of the regression and verification tests with the following command.

.. code-block:: console

   $ cd tests
   $ ctest -VV

However, if you want to run a specific test, you just need to go inside the specific test directory you want to run and use :code:`ctest -VV` command to run the test. For instance:

.. code-block:: console

   $ cd tests/regression
   $ cd AD_Elastic
   $ ctest -VV

This should successfully run the AD_Elastic test.
