---
ID: 1849
post_title: 'Review: Deleaker, C++ memory leak detection'
author: Jan Wilmans
post_excerpt: ""
layout: post
permalink: >
  http://nullptr.nl/2019/07/review-deleaker/
published: true
post_date: 2019-07-23 20:34:40
---
# A software review?

This is the first review I've even written as a blog post on my nullptr.nl website. Why start now? I considered whether I wanted to associate myself with any particular software and never really felt the need. However, in this case I was asked to review [Deleaker][1] by the author Artem Razin (Softanics). For reference, see [@deleaker][2] on Twitter or the [website][1]. Artem asked me nicely and offered me a license for the product in return and I was actually curious to try a memory leak detection program.

## What is deleaker?

According the website deleaker is a 'Profiler for C++, C#, .NET AND DELPHI'. Let me be clear up front, I have only looked at it from the C++ perspective and particularly only used it as a Visual Studio plugin.

Some features:

*   integrates with Visual Studio (I tried VS2019 v16.1.6, the latest release as of July 2019) 
*   finds different kinds of leaks: memory, GDI, handles and others
*   profiles unmanaged and .Net code
*   supports both 32-bit and 64-bit applications
*   has report and export features

## Impressions

Installation was really easy, just year regular next, next, finish wizard with no special settings needed. I did experience an little more delay during the installation than I consider 'normal', but after waiting a extra minute or two, everything was installed correctly.

First thing I noticed was, that if you try Deleaker without activation of a license, it works fine, it tell you *if* and *what* you problem leaks. However, it does not tell you *where* you leaked it. This means, you can try it before buying, to see what and how much it will find, but to actually start solving issues, you need to pay for a license.

After entering the license, you can full source-code references and tells you not only what you leaked, but also where you allocated it.

# Debugview++ experiment

To take Deleaker for a test-spin I took a fresh clone of [DebugView++][3], one of my own open source projects and fired it up. I'm a consistent RAII user and standard library container user, and I felt pretty confident that, without ever actually measuring it explicitly, I did not have any (serious) memory leaks in this application.

I started the program in debug mode, played around with it and closed it. And... to my surprise Deleaker found I a couple of leaks! Three GDI handle leaks! This was of course completely unacceptable and had to be disproven immediately! After investigation its turned out the GDI handles were indeed leaked and Deleaker reported correctly.

Here is the innocuous looking code:

    int CMainFrame::LogFontSizeFromPointSize(int fontSize)
    {
        return -MulDiv(fontSize, GetDeviceCaps(GetDC(), LOGPIXELSY), 72);
    }
    

After reading [GetDC msdn page][4] it turns out the handle you get from GetDC() must released using [ReleaseDC()][5]. And, I was doing this correctly in almost all cases, except here. In [this commit][6] you can see the change I made to solve the leaks. I just used the correct Raii wrapper object to guard the handle's scope and the leak was solved.

# random artificial leak experiment

To see if Deleaker would also detect 'normal' memory leaks, I changed one of my std::make_unique<> calls into a (*cough*) 'new' call. Re-ran the program, closed it, and bingo: Deleaker found it immediately.

# Conclusion

This is hardly a thorough review of all Deleaker capabilities, but it conclusion I must say I'm impressed. The learning curve to use the tool is extremely low, the interface is clear and the detection seems to work correctly.

 [1]: https://www.deleaker.com/
 [2]: https://twitter.com/deleaker
 [3]: https://github.com/CobaltFusion/DebugViewPP
 [4]: https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getdc
 [5]: https://docs.microsoft.com/en-gb/windows/win32/api/winuser/nf-winuser-releasedc
 [6]: https://github.com/CobaltFusion/DebugViewPP/commit/a2357d2eee8780946420a6baecacc8b2b1d77a15