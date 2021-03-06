---
ID: 1516
post_title: >
  How to learn more as a C++ software
  engineer?
author: Jan Wilmans
post_excerpt: ""
layout: post
permalink: >
  http://nullptr.nl/2019/01/better-cpp-software-engineer/
published: true
post_date: 2019-01-03 08:52:09
---
I couple of years ago, I was looking for new ways to improve myself. I had done several trainings on C++, C#, software processes, communication, leadership, etc. However, I was not really finding want I felt I needed to improve myself. This is of course, very subjective, the courses were good, I learned things, but I was not really satisfied with the pace nor the result.

Then at some point in 2016, I found [CppCast][1]. This is a weekly podcast with news on c++ topics. Things started rolling from there, a few months later, I was a guest on cppcast [myself][2]. I learned about [cppcon][3], a c++ conference (back then held in [Bellevue][4]) and convinced my employer to sponsor my travel, entrance fee and stay there.

Seattle, is a 10 hour flight from the Netherlands, but well worth it! I was there for 10 days, including a pre-conference course by [Stephen Dewhurst][5]. I still feel these 10 days where a turning point for me. I met so many great people, I was in awe that so many teachers were all gathered there, easy to approach and talk to. And also many people, just like me, all searching for more knowledge.

After my first conference I was hooked, I learned about how the [ISO c++ committee][6] works, I met members of SG14 (the study group focused on low latency), I met [Odin Holmes][7] there and learned about the [embo.io][8] conference. In the months after, I visited more conferences, local talks and retrospectively listened to all episodes of CppCast (well over 100 by then, going on 200 now). I also learned about other c++ resources, communities and peer groups.

Looking back, once I found 'my way', the community I was looking for was there all along, I just did not know where to look. Below I will list resources that I found helpful, starting with the 'best starting points', followed by with what I hope is a useful set of links.

# Getting started

So where to start? What if you what to learn more about c++ and let's assume for the moment, you want to nerd out for the evening. (so stay at home, not out in the world, talking to people, that will be stage 2 :)

Good starting points:

*   join twitter
*   find [#include C++][9] 
*   join the [#include discord channel][10] 

Join twitter you may ask? Isn't that a little 2015? While it may seem that way at first, a surprising amount of software engineers and teachers are on twitter! I can really recommend it and if you encounter content you don't appreciate, just **mute** some accounts to keep your tweets interesting and on topic. Starting by following [@c_plus_plus][11] and me ([@janwilmans][12]) of course :) To get the most out of twitter, it helps to follow several more people, just click on [@c_plus_plus][11] and see who's following that and go from there.

The [#include C++][9] community is very welcoming to *everybody*, seriously, it is their goal to be inclusive in all aspects and you will find knowledgeable people on the discord channel who can give you pointers to new resources or answer questions directly. In there own words:

> IncludeCpp is a global, inclusive, and diverse community for developers interested in C++. Join our [discord][10].

## Watch talks from conferences for example: cppcon, cppnew or meeting c++ conferences

The links in this article is a rather arbitrary set, there is no particular order or purpose to them, other then to show you that the information exists. Really it is out there, search on google, search on youtube, ask around on twitter for specific topics. Below I list a few I particularly liked or found useful in my daily job.

## Recommended talks

*   [CppCon 2015: Joshua Gerrard "The dangers of C-style casts"][13] is a nice 5-minute talk to get started with.
*   [CppCon 2015: T. Winters & H. Wright “All Your Tests are Terrible..."][14] 
*   [ITT 2016 - Kevlin Henney - Seven Ineffective Coding Habits of Many Programmers][15]
*   [CppCon 2016: Howard Hinnant - A ＜chrono＞ Tutorial (it's about time)][16]

## C++ resources

*   <https://abseil.io/tips/>
*   <http://ericniebler.com/>
*   [Fluent C++ - Jonathan Boccara's blog][17]
*   <https://www.youtube.com/c/Cppchat>
*   <http://cppcast.com/>
*   [Jason Turner's C++ Weekly][18] 

## More general resources

*   [Kevlin Henney][19] is a great speaker on many topics, check him out, also on [youtube][20].

## Tips on tweeting and posting on forums:

*   ask questions! and when you do explain **what you already did** to explore the problem and why you did not get to the desired answer/result. This helps to give context and avoids answers like 'just google it'.
*   use common sense, be nice, if you see a question you can answer, why not chip in and answer it. You may sometimes get it wrong, but that is OK, you will learn from that also!

# Stage 2, meanwhile in real life...

*   go to a conference! (see list of conferences below) they are the most immersive way to learn a lot in a short amount of time
*   if a conference seems too expensive, remember that volunteers and students often get special discounts and some even offer free access!
*   if you are employed, remind your boss that its is no more expensive compared to a regular course/training and much more immersive and interactive.
    
    *   <https://channel9.msdn.com/events/CPP/> the cppcon conference produces over 100 recorded talks ever year!
    *   <https://www.google.nl/search?q=youtube+cppcon>

## Other things to do

*   read the manual [cppreference.com][21]
*   watch the conference video's! there are many many hour of premium content available online, free of charge
*   read books (really!), I recommend, in order:
    
    *   [A Tour of C++ by Bjarne Stroustrup][22] [>bol<][23]
    *   [Effective Modern C++ by Scott Meyers][24] [>bol<][25]
    *   [C++ concurrency in action by Anthony Williams][26] [>link<][27]

*   good references
    
    *   [The C++ Programming Language (4th Edition)][28] 
    *   [The ISO Standard document, the official specification][29]

These last ones are not good to start with, but it an excellent reference if you want to dive into the full details.

I got feedback from some the great folks on the [orange page][30] that just reading books is not going to make you a good software engineer. Also they mentioned: "all that stuff about >Variadic templates, meta programming, memory barriers and the specifics of memory layout< do you really need to know all that? Is that not too obscure and not used on daily basis" ?

There is absolutely truth in this. First of all: practice, practice, practice. Programming is, just like so many things a skill that is really honed by doing it *a lot*. Still the mentioned books are great. "A Tour of C++", make sure to get the second edition that was just released, is only 256 (yes!) pages. I encourage you to read a few chapters and as soon as you found something new, do try it! Open up your favorite editor and try it out. Also by all means skip chapters that you find difficult, or uninteresting, read through the books to get a sense of the whole and return to the more difficult to grasp parts later.

Also worth mentioning, [The 7 Habits of Highly Effective People][31] is not about c++ at all, its about self-management, but it changed my life. Of course there are many other good books, these are just the ones that I read and really stuck with me.

*   join communities
    
    *   <https://stackoverflow.com/> is an excellent place for discussion and to ask questions 
    *   join [cpplang on slack][32]

## List of conferences

*   <https://cppcon.org/> ([student discounts][33]) 
*   <http://embo.io/>
*   <http://cppnow.org/> 
*   <https://cpponsea.uk/>
*   <https://meetingcpp.com/>
*   <https://codedive.pl/> (free!)

## Meet dutch C++ people at local groups:

*   <https://twitter.com/TheDutchCppGrp>
*   <http://www.040coders.nl>

## Good talks on slightly more advanced topics

*   [CppCon 2017: Ben Deane & Jason Turner “constexpr ALL the Things!][34]
*   [CppCon 2016: Ben Deane - Using Types Effectively][35]
*   [CppCon 2018: Robert Ramey - Safe Numerics][36]

## Funny videos about computer complexities

Take these with a grain of salt, yes there is truth to them, but you will probably never have to solve these problems yourself.

*   [The Problem with Time & Timezones - Computerphile][37]
*   [Internationalis(z)ing Code - Computerphile][38]

## ping-backs

*   <https://www.kylegalbraith.com/learn-by-doing/volume/36/how-do-you-feel-about-cplusplus.html>
*   <https://news.ycombinator.com/item?id=18814697>

## Note from the author

This post got an unusual amount of attention, I would like to emphasize that it a work in progress, I will update it regularly.

 [1]: http://cppcast.com
 [2]: http://cppcast.com/2017/09/jan-wilmans/
 [3]: https://cppcon.org/
 [4]: https://www.google.nl/maps/place/Bellevue,+WA,+USA
 [5]: https://www.youtube.com/watch?v=PFdWqa68LmA
 [6]: https://isocpp.org/
 [7]: https://www.youtube.com/watch?v=tNXyNa6kf4k&t=1s
 [8]: http://embo.io/
 [9]: https://twitter.com/include_cpp
 [10]: https://t.co/XafTulMibe
 [11]: https://twitter.com/c_plus_plus
 [12]: https://twitter.com/janwilmans
 [13]: https://www.youtube.com/watch?v=DAvZ3OG9cNo
 [14]: https://www.youtube.com/watch?v=u5senBJUkPc
 [15]: https://www.youtube.com/watch?v=ZsHMHukIlJY&t=2369s
 [16]: https://www.youtube.com/watch?v=P32hvk8b13M
 [17]: https://www.fluentcpp.com/
 [18]: https://www.youtube.com/playlist?list=PLs3KjaCtOwSZ2tbuV1hx8Xz-rFZTan2J1
 [19]: https://twitter.com/KevlinHenney
 [20]: https://www.google.nl/search?tbm=vid&q=Kevlin%20Henney
 [21]: https://cppreference.com
 [22]: https://www.amazon.com/Tour-C-Depth/dp/0321958314
 [23]: https://www.bol.com/nl/p/a-tour-of-c/9200000096584509/
 [24]: https://www.oreilly.com/library/view/effective-modern-c/9781491908419/
 [25]: https://www.bol.com/nl/p/effective-modern-c/9200000036037659/
 [26]: https://www.manning.com/books/c-plus-plus-concurrency-in-action-second-edition
 [27]: https://www.bogotobogo.com/cplusplus/files/CplusplusConcurrencyInAction_PracticalMultithreading.pdf
 [28]: http://www.stroustrup.com/4th.html
 [29]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2017/n4659.pdf
 [30]: https://news.ycombinator.com/item?id=18814697
 [31]: https://www.franklincovey.com/the-7-habits.html
 [32]: https://cpplang.now.sh/
 [33]: mailto:students@cppcon.org
 [34]: https://www.youtube.com/watch?v=PJwd4JLYJJY
 [35]: https://www.youtube.com/watch?v=ojZbFIQSdl8
 [36]: https://www.youtube.com/watch?v=93Cjg42bGEw
 [37]: https://www.youtube.com/watch?v=-5wpm-gesOY
 [38]: https://www.youtube.com/watch?v=0j74jcxSunY