Coding Style
================

This section contains all coding style guidelines related to the Morphorm project. They are organized by Variables, Classes, Functions, Files and Other Conventions.

Special emphasis is given to the cardinal rules:

* **Each statement shall always be in a separate line however small it may appear.**
   
.. code-block:: c++

   int tOne = 0; int tTwo = 1;  // BAD!
   
   int tOne = 0;                // GOOD!
   int tTwo = 1;                // GOOD!
   
An exception to the above rule is reserved for matrix data structures, e.g.,

.. code-block:: c++

   std::vector<std::vector<double>> tMatrix = { {C11, C22, C33},
                                                {C21, C22, C23},
                                                {C31, C32, C33} }; // GOOD!, PREFERRED

   std::vector<std::vector<double>> tMatrix = 
       { {C11, C22, C33}, 
         {C21, C22, C23}, 
         {C31, C32, C33} }; // GOOD!

   std::vector<std::vector<double>> tMatrix = { {C11, C22, C33}, 
   {C21, C22, C23}, 
   {C31, C32, C33} }; // BAD!
   
* **All Morphorm source code shall always be inside the appropriate Morphorm module (Pareto, Analyze, Optimize, Sample, Model) namespace. Any source code associated with a Morphorm module is considered Morphorm source code.**

.. code-block:: c++

   namespace Pareto
   {
       class MyParetoClass
       {
           // source code
       };

       inline void my_pareto_inline_function()
       {
           // source code
       }
   }
   
File Names
----------
File names shall always begin with the Morphorm Module name first, e.g. Pareto\_, follow by an `upper camel case <https://en.wikipedia.org/wiki/Camel_case>`_ name. For instance, the following file name :code:`Pareto_MyFileName.hpp` is acceptable.

Variables
---------
The following guidelines apply to variables.

Naming Variables
^^^^^^^^^^^^^^^^
Use clear and concise names for variables. If a variable's name is to have multiple words, it should become increasingly accurate from right to left (i.e. Adjectives always go left). For example, :code:`aUserId` should be used over :code:`aIdUser`. Both are obviously some type of identification, thus "**user**" is the adjective that describes the "**Id**". When variables have names with multiple words, use `lower camel case <https://en.wikipedia.org/wiki/Camel_case>`_ to distinguish each word, e.g. :code:`aUserId`. Use the proper prefix for a variable (see :ref:`Prefixes <prefixes>`). Finally, common words should be abbreviated (see :ref:`Abbreviations <abbreviations>`).

.. _prefixes:

Prefixes
^^^^^^^^
When naming variables, use the following prefixes depending on the type of variable:

* :code:`Member variable`: m - (e.g., mVals, mNumRows)
* :code:`Argument variable`: a - (e.g., aVals, aInputParameter)
* :code:`Temporary variable`: t - (e.g., tVector)

.. _abbreviations:

Abbreviations
^^^^^^^^^^^^^
The following abbreviations are approved in Morphorm source code:

* :code:`Glb` - Global (e.g. GlbIDs)
* :code:`Loc` - Local (e.g. LocIDs)
* :code:`Id` - 1-based ids
* :code:`Ind` - 0-based index
* :code:`num` or :code:`Num` - number (i.e. numNodes, tNumNodes)


Declaring Variables
^^^^^^^^^^^^^^^^^^^
When declaring variables, use the following guidelines:

* Declare and initialize one variable at a time.

.. code-block:: c++

   int aAnInteger, anotherInteger // BAD!

   int aAnInteger                 // BAD!
   int aAnotherInteger

   int    aAnInteger  = 1;        // GOOD!
   double aRealNumber = 5.0;

   int aAnInteger = 1;            // GOOD!
   double aRealNumber = 5.0;

* The characters :code:`*` and :code:`&` should be written together with the types of variables instead of with the names of variables in order to emphasize that they are part of the type definition.

.. code-block:: c++

   int *anInteger    // BAD!
   int* anIntPointer // GOOD!

Classes
-------
The following guidelines apply to classes.

Class Name
^^^^^^^^^^
Class names should always use `upper camel case <https://en.wikipedia.org/wiki/Camel_case>`_, e.g. :code:`MyClass`. Further, if a class name has more than one word, use upper camel case and **Do Not** separate the names with underscores.

.. code-block:: c++

   My_Class_Name  // BAD!
   MyClassName    // GOOD!

Functions
---------
The following guidelines apply to functions.

.. Note::

   Constructors (and destructors) are the exception to this rule since C++ requires all constructor and destructors to have the same name as the class name.

Function Names
^^^^^^^^^^^^^^
Class function names should have `lower camel case`_ names. If a function name includes multiple words, do not separate the names with underscores.

.. code-block:: c++

   my_class_function_name()   // BAD!
   myClassFunctionName()      // GOOD!

**Non-member function names are an exception to this rule.** A non-member function is a function that is not define inside a class, e.g. inline function. Non-member functions should have lower case names, e.g.

.. code-block:: c++

   Function()   // BAD!
   function()   // GOOD!

**If the non-member function name has multiple words**, separate each name with an underscore as follows

.. code-block:: c++

   MyFreeFunctionName()      // BAD!
   myFreeFunctionName()      // BAD!
   myfreefunctionname()      // BAD!
   my_Free_Function_Name()   // BAD!
   My_Free_Function_Name()   // BAD!

   my_free_function_name()   // GOOD!

Function Declaration
^^^^^^^^^^^^^^^^^^^^
This guidelines will use the word 'tab' when referring to guidelines regarding indentation. Note that 1 'tab' is 4 regular spaces. This preference is part of the group's `Eclipse development environment <https://www.eclipse.org/>`_ preferences (see Importing for details on importing preferences). When declaring a function, use the following guidelines:

* When using :code:`auto`, the :code:`->` operator followed by :code:`declrtype` should align with the function name.
* When declaring functions, the leading parenthesis is written on the same line as the function name with no spaces between them.
* Similarly, the trailing parenthesis is written on the same line as the last argument, if any. If the function has no arguments, then the trailing parenthesis is written in the same line as the function name.
* Each argument is written on a separate line, in the following order: :code:`argument type`, :code:`qualifiers` (if any, e.g. :code:`const` or :code:`&` ), :code:`argument name`, :code:`default values` (if any).
* Additionally, if a function has more than one :code:`argument`, the :code:`types` are left-aligned, then the :code:`qualifiers` are left-aligned,and so on.
* Both leading and trailing braces, i.e. :code:`{}`, are written in their own lines and align with the function name.
* Function names are indented with one :code:`tab`, i.e. four regular spaces.
* The body of a function is indented with :code:`two tabs`, i.e. eight regular spaces.

The following function declaration adheres to these guidelines:

.. code-block:: c++

   // GOOD!
   template<typename ScalarType>
   void axpy(const ScalarType& aAlpha, 
             const Pareto::Vector<ScalarType>& aInput,
             Pareto::Vector< ScalarType >& aOutput)
   {
       const auto tLength = aInput.size();
       for(decltype(tIndex) = 0; tIndex < tLength; tIndex++)
       {
            aOutput[tIndex] += aAlpha * aInput[tIndex];
       }
   }

   // GOOD!
   template<typename ScalarType>
   void axpy
   (const ScalarType& aAlpha, 
    const Pareto::Vector<ScalarType>& aInput,
    Pareto::Vector<ScalarType>& aOutput)
   {
       const auto tLength = aInput.size();
       for(decltype(tIndex) = 0; tIndex < tLength; tIndex++)
       {
            aOutput[tIndex] += aAlpha * aInput[tIndex];
       }
   }

   // GOOD!
   template<typename ScalarType>
   void axpy
   (const ScalarType                  & aAlpha, 
    const Pareto::Vector<ScalarType>   & aInput,
    Pareto::Vector<ScalarType>         & aOutput)
   {
       const auto tLength = aInput.size();
       for(decltype(tIndex) = 0; tIndex < tLength; tIndex++)
       {
            aOutput[tIndex] += aAlpha * aInput[tIndex];
       }
   }

   // BAD - HAVING MULTIPLE ARGUMENTS IN ONE LINE!
   template<typename ScalarType>
   void axpy(const ScalarType & aAlpha, const Pareto::Vector<ScalarType> & aInput,
             Pareto::Vector<ScalarType>       & aOutput)
   {
       const auto tLength = aInput.size();
       for(decltype(tIndex) = 0; tIndex < tLength; tIndex++)
       {
            aOutput[tIndex] += aAlpha * aInput[tIndex];
       }
   }

   // BAD - NON-ALIGNED ARGUMENTS!
   template<typename ScalarType>
   void axpy(const ScalarType & aAlpha, 
      const Pareto::Vector<ScalarType> & aInput,
         Pareto::Vector<ScalarType>       & aOutput)
   {
       const auto tLength = aInput.size();
       for(decltype(tIndex) = 0; tIndex < tLength; tIndex++)
       {
            aOutput[tIndex] += aAlpha * aInput[tIndex];
       }
   }

Other Conventions
-----------------

Operator Spacing
^^^^^^^^^^^^^^^^
`C++ operators <https://www.geeksforgeeks.org/operators-c-c/>`_ should be spaced as follows:

* Always use spaces before and after the following operators: =, +, -, \*, and all logical operators.
* Do not use spaces around ‘.’ or ‘->', nor between unary operators and operands.

.. code-block:: c++

   i++    // this is GOOD!
   i ++   // this is BAD!
 
   AFullArray.AMemberFunction     // this is GOOD!
   AFullArray . AMemberFunction   // this is BAD!
 
   this->MemberFunction     // GOOD!
   this -> MemberFunction   // BAD!

Switch Statements
^^^^^^^^^^^^^^^^^
A switch statement must always contain a default branch use to handle unexpected cases.

.. code-block:: c++

   switch (VariableName)
   {
       case ACASE:
       {
           // lines of code
           break;
       }
       case ANOTHERCASE:
       {
           // lines of code
           break;
       }
       default: // this is necessary
       {
           // this is the default case
       }
   } // end of switch structure











