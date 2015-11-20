---
layout: post
title: C++&#58; Namespace code style in Google
categories: tech
---

## Named namespaces in Google Code Style

* Namespaces wrap the entire source file after includes, gflags definitions/declarations and forward declarations of classes from other namespaces.

    // In the .h file
    namespace mynamespace {

      // All declarations are within the namespace scope.
      // Notice the lack of indentation.
      class MyClass {
      public:
        ...
        void Foo();
      };

    }  // namespace mynamespace

    // In the .cc file
    namespace mynamespace {

      // Definition of functions is within scope of the namespace.
      void MyClass::Foo() {
        ...
      }

    }  // namespace mynamespace

* More complex .cc files might have additional details, like flags or using-declarations.

    #include "a.h"

    DEFINE_bool(someflag, false, "dummy flag");

    using ::foo::bar;

    namespace a {

      ...code for a...         // Code goes against the left margin.

     }  // namespace a

* Do not declare anything in namespace std, including forward declarations of standard library classes. Declaring entities in namespace std is undefined behavior, i.e., not portable. To declare entities from the standard library, include the appropriate header file.

* You may not use a using-directive to make all names from a namespace available.

    // Forbidden -- This pollutes the namespace.
    using namespace foo;

* You may use a using-declaration anywhere in a .cc file (including in the global namespace), and in functions, methods, classes, or within internal namespaces in .h files.

* Do not use using-declarations in .h files except in explicitly marked internal-only namespaces, because anything imported into a namespace in a .h file becomes part of the public API exported by that file.

    // OK in .cc files.
    // Must be in a function, method, internal namespace, or
    // class in .h files.
    using ::foo::bar;

* Namespace aliases are allowed anywhere where a using-declaration is allowed. In particular, namespace aliases should not be used at namespace scope in .h files except in explicitly marked internal-only namespaces.

    // Shorten access to some commonly used names in .cc files.
    namespace baz = ::foo::bar::baz;

    // Shorten access to some commonly used names (in a .h file).
    namespace librarian {
      namespace impl {  // Internal, not part of the API.
        namespace sidetable = ::pipeline_diagnostics::sidetable;
      }  // namespace impl

      inline void my_inline_function() {
        // namespace alias local to a function (or method).
        namespace baz = ::foo::bar::baz;
        ...
      }
    }  // namespace librarian

* Do not use inline namespaces.

## Namespace discussion in Mesos

* [Move "using mesos::fetcher::FetcherInfo" into internal namespace in "fetcher.hpp"](https://issues.apache.org/jira/browse/MESOS-3963)
