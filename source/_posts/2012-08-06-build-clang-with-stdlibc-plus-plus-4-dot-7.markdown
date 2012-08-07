---
layout: post
title: "Build Clang with stdlibc++ 4.7 in Mac OS"
date: 2012-08-06 21:39
comments: true
sharing: true
footer: true
categories: [C++, Clang, GCC] 
---

Claim: I am no where close to a C++ expert, but just trying to learn things as life goes along. This post is for newbies like me trying to make clang work in C++11 mode with libstdc++.

## Motivation

* C++11 has been out for a while now and I really want to get my hands on it to try it out.
* Clang is a great Compiler (witht best-in-class quality and a lot of features). 
* The thing that I really enjoy is the AST that you can literally travese the source code like a "tree". There are quite a few plugings that build on top of it
* Vim Clang-Complete - quite cool, a really compiler generated complete, alot better than the ctags.
* Syntax Highlight - i haven't really tried that in Vim but looks possible and definitely interesting.

However, in order to use clang with C++11, i either have to use libc++ or rebuild clang with libstdc++4.6/7. This is kinda pain for a newbie like me. Things that I have tried that didn't work out:

* MacPort GCC4.7 - install with sudo port install gcc47
* This only give you the gcc toolchains, not the llvm/clang.
* MacPort Clang 3.2 - installs clang/llvm without gcc toolchains, that means I am back to the square one.
* The post <http://clang.llvm.org/get_started.html> is bit advanced for newbie (at that time, i didn't even know what is a GCC installation directory is. If you don't know like me, keep reading).
* And someone has similar experience <http://lists.cs.uiuc.edu/pipermail/llvmdev/2012-April/048824.html>. 

Unforutunately that post is for ubuntu, not Mac OS. Can't just copy/paste! Oh yeah, forgot. **--gcc-tool-chain** configure flag doesn't work for by the way. Maybe I should send the problem to cfe?
* After spending sometime trial and error, finally I am able to compile a C++11 code with clang with libstdc++ 4.7. I am not sure if this way is correct though, or if there is a better way? if someone knows, please comment below.

## Preparetion
* Make sure Xcode commandline tools is installed. We need it to build gcc

## Build GCC 4.7
* Download GCC 4.7 from svn. (To keep things easy and concrete, I'll give example path as i.e $HOME/Repo/gcc47)
```
$ mkdir $HOME/Repo/buildgcc47
$ cd $HOME/Repo/buildgcc47
$ ../gcc47/configure
$ make -j 8
$ sudo make DESTDIR=/usr/local install
```

You need to watch for errors. How to solve error and successfully build gcc 4.7 is beyond the scope of this post, so you'll have to figure it out. There are currently some dependency for gcc to build, namely mpr, gmp, and mpc?? ( i don't remember the last one )
The "make -j 8" is to build gcc with 8 jobs (I have 4 cores with hyperthread, you can choose a number instead of **8** based on your computer configuration)
The last step will install your newly built gcc to /usr/local

# Build Clang
* Download clang 3.2 from svn (i.e. $HOME/Repo/llvm)
* **READ TO COMPLETE BEFORE TRYING** build clang according to this guide [build clang page](url "http://clang.llvm.org/get_started.html#build") will not give me correct libstdc++ 4.7 and header search. So read next bullet point. But you need to check out clang and compiler-rt as stated in the web.
* So, before configure or build, modify the InitHeaderSearch.cpp in llvm/tools/clang/lib/Frontend/InitHeaderSearch.cpp. Find the FIXME and add the following around line 354:
<D-c>{% codeblock [InitHeaderSearch.cpp] [lang:c++] %}
    case llvm::Triple::x86:
    case llvm::Triple::x86_64:
      AddGnuCPlusPlusIncludePaths("/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/../../../../include/c++/4.7.1", "", "", "", triple);
      AddGnuCPlusPlusIncludePaths("/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/../../../../include/c++/4.7.1/x86_64-apple-darwin12.0.0", "", "", "", triple);
      AddGnuCPlusPlusIncludePaths("/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/../../../../include/c++/4.7.1/backward", "", "", "", triple);
      AddGnuCPlusPlusIncludePaths("/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/include", "", "", "", triple);
      AddGnuCPlusPlusIncludePaths("/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/include-fixed", "", "", "", triple);
      AddGnuCPlusPlusIncludePaths("/usr/local/include", "", "","", triple);
      AddGnuCPlusPlusIncludePaths("/usr/include", "", "", "", triple);
      AddGnuCPlusPlusIncludePaths("/System/Library/Frameworks", "","","",triple);
      AddGnuCPlusPlusIncludePaths("/Library/Frameworks", "", "", "", triple);

      /*
      AddGnuCPlusPlusIncludePaths("/usr/include/c++/4.7.1",
                                  "i686-apple-darwin12", "", "x86_64", triple);
      AddGnuCPlusPlusIncludePaths("/usr/include/c++/4.7.1",
                                  "i686-apple-darwin12", "", "", triple);
      */
      break;
{% endcodeblock %}
* now configure and build your clang
* I installed it to /usr/local again (optional)

# Set the Linked library in bash environment
* after the compilation, if you use the newly clang
{% codeblock [My Clang Include Path] [lang:bash]%}
clang version 3.2 (trunk 161295)
Target: x86_64-apple-darwin12.0.0
Thread model: posix
 "/usr/local/bin/clang" -cc1 -triple x86_64-apple-macosx10.8.0 -fsyntax-only -disable-free -main-file-name null -pic-level 2 -mdisable-fp-elim -masm-verbose -munwind-tables -target-cpu core2 -target-linker-version 134.7 -v -resource-dir /usr/local/bin/../lib/clang/3.2 -fmodule-cache-path /var/folders/1l/m5km38_d3lxbrvysgv6s42680000gn/T/clang-module-cache -fdeprecated-macro -fdebug-compilation-dir /Users/chen/Repository/llvm -ferror-limit 19 -fmessage-length 122 -stack-protector 1 -mstackrealign -fblocks -fobjc-runtime=macosx-10.8.0 -fobjc-dispatch-method=mixed -fobjc-default-synthesize-properties -fcxx-exceptions -fexceptions -fdiagnostics-show-option -fcolor-diagnostics -x c++ /dev/null
clang -cc1 version 3.2 based upon LLVM 3.2svn default target x86_64-apple-darwin12.0.0
ignoring nonexistent directory "/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/../../../../include/c++/4.7.1/x86_64-apple-darwin12.0.0/backward"
ignoring nonexistent directory "/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/../../../../include/c++/4.7.1/backward/backward"
ignoring nonexistent directory "/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/include/backward"
ignoring nonexistent directory "/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/include-fixed/backward"
ignoring nonexistent directory "/usr/local/include/backward"
ignoring nonexistent directory "/usr/include/backward"
ignoring nonexistent directory "/System/Library/Frameworks/backward"
ignoring nonexistent directory "/Library/Frameworks/backward"
ignoring duplicate directory "/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/../../../../include/c++/4.7.1"
ignoring duplicate directory "/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/../../../../include/c++/4.7.1/x86_64-apple-darwin12.0.0"
ignoring duplicate directory "/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/../../../../include/c++/4.7.1/backward"
ignoring duplicate directory "/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/../../../../include/c++/4.7.1/backward"
ignoring duplicate directory "/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/include"
ignoring duplicate directory "/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/include-fixed"
ignoring duplicate directory "/usr/local/include"
ignoring duplicate directory "/usr/include"
ignoring duplicate directory "/System/Library/Frameworks"
ignoring duplicate directory "/Library/Frameworks"
ignoring duplicate directory "/usr/local/include"
ignoring duplicate directory "/usr/include"
#include "..." search starts here:
#include <...> search starts here:
 /usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/../../../../include/c++/4.7.1
 /usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/../../../../include/c++/4.7.1/backward
 /usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/../../../../include/c++/4.7.1/x86_64-apple-darwin12.0.0
 /usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/include
 /usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/include-fixed
 /usr/local/include
 /usr/include
 /System/Library/Frameworks
 /Library/Frameworks
 /usr/local/bin/../lib/clang/3.2/include
 /System/Library/Frameworks (framework directory)
 /Library/Frameworks (framework directory)
End of search list.
{% endcodeblock %}
* if your header search path is different, go back and check to see where you did wrong
* Now, if you try to compile, the linker will still linked to libstdc++ 4.2 that comes with Mac OS. To overcome that, i modify the bash\_profile

```
export PATH=/usr/local/bin:/usr/local/sbin:$PATH
export LIBRARY_PATH=/usr/local/lib:/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1:/usr/local/bin/../lib/gcc:/usr/local/bin/../lib/gcc/x86_64-apple-darwin12.0.0/4.7.1/../../../:/lib:/usr/lib
```
* Now, quite your terminal and restart. It should use the newly compiled clang with gcc 4.7 libstdc++
# End Words
* Now you can try in your main.cpp. It should just compile
```
#include <memory>
#include <tuple>
```
* One last thing, I mean, I shouldn't have to do all these just to get clang compiled with libstdc++. It allows me to set --gcc-tool-chain. That option supposed to work. Maybe it's already working, i just need to modify the bash environment variables. I don't know, i didn't give it a try. If someone did try, or has a better way to do this, please post comments below
