---
ID: 271
post_title: 'std::shared_ptr<> broken?'
author: Jan Wilmans
post_excerpt: ""
layout: post
permalink: >
  http://nullptr.nl/2017/09/shared_ptr-broken/
published: true
post_date: 2017-09-03 00:02:00
---
Some time ago, I decided I had an object I wanted to share the ownership of. Here is a very contrived example:

    class Foo
    {
    public:
        void SetInfo(const std::string& value) { info = value; }
        std::string GetInfo() { return info; }
    private:
        std::string info;
    };
    
    class Bar
    {
        void SetFoo(std::shared_ptr<Foo> aFoo) { foo = aFoo; }
        void doUpdate()
        {
            auto myFoo = foo;
            if (myFoo)
            {
                doScreenUpdate(*myFoo);
            }
        }
        void doScreenUpdate(const Foo& aFoo)
        {
            std::cout << aFoo.GetInfo() << "\n";
        }
    private:
        std::shared_ptr<Foo> foo;
    };
    

Two threads are using Bar to update Foo, one only reads values occasionally updating the screen, the other updates values coming in. Now I had learned from Herb Sutters 'Writing Good C++14 By Default' talk at CppCon 2015, (from 0:58:00 onwards) that you should not use a member-shared_ptr directly, because it could be invalidated while you're using it. So like a good programmer, I copy it into myFoo before using it. Assuming SetFoo() is called before doUpdate() is, for example before any thread is started, there is no problem.

However, lets look at what would happen if SetFoo() is called while doUpdate() is in progress. If SetFoo() sets foo to a new value before 'auto myFoo = foo;' then myFoo will be valid and the doScreenUpdate() will just show the new value and otherwise, it will show the old value, right? What about when we call SetFoo(nullptr); ? Well if that happens before foo is assigned to myFoo, then nothing will happen and otherwise it will show the old value. Or so I thought...

**The actual answer: no, its undefined behaviour in all cases.**

The assumption Herb made in his talk, is that the modifier of foo is a recursive call on the same thread. However in this case the modifier is on another thread and things get more complicated.

The result in myFoo after assigning foo is: a copy of the old value of foo, a copy of the new value of foo, **or a mix of the two**. The first time I saw this behaviour, I thought there must be some terribly wrong with my program and it turns out there was.

**The problem here is that access to a shared_ptr<> is not atomic.**

Now, implementations shared_ptr can vary, but on vs2017 its size is 8 bytes in x86 mode and 16 bytes in x64 mode. It basically comes down to two pointers, one to the actual object and one to the internal shared state. This means that if you copy the shared_ptr while at the same time assigning it a new value, you may end up with the object pointer of one and the shared state of the other. Needless to say this is very bad :)

I could go into possible solutions for this problem, but the actual session to be learned here is: do not access shared_ptr's from multiple threads. I got this confirmed from STL:

> std::shared_ptr<> protects its **refcount**, so simultaneous reads/copies of the same shared_ptr or read/writes of different shared_ptrs pointing to the same object, are ok.

So the refcount is protected, you can safely copy a shared_ptr across threads, but you cannot write to the same shared_ptr object from multiple threads. Again STL:

> The usual thread safety guarantees apply. Multiple threads simultaneously read/writing a single object = bad.

And if you really have to, then make sure to use a synchronization primitive such as a mutex to make the access exclusive. As a final note: In C++14 we got std::atomic, unfortunately it will not help us in this case, as it can only operate on basic types. (bool, int, double, etc.)

In C++17 however we're likely to get support for std::atomic_shared_ptr<> which would allow this kind of usage. (Update: we didn't get std::atomic_shared_ptr<> in C++17, but is proposed for C++20). The only current implementation that I know of is a lock-free implementation available from the just::thread Pro library <https://www.stdthread.co.uk> by Anthony Williams.

other related resources:

*   <https://www.fluentcpp.com/2017/08/25/knowing-your-smart-pointers/>
*   <https://probablydance.com/2015/09/07/a-surprisingly-useful-little-class-twowaypointer/>
*   <https://github.com/briterator/SG14/blob/master/SG14/exposed_ptr.h>