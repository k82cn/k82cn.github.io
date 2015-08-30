---
layout: post
categories: tech
title: Thinking in shared_ptr/auto_ptr
---

The father of C++ said that he never meet MLK issue after using auto pointer. It’s a good practice to use auto pointer to replace malloc/free. There are also some practice for auto pointer.

## auto pointer in C++

Compared to C, constructor/deconstructor is an important improvement of C++. It’s not only make sure that the object/struct will be automatically initialized, but also keep C++ developer far away from MLK issue. The smart C++ developer used de-constructor to release memeory automatically when the reference holder destroy (out of scope, or delete). Here is an simple example to show how auto pointer work:

    class SimpleAutoPointer
    {
        public:
            SimpleAutoPointer(void* ptr)
            {
                m_rawPtr = ptr
            }
    
            ~SimpleAutoPointer()
            {
                delete m_rawPtr;
            }
    
        private:
            void* m_rawPtr;
    };
    
    int main(int argc, char** argv)
    {
        {
            SimpleAutoPointer ptr(new int);
        } // the memory will be free when SimpleAutoPointer exit the scope
    }

## std::shared_ptr VS std::auto_ptr

The above sections of the code has lots of drawbacks:

1. Can not delete the pointer that pointing to an array
2. Assignment is invaild; if not reference counter, double free error will destroy your application
3. Template is better than void*
4. Can not access the raw pointer
5. Can not execute raw pointer’s function
6. ……

It’s really a hard work to build a strong auto pointer class; but STL has help us on completing this work. There are two kind of auto pointer: std::tr1::shared_ptr & std::tr1::auto_ptr.

* __std::tr1::shared_ptr__ : std::shared_ptr is a smart pointer that retains shared ownership of an object through a pointer. Several shared_ptr objects may own the same object; the object is destroyed when the last remaining shared_ptr pointing to it is destroyed or reset. The object is destroyed using delete-expression or a custom deleter that is supplied to shared_ptr during construction. A shared_ptr may also own no objects, in which case it is called empty.
* __std::tr1::auto_ptr__: auto_ptr objects have the peculiarity of taking ownership of the pointers assigned to them: An auto_ptr object that has ownership over one element is in charge of destroying the element it points to and to deallocate the memory allocated to it when itself is destroyed. The destructor does this by calling operator delete automatically. Therefore, no two auto_ptr objects should own the same element, since both would try to destruct them at some point. When an assignment operation takes place between two auto_ptr objects, ownership is transferred, which means that the object losing ownership is set to no longer point to the element (it is set to the null pointer).

## auto pointer in practice

Thanks to STL’s great work, C++ developer has a strong auto pointer class to use; but there is still a trap which will introduce MLK issue, its name is “Circular References”. The “Circular References” will happen when the two objects point to each other by auto pointer. For now, there is no technical to resolve “Circular Reference” issue; but by following this coding practice, we can avoid this kind of issue: clarify the responsiblity of releasing the object; if the object did not have the responsiblity of releasing the object, it should hold the raw pointer; otherwise, it should hold the auto pointer. There must be reply that: how about the two object take the responsibility of each other? There are two options:

* Option 1: change the design to break the dependency
* Option 2: create the third class as the owner/manager of those two classes

## Summary

Enjoy your journey in C++ world with auto pointer!!!
