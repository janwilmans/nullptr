---
ID: 868
post_title: >
  Enhance type safely using Opaque
  Typedefs aka phantom types
author: Jan Wilmans
post_excerpt: ""
layout: post
permalink: http://nullptr.nl/2018/02/phantom-types/
published: true
post_date: 2018-02-22 00:05:57
---
Its seems every year (see references at the bottom) someone writes about having strong typedefs in C++. Hey, it's end of February and nobody did it for 2018 yet!?

The idea is simple (or so it seems). Suppose you would like to distinguish variables that have the same basic type and have the compiler enforce that, this does not seem possible at first glance. For example, this will not compile:

    using AxisId = int;
    using ControllerId = int;
    
    void f(AxisId id) {}
    void f(ControllerId id) {} // error: function 'void f(AxisId)' already has a body
    
    void g(AxisId id) {}
    void foo()
    {
       g(ControllerId::A);        // very wrong, but compiles fine and without warning
    }
    

Error 1 error C2664: 'void StrongTyped::g(StrongTyped::AxisId)' : cannot convert argument 1 from 'StrongTyped::ControllerId' to 'StrongTyped::AxisId' C:\Users\jan\Documents\Visual Studio 2013\Projects\initialized_vars\initialized_vars\initialized_vars.cpp 35 1 initialized_vars

The idea here is that axes and controllers are identified by an index, the indexes are not unique, both ranges contains 0, 1, 2 3, etc... Now function g() should only be called with an AxisId, but if you were to give it '1' or a ControllerId, that would happily compile. Also as the example shows, typedef'ed basic types cannot be used to overload a method.

This case is basically now a solved problem in C++11 with the introduction of 'enum class', as it can do a special trick for integers:

    enum class ControllerId     // regular enum class ControllerId
    {
        A, B, C
    };
    
    enum class AxisId           // specifically valued Id
    {
        x = 0,
        y = 1,
        z = 2
    };
    
    void f(AxisId id) {}
    void f(ControllerId id) {} // Ok, ControllerId is a unique type
    
    void g(AxisId id) {}
    void foo()
    {
       g(ControllerId::A); // error: cannot convert argument 1 from 'StrongTyped::ControllerId' to 'StrongTyped::AxisId' // nice, very clear message!
    }   
    

However, what about the case where there is not a known/fixed amount of members:

    using Days = int;
    using Weeks = int;
    
    int getWorkingHours(Days value)
    {
        return value* 8;
    }
    
    int getWorkingHours(Weeks value) // error: function 'int getWorkingHours(int)' already has a body
    {
        return value* 8 * 5;
    }
    

Here it would have been nice to overload the method 'getWorkingHours' with arguments of different type for 'day' or 'week'. It turns out this is actually not all that hard, let have the enum class do its thing:

    enum class Days;  // notice: enum declarations without any specific values 
    enum class Weeks;      
    
    template <typename T>
    auto value_of(const T& t) -> std::underlying_type_t < T >
    {
        static_assert(std::is_enum<T>::value, "argument of value_of is not an enum or enum class");
        return static_cast<std::underlying_type_t<T>>(t);
    };
    
    int getWorkingHours(Days value)
    {
        return value_of(value) * 8;
    }
    
    int getWorkingHours(Weeks value)
    {
        return value_of(value) * 8 * 5;
    }
    
    void strong_integer()
    {
        std::cout << "hours in 2 days: " << getWorkingHours(Days(2)) << "\n";    // outputs: 16
        std::cout << "hours in 2 weeks: " << getWorkingHours(Weeks(2)) << "\n";  // outputs: 80
    }
    

What happens here is that by specifying the enum class without any specific values it can only be assigned by being explicit about it, which is exactly what we want.

The 'value_of' template is not strictly necessary, a 'static_cast<int>(x)' would also work just as well. However you would have to make sure you specify the correct underlying type, otherwise you risk a narrowing conversion without any diagnostic warnings because they would be silenced by the static_cast<>. By using the value_of template you don't have to repeat yourself *and* if the underlying type would change at some point appropriate warnings would be emitted.</int>

So, this seems to work quite well of integer values, but what about other basic types, like for example floating point numbers?

# Various use cases for strong types

These are examples of applications of strong types from different fields. The use cases vary slightly but they are all in the same category of **distinct basic types**.

*   distinct units: disallow mixing up quantities in different dimensions (distance, weight, volume, etc.)
*   distinct ids: disallow mixing up the handles or ids of different kinds
*   distinct memory: disallow mixing real-, virtual- memory addresses and offsets

another way to look at the application of the district types is:

*   make overloading possible for different types basic off the same basic type.
*   make non-sensible operations compilation errors, such as dividing by an offset or subtracting an address from an offset.
*   verifying data: have user-input validated and put into a verified type

This is an example of an fairly simple 'Strong type'. Different instances of StrongType<> are not interchangeable, so you as the developer will have to be explicit about any conversion.

    template <typename T, typename ID>
    class StrongType
    {
    public:
        StrongType(const StrongType&) = default;
        StrongType(StrongType&& value)
        {
            m_value = value.m_value;
        }
        StrongType& operator=(const StrongType&) = default;
        StrongType& operator=(StrongType&& other)
        {
            if (this != &other)
            {
                m_value = other.m_value;
            }
            return *this;
        }
        explicit StrongType(T value) : m_value(value) {} // basic type constructor
        virtual ~StrongType() = default;
    
        double get() const { return m_value; }
    
    private:
        T m_value;
    };
    
    template <typename S, typename ID>
    std::ostream& operator<< (std::ostream& os, const StrongType<S, ID>& value)
    {
        os << value.get();
        return os;
    }
    
    struct tag_meter {};
    struct tag_millimeter {};
    
    using Meters = StrongType < double, tag_meter >;
    using MilliMeters = StrongType < double, tag_millimeter >;
    
    MilliMeters toMillimeters(Meters m)
    {
        return MilliMeters(m.get() * 1000.0); // explicit construction and .get() are needed 
    }
    
    Meters toMeters(MilliMeters m)
    {
        return Meters(m.get() / 1000.0);
    }
    
    void strong_double()
    {
        Meters m = Meters(1.2);
        auto mm = toMillimeters(m);
        auto m2 = toMeters(mm);
        std::cout << "the length is: " << mm << "\n";
        std::cout << "the length is: " << m2 << "\n";
        //mm = m2; // no operator found which takes a right-hand operand of type 'StrongTyped::Meters
        //m2 = mm; // no operator found which takes a right-hand operand of type 'StrongTyped::MilliMeters 
    }
    

I have made the 'template class StrongType' a little more verbose by implementing the move assignment operator and move constructor explicitly so it can be used on somewhat older compilers as well. On a C++17 compliant compilers all [rule-of-five][1] constructors could be defaulted.

So this is all very nice, but we can take it one step further; we could implement assignment operators for Meters on a Millimeter class and vice versa.

    namespace si
    {
        class MilliMeters;
    
        using MetersType = StrongType < double, tag_meter > ;
        class Meters : public MetersType
        {
        public:
            Meters(double v) : MetersType(v) {}
            Meters(const MilliMeters& v);
            Meters& operator=(MilliMeters);
        };
    
        using MilliMetersType = StrongType < double, tag_millimeter >;
        class MilliMeters : public MilliMetersType
        {
        public:
            MilliMeters(double v) : MilliMetersType(v) {}
            MilliMeters(const Meters& v);
            MilliMeters& operator=(Meters);
        };
    
        Meters::Meters(const MilliMeters& v) : MetersType(v.get() / 1000.0) {}
    
        Meters& Meters::operator=(MilliMeters value)
        {
            set(value.get() / 1000.0);
            return *this;
        }
    
        MilliMeters::MilliMeters(const Meters& v) : MilliMetersType(v.get() * 1000.0) {}
    
        MilliMeters& MilliMeters::operator=(Meters value)
        {
            set(value.get() * 1000.0);
            return *this;
        }
    
        void h(MilliMeters value) {}
    
        void foo()
        {
            Meters m = Meters(2); 
            h(m);
        }
    }
    

As you can see, this is becoming more and more cumbersome. It makes sense to be able to add/subtract Meters from Millimeters. But that would also make sense for types like: picometers, micrometers, centimeters, decimeters, kilometers, etc etc. And why not addition ? Division? Multiplication ? Meters squared could actually return a new type SquareMeters... The amount of operations quickly explodes and became very tedious to implement. Fortunately, ideas are rarely unique or new, so searching for an existing solution quickly yielded one: [Boost.Units][2]

The Boost.Units library offers many SI units and common operations out of the box. Its documentation leaves much to be desired, but watching [Robert Ramey's CPPCON 2015 talk "Boost Units Library for Correct Code"][3] is a good introduction.

To summarize: Strong types can be implemented in C++ and getting basic type safety that way is not that difficult. The downside of doing a naive implementation is that for doing arithmetic requires lots of explicit constructions and .get() calls which is ugly or at best, no longer looks natural. For SI units Boost.Units can offer an alternative. As a final note: the ideas in this post were distilled from the references below, the code samples above were written be me and can be used freely.

> References

[C++Now 2017: Jonathan Müller “Type-safe Programming"][4]  
[Arne Mertz' 2016 Simplify C++ artical "Use Stronger Types!"][5]  
[CppCon 2015: Kyle Markley "Extreme Type Safety with Opaque Typedefs"][6] => [Link to Kyle Markley's sources][7]  
[CppCon 2015: Robert Ramey “Boost Units Library for Correct Code"][3]  
[The blog at the bottom of the sea - Getting Strongly Typed Typedefs Using Phantom Types][8]  
[Jonathan Müller's blog about strong typedefs][9] => [Link to Jonathan Müller's sources][10]  
[Link to Opaque typedef proposal][11]  
[WG21/N1891 = J16/05-0151 Walter E. Brown's 2005 Opaque Typedefs proposal][12]  
[N2141 = 06-0211 Alisdair Meredith Strong Typedefs in C++09][13] [Boost BOOST_STRONG_TYPEDEF][14]

 [1]: http://en.cppreference.com/w/cpp/language/rule_of_three
 [2]: http://www.boost.org/doc/libs/1_66_0/doc/html/boost_units.html
 [3]: https://www.youtube.com/watch?v=qphj8ZuZlPA&t=1192s
 [4]: https://www.youtube.com/watch?v=iihlo9A2Ezw
 [5]: https://arne-mertz.de/2016/11/stronger-types/
 [6]: https://www.youtube.com/watch?v=jLdSjh8oqmE
 [7]: https://sourceforge.net/projects/opaque-typedef/
 [8]: https://blog.demofox.org/2015/02/05/getting-strongly-typed-typedefs-using-phantom-types/
 [9]: http://foonathan.net/blog/2016/10/19/strong-typedefs.html
 [10]: https://github.com/foonathan/type_safe
 [11]: https://github.com/viboes/opaque/blob/master/libs/opaque/doc/opaque.pdf
 [12]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2005/n1891.pdf
 [13]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2141.html
 [14]: http://www.boost.org/doc/libs/1_61_0/libs/serialization/doc/strong_typedef.html