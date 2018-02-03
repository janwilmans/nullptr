---
ID: 604
post_title: 'Introducing mt_shared_ptr<>, a shared pointer for multi-threaded use'
author: Jan Wilmans
post_excerpt: ""
layout: post
permalink: >
  http://nullptr.nl/2018/01/introducing-mt_shared_ptr/
published: true
post_date: 2018-01-27 20:09:02
---
Disclaimer: this is a work-in-progress, is bound to contain some mistakes and the code was only briefly tested. Use at you own discretion.

In a previous blog post called '[std::shared_ptr<> broken?][1]' I talked about the problems of using a std::shared_ptr<> from more then one thread at the same time.

There I mentioned the reading/writing to a single std::shared_ptr<> (stored as a member variable?) is unsafe and causes undefined behaviour. To re-iterate:

*   a std::shared_ptr<> protects its ref-count but only as long as the same instance of the std::shared_ptr<> is not read/modify by multiple threads as the same time.
*   maybe needless to say: it does not protect the access to the object it points to at all.

So it is safe to read/modify a copy of a std::shared_ptr<>, if each thread has its own copy. However, if you need to modify it from two threads at the same time, or even read from one thread and modify from another thread, you need some kind of locking mechanism to do that correctly.

In this post I will introduce a poor man's version of std::atomic_shared_ptr<> with a small addition that makes it harder to use incorrectly.

So first of all, before we can fix anything, we need something broken, here it is. First we create a class Foo with a method Bar() that when called after destruction will have a high likelihood of causing a crash:

    class Foo
    {
    public:
        Foo()
        {
            pint = &count;
        }
    
        ~Foo()
        {
            pint = nullptr;
        }
    
        void Bar()
        {
            sink = *pint;   // make sure a Bar() call on a destroyed Foo object will crash the process
        }
    
    private: 
        int sink;
        int count;
        int * pint;
    };
    

Then we just access it very frequently in one thread (t1), while re-assigning it new values in another thread (t2).

    #include <iostream>
    #include <exception>  
    #include <thread>  
    
    #include <memory>
    #include <mutex>
    
    int main()
    {
        std::shared_ptr<Foo> f = std::make_shared<Foo>();
        bool running = true;
        std::thread t1([&]
        {
            while (running)
            {
                try
                {
                    //f->Bar();  // causes crashes (and that is the expected and correct behaviour)
                    auto c = f;
                    if (c)
                        c->Bar();
                }
                catch (const std::exception& e)
                {
                    std::cout << "exception: " << e.what() << "\n";
                }
            }
        });
    
        std::thread t2([&]
        {
            for (int i = 0; i < 1e6; ++i)
            {
                f = std::make_shared<Foo>();
                f.reset();
            }
        });
    
        t2.join();
        running = false;
        t1.join();
        std::cout << "All was fine\n";
        return 0;
    }
    

The result, as expected: a crash (on windows that looks like this)

<table>
  <tr>
    <td>
      <img src="http://nullptr.nl/wp-content/uploads/2018/01/mt_shared_ptr_crash.png" alt="crash dialog" width="366" height="186" class="alignleft size-full wp-image-608" />
    </td>
  </tr>
</table>

Now to fix this, we need to change very little:

    #include "mt_shared_ptr.h"
    
    int main()
    {
        mt_shared_ptr<Foo> f = std::make_shared<Foo>(); // replaced std::shared_ptr<> with mt_shared_ptr<> 
        bool running = true;
        std::thread t1([&]
        {
            while (running)
            {
                try
                {
                    //f->Bar();  // this will now throw instead of crashing
                    auto c = f.share();  // added .share() to take an explicit copy
                    if (c)
                        c->Bar();
                }
                catch (const std::exception& e)
                {
                    std::cout << "exception: " << e.what() << "\n";
                }
            }
        });
    
        std::thread t2([&]
        {
            for (int i = 0; i < 1e6; ++i)
            {
                f = std::make_shared<Foo>();
                f.reset();
            }
        });
    
        t2.join();
        running = false;
        t1.join();
        std::cout << "All was fine\n";
        return 0;
    }
    

So how does that work? Well, in mt_shared_ptr, there is a mutex that gets locks for every operation performed on it. So when you call f.reset(), call .shared() or re-assign it, it locks the mutex first, so the access to the internal shared_ptr<> is always exclusive.

I hear some of you thinking: arrgh, that must be sooo slow. Well, I'm glad you asked :) Lets do some inaccurate and statistically worthless measurements. The 1e6 (1 million) iterations take about 4.0 seconds, compared to 1.5s when not using it. Which means introducing something like 2.5us for every access. So I guess you don't what to do this in a tight loop but remember you only have to call share() once it get an exclusive copy of the shared_ptr<> and after that the overhead is back to what it was with a plain std::shared_ptr<>.

Also, there is one more trick mt_shared_ptr can do: if you call operator-> and the internal shared_ptr is empty, it throws an exception instead letting it be dereferenced.

NOTE: as pointed out by Peter Dimov there is a point of attention that the destruction of the object pointed to by ptr in class mt_shared_ptr should never run while holding the lock. I've introduced the use of swap() and local temporary variables in the operator= and reset() methods to make sure that does not happen.

Also I've removed the locks from the constructors, because only one constructor will ever run and only once for any single object.

Note 1: it has been suggested to remove operator-> because consecutive calls could go to different but valid objects, such as:

      sp->func1();  // sp refers to object A
      sp->func2();  // sp might refer to object B
      sp->func3();  // sp might refer to object C
    

Since sp could be re-assigned between calls, you probably wanted:

    auto p = sp.share();
    p->func1();
    p->func2();
    p->func3();
    

I consider this sane/expected behaviour and you could still write sp.share()->func1(); several times if the operator-> was not there. (don't do that) However, I think it is a valid argument that it may provoke incorrect usage (its easy to use incorrectly). So I've commented out the operator-> but left it in commented to allow to be used for step-by-step migration of existing code. You can drop in mt_shared_ptr<> as a shared_ptr<> replacement and code will fail to compile in places where operator-> was used (lots of place I expect). So then you start fixing them, but if it turns out to be too much work to do in one go you could temporarily enable the operator-> to make it compile again and allow you to run tests etc.

Note 2: I tried to make mt_share_ptr<> behave like a **regular type**. This basically means that is has value semantics. However, it is not a regular type because that would mean we have to: - use the [rule of 5][2] (we do) - be equationally complete, that is, operator==() should be implementable as a non-friend, non-member, function. - in C++11 (or above) you should provide a specialization of std::hash<> for equationally complete types - implement operator< to provide a total ordering (or specialize std::less<>() if a natural total ordering is not available)

One might be temped to think some ==, != and <> operators would be nice to have on class mt_shared_ptr, however, this is really just provoking more incorrect usage.

So there we have it, a sub-optimal, but very effective alternative to std::atomic_shared_ptr<> that can used with any C++11 (and up) compiler.

* * *

mt_shared_ptr.h:

    /*
    Copyright 2018 Jan Wilmans
    
    Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files(the "Software"), 
    to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, 
    and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions :
    
    The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
    
    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, 
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, 
    WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
    */
    
    #include <iostream>
    #include <exception>  
    #include <thread>  
    
    #include <memory>
    #include <mutex>
    
    template <typename T>
    class mt_shared_ptr
    {
        using ptr_t = std::shared_ptr<T>;
    public:
        mt_shared_ptr() = default;
        mt_shared_ptr(const mt_shared_ptr& t) : ptr(t.share()) {}
        mt_shared_ptr(mt_shared_ptr&& other) : ptr(other.ptr)
        {
            other.ptr.reset();
        }
    
        mt_shared_ptr& operator=(const mt_shared_ptr& other)
        {
            if (this != &other)
            {
                ptr_t p = ptr;
                ptr_t q = other.share();
                std::lock_guard<std::mutex> lock(mutex);
                ptr = q;
            }
            return *this;
        }
    
        // no locking needed since moving implies the rhs is not accessed
        mt_shared_ptr& operator=(mt_shared_ptr&& other)
        {
            if (this != &other)
            {
                ptr = std::move(other.ptr);
            }
            return *this;
        }
    
    
        // we make sure we do not (possibly) run the destructor of the object that ptr references while holding the lock
        mt_shared_ptr & operator=(ptr_t p) // shared_ptr assignment
        {
            std::lock_guard<std::mutex> lock(mutex);
            ptr.swap(p);
            return *this;
        }
    
        mt_shared_ptr(const ptr_t& p) : ptr(p) {} // constructor taking a shared_ptr<T>
    
        ptr_t operator->() const
        {
            std::lock_guard<std::mutex> lock(mutex);
            if (!ptr) throw std::runtime_error("mt_shared_ptr<> nullptr dereference");
            return ptr;
        }
    
        ptr_t share() const
        {
            std::lock_guard<std::mutex> lock(mutex);
            return ptr;
        }
    
        void reset()
        {
            ptr_t delayRelease;
            std::lock_guard<std::mutex> lock(mutex);
            delayRelease = ptr;
            ptr.reset();
        }
    
        void reset(ptr_t p)
        {
            std::lock_guard<std::mutex> lock(mutex);
            ptr.swap(p);
        }
    
    private:
        mutable std::mutex mutex;
        ptr_t ptr;
    };
    

References:

*   as pointed out by Peter Dimov and Glen Fernandes, there is also a [boost implementation of atomic_shared_ptr][3] 
*   <http://www.boost.org/doc/libs/1_66_0/libs/smart_ptr/doc/html/smart_ptr.html#shared_ptr_free_functions>
*   <http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2674.htm>
*   <http://www.open-std.org/Jtc1/sc22/wg21/docs/papers/2014/n4033.html>

<div class="footer">
  Blog post "Introducing mt_shared_ptr<>, a shared pointer for multi-threaded use" © 2018 nullptr.nl
</div>

 [1]: http://nullptr.nl/2017/09/shared_ptr-broken/
 [2]: http://en.cppreference.com/w/cpp/language/rule_of_three
 [3]: http://www.boost.org/doc/libs/1_66_0/libs/smart_ptr/doc/html/smart_ptr.html#shared_ptr_atomic_access