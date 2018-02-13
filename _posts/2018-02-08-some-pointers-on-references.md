---
ID: 683
post_title: >
  Personal preferences for the use of
  pointers and references
author: Jan Wilmans
post_excerpt: ""
layout: post
permalink: >
  http://nullptr.nl/2018/02/some-pointers-on-references/
published: true
post_date: 2018-02-08 11:38:52
---
A blog post about pointers and references? What kind of newbie am I? Well, the kind with 20+ years of programming experience that is not afraid to admit his ignorance, apparently?

on the other hand;

*   maybe its not that easy?
*   maybe I didn't paying that much attention when it was explained to me?
*   maybe things have changed over time?
*   or maybe I just have not spend enough time thinking about the topic?

**I think it might be all of the above.**

> How can we remember our ignorance, which our growth requires, when we are using our knowledge all the time? - Henry David Thoreau

## So what do we have, lets look at some [history][1] first

First came **pointers**, they were the first of their kind in C++, as inherited from C in 1979 they:

*   can be left uninitialized
*   can be null assigned
*   can be reassigned.
*   do not express ownership
*   allow (pointer) arithmetic
*   can be copied and moved

Then there were **references**, they where introduced in 1985 [for operator overloading][2] but also have other benefits. They are by their very nature always initialized. This is because it is not possible to declare a reference without also initializing it. Their most important defining attribute, aside from their slick syntactical appearance: they cannot be reassigned. References basically are pointers, but support only a subset of the features and more convenient syntax. This makes them hard(er) to use incorrectly.

So references:

*   are always initialized
*   can be null assigned (but please never do that)
*   **cannot be reassigned**
*   explicitly express the absence of ownership
*   don't allow arithmetic
*   can be copied and moved

Then came **smart pointers**, well they had a rough start, std::auto_ptr had its problems and is best forgotten. The reference counted shared_ptr appeared in boost around 2001. Later came unique_ptr and recently atomic_shared_prt. Then came new C++ standards now including **std::shared_ptr**, **std::unique_ptr**. (C++11), **std::atomtic_shared_prt** is coming in C++20 to the standard library.

Smart pointers are called smart because they manage the lifetime of objects in some way. shared_ptr does this by reference counting, so the last instance of shared_ptr releases the memory of the object. I contrast unique_ptr explicitly expresses exclusive ownership and releases memory when it leaves scope.

**shared_ptr**

*   cannot be uninitialized
*   can be null assigned
*   can be reassigned
*   express shared ownership
*   can be copied and moved

**unique_ptr**

*   cannot be uninitialized
*   can be null assigned
*   can be reassigned
*   express shared ownership
*   can be moved but **can not be copied**

Enter the [GSL][3] in 2015 including, among many other things, [not_null<> and owner<> template][4]:

**not_null<Foo*>**

*   can not be uninitialized
*   cannot be null assigned
*   can be reassigned
*   do not express ownership
*   can be copied and moved

They can also be combined, for example:

**not_null<shared_ptr></shared_ptr>**

*   cannot be uninitialized
*   cannot be null assigned
*   can be reassigned
*   express shared ownership
*   can be copied and moved

A similar albeit slightly different in philosophy is the [nn<> type from dropbox][5].

Throughout this post, where I talk about 'pointers', I mean all kinds, so: raw, smart, atomic and also references. This summary is certainty not an exhaustive list of existing (smart) pointers but lets compare the specimens we have here.

# Different kinds of pointers with different use cases

Pointers, or what today we like to call 'raw pointers', to contrast them from 'smart' pointers, have kind of folklore about them. They are attributed qualities like 'bad style' or even 'being evil'. I my opinion pointers are useful and harmless in themselves but like anything they can be misused in a way that can become problematic.

> "Warning: towels can be harmful if swallowed in large quantities". -- Douglass Adams.

That being said, I think in general pointers should be considered a last resort, not a default. Or to put it another way: **Use a reference if you can, use a pointer if you have to.**

As a rule of thumb I:

*   prefer pointers that express ownership 
*   prefer pointers that cannot be uninitialized 
*   prefer pointers that cannot be null
*   prefer pointers that cannot be copied

This basically orders my preference for pointers as:

    - Foo&
    - unique_ptr<Foo>
    - not_null<shared_ptr<Foo>>
    - shared_ptr<Foo>
    - [gsl::owner<Foo*>]
    - [gsl::not_null<Foo*>]
    - Foo*
    

[gsl::not_null<Foo*>][4] [gsl::owner<Foo*>][6]

Depending on the other attributes that you need, you may have the luxury of choice, however by default, reference and unique_ptr are probably your friends.

If you want to express that passing an argument is optional and the callee does not get ownership, a raw pointer fits the job. However, it will not express the intent explicitly, so it what are the alternatives?

For example:

    void Func(Foo* foo, int x) 
    {
        if (foo == nullptr)
        { 
            // throw / report error
        }
        foo->Func(x);
    }
    

Might be more clearly expressed as:

    void Func(gsl::not_null<Foo> foo, int x) 
    {
        foo->Func(x);
    }
    

This has two advantages; first of all it makes explicit that the argument is not optional and second, gsl-aware static analysis tools can use the type information to give better diagnostic messages. However, there is still no expression of ownership, so we might write:

    void Func(Foo& foo, int x)
    {
        // do X
        foo.Func(x);
    }
    

Here we use a reference to express explicitly that our 'foo' argument is not optional and the caller does not receive ownership. So if references are so great, why not just always use them? Consider this data structure:

    struct Time {}
    struct Task 
    {
        Time& time;
    };
    
    Time t;
    Task task = { t };
    

Here a Task can be created, but can never be rescheduled, which would mean that to rescheduled the Task, you would have to create a new Task. This can lead to very impractical structures, so instead it could be expressed as:

    struct Task 
    {
        not_null<Time*> time;
    };
    
    Time t;
    Task task = { &t };
    

Here a Task can be assigned a time, it has to have a time at construction, the time can be changed later, but can only be re-assigned another Time object.

There is another advantage to this because references can be null assigned and to make matters worse, there is no way to check when that happens.

    Time * t = nullptr; 
    Task task = { *t }; // please never null assign a reference!
    

Then to stack up more potential problems:

    void dowork(Foo& foo)
    {
    }
    
    struct Bad
    {
        void set(Foo* f) { foo = f; }    
        void use() { dowork(*f); } 
        Foo* foo;
    }
    

If this code is used in more one then one thread at once, possibly one thread calls use() and another calls **set()** setting foo to nullptr, no amount of checking in dowork() can prevent the potential for a crash. Of course protecting the access to **foo** using a mutex could solve that and there are also [better ways][7], but solutions is not what this part of the post is about. We're digging into the problems first.

Note that using a reference is not causing this problem and its not even responsible for hiding the problem. The problem is that calling this code from 2 threads was unsafe to begin with, no matter what kind of pointer you use.

This does not mean references or raw pointers (or any kind of pointer for that matter) are bad. The problem here was that the author did not think about who controls the lifetime of the object foo. You might think that using a unique_ptr or shared_ptr would improve this. Lets see about that:

    void dowork(std::shared_ptr<Foo> foo)
    {
    }
    
    struct Bad
    {
        void set(std::shared_ptr<Foo> f) { foo = f; }    
        void use() { dowork(f); } 
        std::shared_ptr<Foo> foo;
    }
    

This code is just as bad for use in two threads because 'foo' is potentially copied and re-assigned at the same time. See for more information my other blog posts:

<http://nullptr.nl/2018/01/introducing-mt_shared_ptr/> <http://nullptr.nl/2017/09/shared_ptr-broken/>

To summarize so guidelines I like use:

*   use references to express that no ownership is passed, it is be far the easiest to understand/read
*   never assign the value of a pointer to a reference if it was optional
*   not_null<> can help to make legacy code more readable without actually changing its meaning

 [1]: http://en.cppreference.com/w/cpp/language/history
 [2]: http://www.stroustrup.com/bs_faq2.html#pointers-and-references
 [3]: https://github.com/Microsoft/GSL
 [4]: https://github.com/Microsoft/GSL/blob/64a7dae4c6fb218a23b3d48db0eec56a3c4d5234/include/gsl/pointers#L69
 [5]: https://github.com/dropbox/nn
 [6]: https://github.com/Microsoft/GSL/blob/64a7dae4c6fb218a23b3d48db0eec56a3c4d5234/include/gsl/pointers#L51
 [7]: https://github.com/copperspice/libguarded