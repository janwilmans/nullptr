---
ID: 791
post_title: Common code smells by Ólafur Waage
author: Jan Wilmans
post_excerpt: ""
layout: post
permalink: >
  http://nullptr.nl/2018/02/common-code-smells/
published: true
post_date: 2018-02-11 20:33:13
---
The following post was triggered by a [twitter post][1] by Ólafur Waage about his observations of common mistakes during code reviews over a period of 4 months. Some of these were not news to me but I had not seen them collected as a set anywhere in a single place.

I have added and/or rephrased some passages based personal experience.

> [One function that does everything][2]

Many think that they should only create functions if they are reused. **No.** Create functions for anything that can be thought of as a generic action. Or create functions to document steps of a process. This greatly helps in readability and structure.

> [Early exits][3]

    int foo() 
    { 
      if (x) 
      { 
        // lots of code ... 
         return y; 
      }
      return 0; 
    }
    

Reverse that statement. Now the reader has a lot to go through to make sure the "bad" exit case is ok, this takes up mental space and makes reasoning about you code more difficult. If it's reversed, everything below the if does not matter for the **x == false** case.

    int foo() 
    { 
      if (!x) return 0;
      // lots of code ... 
      return y; 
    }
    

This also reduces the amount of brackets and indentation levels.

> [Avoid magic numbers][4]

    if (something > 5) ...
    

What is this 5? Give it a name and put it somewhere, this number has context, show it. Put it in the scope that makes sense, it does not have to be in a big scope or reused, in the function itself is also good.

    const int maximumTemperature = 5;
    if (something > maximumTemperature ) ...
    

Notice that your brain is now probably suddenly trying to make sense of the function and questions arise like:

*   should this be a floating pointer number 
*   are these in Celsius of Fahrenheit

These are good questions and that kind of question may be answered by the name of the variable sometimes. This also leads into the question: should that not be part of the type of the variable so trying to assign degrees of Celsius to degrees Fahrenheit would be a compiler error. This can be accomplished by using the [boost units library][5]. There is an excellent [talk on this subject by Robert Ramey from CPPCON2015][6].

> [One class for everything][7]

Similar to one function for everything. If you can categorize the actions within the class, then you can make classes out of those categories. It is really quite a burden to maintain 6000 line classes. For example, if you have a 10-step process, than maybe create a class, give it the name of the process and create 10 functions to express the steps.

> [Inconsistent formatting style][8]

Reading through code is a strain mentally, not only do you have to parse what each line is doing, you need to parse the structure. If it's inconsistent it makes it harder to parse. What style doesn't matter, just make it the same everywhere. Tools can be a big help here. An example is [clangformat][9], it can format your code or just a selection of it to make it conform to an agreed style by the team.

> [Comments everywhere][10]

    // Increment value by one 
    i++;
    

My issue with comments is that now you have 2 meanings for everything. The line has a meaning and the comment. And they must match, or else there is trouble. Add a comment when something needs explaining, when reading the code by itself would not make sense. But you can almost always just make a function that describes that action, no comments needed then. The compiler does not read comments (obviously) so if the comment describe some precondition the must hold, just check the precondition or use static_assert is possible.

> [Performing complex actions in if statements][11]

    if (myFoo->Action(aBar + aBaz->GetStuff(18)))
    

The only time you should be placing direct actions in **if** statements is if you want to block the execution with an && ([so-called short circuit evaluation][12]). Otherwise put that into a variable and then ask the question.

> [Stringly typed][13]

    if (a == "foo") {...} else if(a == "bar") {...}
    

Unless you are doing actual string data parsing, you should be using classes, enums or at least numbers for determining what is what. Strings are slower to evaluate and offer no type-safety.

 [1]: https://twitter.com/olafurw/status/962442310473658369
 [2]: #one_func
 [3]: #earlyexits
 [4]: #magicnumbers
 [5]: http://%20%20http://www.boost.org/doc/libs/1_66_0/doc/html/boost_units.html
 [6]: https://www.youtube.com/watch?v=qphj8ZuZlPA&t=1121s
 [7]: #oneclass
 [8]: #formatting
 [9]: https://llvm.org/builds/
 [10]: #comments
 [11]: #complexaction
 [12]: https://en.wikipedia.org/wiki/Short-circuit_evaluation
 [13]: #stringly