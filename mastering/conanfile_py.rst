Use conanfile.py for consumers
===============================

You can use a ``conanfile.py`` for installing/consuming packages, even if you are not creating a package with it. You can also use the existing ``conanfile.py`` in a given package while developing it to install dependencies, no need to have a separate ``conanfile.txt``.

Let's take a look at the complete ``conanfile.txt`` from the previous *timer* example with POCO library, in which we have added a couple of extra generators

.. code-block:: text
   
      [requires]
      Poco/1.7.3@lasote/stable
      
      [generators]
      gcc
      cmake
      txt
      
      [options]
      Poco:shared=True
      OpenSSL:shared=True
      
      [imports]
      bin, *.dll -> ./bin # Copies all dll files from the package "bin" folder to my project "bin" folder
      lib, *.dylib* -> ./bin # Copies all dylib files from the package "lib" folder to my project "bin" folder


The equivalent ``conanfile.py`` file is:

.. code-block:: python

   from conans import ConanFile, CMake
   
   class PocoTimerConan(ConanFile):
      settings = "os", "compiler", "build_type", "arch"
      requires = "Poco/1.7.3@lasote/stable" # comma separated list of requirements
      generators = "cmake", "gcc", "txt"
      default_options = "Poco:shared=True", "OpenSSL:shared=True"
            
      def imports(self):
         self.copy("*.dll", dst="bin", src="bin") # From bin to bin
         self.copy("*.dylib*", dst="bin", src="lib") # From lib to bin


Note that this ``conanfile.py`` doesn't have a name, version, or ``build()`` or ``package()`` method, as it is not creating a package, they are not required.

With this ``conanfile.py`` you can just work as usual, nothing changes from the user perspective.
You can install the requirements with (from mytimer/build folder):

.. code-block:: bash

   $ conan install ..


conan build
------------

One advantage of using ``conanfile.py`` is that the project build can be further simplified,
using the conanfile.py ``build()`` method.


If you are building your project with CMake, edit your ``conanfile.py`` and add the following ``build()`` method:

.. code-block:: python
   :emphasize-lines: 13, 14, 15, 16

   from conans import ConanFile, CMake

   class PocoTimerConan(ConanFile):
      settings = "os", "compiler", "build_type", "arch"
      requires = "Poco/1.7.3@lasote/stable"
      generators = "cmake", "gcc", "txt"
      default_options = "Poco:shared=True", "OpenSSL:shared=True"

      def imports(self):
         self.copy("*.dll", dst="bin", src="bin") # From bin to bin
         self.copy("*.dylib*", dst="bin", src="lib") # From lib to bin

      def build(self):
         cmake = CMake(self.settings)
         cmake.configure(self)
         cmake.build(self)

   
Then execute, from your project root:

.. code-block:: bash

   $ mkdir build && cd build
   $ conan install ..
   $ conan build ..
   

The ``conan install`` command downloads and prepares the requirements of your project
(for the specified settings) and the ``conan build`` command uses all that information
to invoke your ``build()`` method to build your project, which in turn calls ``cmake``.

This ``conan build`` will use the settings used in the ``conan install`` which have been cached in the local ``conaninfo.txt`` file in your build folder, which simplifies
the process and reduces the errors of mismatches between the installed packages and the current
project configuration.


If you want to build your project for **x86_64** or another setting just change the parameters passed to ``conan install``:

.. code-block:: bash

   $ rm -rf *  # to clean the current build folder
   $ conan install .. -s arch=x86_64
   $ conan build ..

Implementing and using the conanfile.py ``build()`` method ensures that we always use the same
settings both in the installation of requirements and the build of the project, and simplifies
calling the build system.


Other local commands
----------------------

Conan implements other commands that can be executed locally over a consumer ``conanfile.py`` which is in user space, not in the local cache:

- ``conan source <path>``: Execute locally the conanfile.py ``source()`` method
- ``conan package <path>``: Execute locally the conanfile.py ``package()`` method

These commands are mostly used for testing and debugging while developing a new package, before ``export-ing`` such package recipe into the local cache.


.. seealso:: Check the section :ref:`Reference/Commands<commands>` to find out more.