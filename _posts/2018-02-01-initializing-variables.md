---
ID: 655
post_title: >
  C++, where initializing variables is the
  topic of debate, by experts.
author: Jan Wilmans
post_excerpt: ""
layout: post
permalink: >
  http://nullptr.nl/2018/02/initializing-variables/
published: true
post_date: 2018-02-01 00:57:08
---
Let me first say: I did not come up with the title myself, but have to credit it to cpplang @ slack.com. Unfortunately I forgot who actually said it.

So what is so interesting about initializing a variable you might ask? Lets take a simple struct:

    struct Foo
    {
        int x;
        int y;
    };
    

And create an object (in a local scope)

    void f()
    {
         Foo foo; 
    }
    

This object will have two uninitialized members x and y. Because the standard says: "only initialize **local** variables when explicitly requested". Actually, that is a lie, the [standard][1] says no such thing. But it **is** basically what it comes down to for local variables.

Global or static variables are always zero initialized at before the program starts. However, I'm having difficulty interpreting what the standard says about local variables. So lets do some testing:

## Full code:

    #include <string>
    #include <iostream>
    
    struct FooBar
    {
        FooBar()
        {
            std::cout << "FooBar init, ";
        }
    };
    
    struct Bar
    {
        int x;
        int y;
        FooBar fb;
    };
    
    struct BarKeep
    {
        BarKeep() : bar(Bar()) {}
        Bar bar;
    };
    
    void dump(std::string msg, Bar& b)
    {
        std::cout << msg << "{ " << b.x << ", " << b.y << " }\n";
    }
    
    int main()
    {
        Bar a = { 1, 2 };
        dump("a = ", a);    // OK
    
        Bar b;
        dump("b = ", b);    // Uninitialized!
    
        auto e = Bar();     // VS2013: Copy of Uninitialized == still Uninitialized
        dump("e = ", e);
    
        Bar f = Bar();      // VS2013: Copy of Uninitialized == still Uninitialized
        dump("f = ", f);
    
        BarKeep h{};        // VS2013: Copy of Uninitialized == still Uninitialized
        dump("h = ", h.bar);
    
        Bar c{};            // OK
        dump("c = ", c);
    
        Bar d = {};         // OK
        dump("d = ", d);
    
        auto g = Bar{};     // OK
        dump("g = ", g);
    }
    

Output of VS2013:

    FooBar init, a = { 1, 2 }
    FooBar init, b = { 16853760, 4 }
    FooBar init, e = { 16853760, 4 }
    FooBar init, f = { 16853760, 4 }
    FooBar init, h = { 16853760, 4 }
    FooBar init, c = { 0, 0 }
    FooBar init, d = { 0, 0 }
    FooBar init, g = { 0, 0 }
    

Output of CLANG 7.0 / GCC 7/8

    FooBar init, a = { 1, 2 }   // compiler warning: missing field 'fb' initializer
    FooBar init, b = { 0, 4198208 }
    FooBar init, e = { 0, 0 }
    FooBar init, f = { 0, 0 }
    FooBar init, h = { 0, 0 }
    FooBar init, c = { 0, 0 }
    FooBar init, d = { 0, 0 }
    FooBar init, g = { 0, 0 }
    

So first thing we notice is that the constructor of FooBar is called but the members of Foo can still be uninitialized? This is because we have to distinguish between 'default-initialized' and 'value-initialized'.

For 'a' Bar is explicitly value initialized, that means constructors are called, and basic types are assigned their default value (0). However, 'b' is 'default-initialized' which means that any constructors are called, but no initialization is performed for basic types. Not only the constructor of the object is called, also all constructors of any nested types are called, if any.

Although it looks like the different compilers disagree about the initialization, uninitialized values on the stack can be any value and also: reading from an uninitialized values causes undefined behaviour (UB). So we cant really drawn any conclusions at this point.

I have tried to fill the stack with garbage between tests, but the tests where unaffected.

I think that since compilers seems to disagree about:

    auto b = Bar(); 
    Bar b = Bar();    
    

they should be avoided and:

    Bar b = {};
    

is going to be (and also has been) my preferred syntax. This is because it seems to be value-initializing correctly on all compilers (that I tested here).

Conclusions:

*   the brace syntax is a good way to make sure you are **value** initializing objects that have no constructor. 
*   if you make sure constructors always initialize all basic typed member variables you will never have uninitialized values in those objects no matter what syntax is used.

 [1]: http://eel.is/c++draft/dcl.init