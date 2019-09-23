---
ID: 1975
post_title: CPPCON 2019 Trip Report
author: Jan Wilmans
post_excerpt: ""
layout: post
permalink: >
  http://nullptr.nl/2019/09/cppcon-2019-trip-report/
published: true
post_date: 2019-09-23 10:44:42
---
[<img src="http://nullptr.nl/wp-content/uploads/2019/09/cppcon2019.png" alt="" width="1760" height="252" class="alignnone size-full wp-image-1979" />][1]

[ @ ][2][CppCon][1] 2019 @ Aurora (near Denver) Colorado

Just returned from one wholly awesome week of [CppCon][3]. This year the conference was held at The Gaylord Rockies Convention Center in Aurora Colorado, just outside Denver. This place is huge, take a look:

[<img src="http://nullptr.nl/wp-content/uploads/2019/09/gaylord-hotel.png" alt="" width="1431" height="349" class="alignnone size-full wp-image-1977" />][2]

<img src="http://nullptr.nl/wp-content/uploads/2019/09/cabouse-1.png" alt="" width="2010" height="508" class="aligncenter size-full wp-image-2014" />

I arrived on Friday to meet up with my fellow [CopperSpice][4] team members, Barbara, Ansel, Tim and Peter. We had a two day hacking session working on CopperSpice and hashed out strategies how to better organize our team. We talked about our new libraries CsCrypto, CsPaint, CsDeclarative, integrations with Azure Pipelines, further development of the CMake support and a formal funeral for our Autotools support (well, the actual wording was more like, take it into the desert and shoot it, but remember to take a picture).

## Conference Opening Keynote Bjarne Stroustrup

On Monday Bjarne gave an [overview][5] of the current C++20 status, as always, it was a strait forward, well laid out [presentation][5] that I think will be very informative to anyone who has not been actively keeping up to date with the proposals in flight during the last 3 years.

## Opening for the ‘Back to Basics’ track: Jason Turner, The Best Parts of C++

Jason is always energetic and entertaining to watch; this talk was no exception. I ‘Back to Basics’ track maybe specifically targeted at the newcomers C++, but I think many of us, also the professionals that have been in C++ years, learned something. C++ is evolving quickly and sometimes feels like a completely new language. Jason presented a good mix of time trailed good practices and fresh new insights.

## Latest & Greatest in Visual Studio 2019 for C++, Marian Luparu and Simon Brand

Whether you are a fan or not, its good to see that Visual Studio keeps up with the times and brings us exciting features with every release. However in recent years we have really seen a change in Microsoft and the the killer surprise this time was the open sourcing of the STL, the standard library that comes with visual studio!!

[<img src="http://nullptr.nl/wp-content/uploads/2019/09/stl-github.png" alt="" width="433" height="42" class="alignnone size-full wp-image-1983" />][6]

## Hello World from scratch by Peter Bindels and Simon Brand

A wild ride, energetically presented by Peter and Sy, they explained everything that happens from sourcecode, compilation, linking, loading, until the actual execution to your main() function. I enjoyed this and learned a thing or two along the way.

## Copperspice CsPaint: High performance graphics and text rendering on the GPU for any C++ application

Barbara and Ansel presented the latest development that spun out of the CopperSpice development: [CsPaint][7] a portable high performance graphics library on top of [Vulcan][8]. One of the most notable features was the infinite accuracy text rendering! A unique feature that will be used to render portable and uniform high-DPI 2D UI graphics for CopperSpice on Linux, Windows and MacOS.

[<img src="http://nullptr.nl/wp-content/uploads/2019/09/tote-1-300x230.png" alt="" width="300" height="230" class="alignnone size-medium wp-image-1993" />][4]

## Sean Parent - Better Code: Relationships, Essential Relationships

Seans [presentation][9] was about the relationships in our code, about thinking ahead about what your program will look like, comparing it with a game of chess, although I feel that usually my computer is not actively trying out moves against me :) I enjoyed the talk and I think a good takeaway was to think about the essential relationships between types and software designs in general. I heard someone say 'don't use raw pointers' was also a take-away, I think that is a little to much simplification and I would generalize that into: think/don't forget about the null-cases in the results of your functions. Slides and more available soon on [his site][10].

## Andrei Alexandrescu - Speed Is Found In The Minds of People

Andrei is sort of a unguided projectile in [this talk][11] about sorting while optimizing for medium sized datasets. [<img src="http://nullptr.nl/wp-content/uploads/2019/09/always-inf.png" alt="" width="600" height="286" class="alignnone size-full wp-image-2006" />][11]

## Jason turners talk on - Code Smells

This again was a good talk by Jason, focusing specifically on the red flags that should be investigated when reviewing code.

## SG14 Study Group Meeting

The Wednesday I spend in SG14, the study group for Game Development and Low Latency. Among other things the BLAS (Basic Linear Algebra Subprograms) integration proposals where reviewed.

## Lightning talks

## Emery Berger: compacting memory by meshing

Really interesting talk on compacting memory using meshing and virtual memory re-mapping. Emory demonstrated an 17% reduction in memory usage in a web browser. The really fun part of this being that this could be done without having the source code nor recompilation.

## Matt Godbolt - Ray Casting in threefold / Object Oriented, Functional, data oriented

Matt shows three different ways to implement a path-tracer / ray casting example, analyzing the plus/minuses for the different strategies. A surprising result for me was that in this case the branch prediction was dominating the implementation style.

## Louis Dion - ABI’s for dummies

Also recommended to watch on YouTube if you care about ABI stability. Very nice talk. What surprised me was the `Foo() = delete;` can introduce a breaking ABI change!

## J.F. Bastian - Deprecation of volatile

This was a really good talk on the history and future of volatile. Recommended talk to watch when it goes online.

## Tony Van Eerd - Objects vs Values

I was mostly asleep during this talk ;) this was not due to Tony's presentation, I will have to watch the talk again when it becomes available on youtube.

## Timur Doumler - C++20 the small things

Timur gave an overview of all the little fixes and smaller features that were put into C++20 at the last SG21 meeting in Cologne.

## The Hallway track with Walker Brown, Sean Parent, Stephan T. Lavavej, Lious Dion, Lisa Lippincot, Timur Doumler and so many others, sorry that I cannot mention everybody in this title :)

This years so-called 'hallway track', the conversations you have with people outside the talks were, for me personally, the highlight of the conference. I talked to so many interesting people, I was truly inspired by the enthusiasm and openness of literally everybody I approached. I felt like a was literally radiating with pleasure walking that halls of the conference. Thank you everybody!! I hope to see all of you next year!

While writing this post I noticed [Matt][12] has also done a [trip report][13], so check that one out too! CppCon publishes the slides for all presentations at <https://github.com/CppCon/CppCon2019>

 [1]: https://cppcon.org/timeline2019/
 [2]: https://www.marriott.com/hotels/travel/dengr-gaylord-rockies-resort-and-convention-center/
 [3]: https://www.youtube.com/watch?v=DVoAdR52sq4
 [4]: https://www.copperspice.com/
 [5]: https://www.youtube.com/watch?v=u_ij0YNkFUs&t=4s
 [6]: https://github.com/microsoft/STL
 [7]: https://github.com/copperspice/cs_paint
 [8]: https://www.khronos.org/vulkan/
 [9]: https://www.youtube.com/watch?v=ejF6qqohp3M&t=283s
 [10]: https://sean-parent.stlab.cc/
 [11]: https://www.youtube.com/watch?v=FJJTYQYB1JQ
 [12]: https://godbolt.org/
 [13]: https://xania.org/201909/cppcon-2019-trip-report