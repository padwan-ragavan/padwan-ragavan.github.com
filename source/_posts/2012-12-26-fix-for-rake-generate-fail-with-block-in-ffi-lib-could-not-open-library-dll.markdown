---
layout: post
title: "fix for rake generate with sytax highlight code block - fail with block in ffi_lib: Could not open library .dll"
date: 2012-12-26 01:17
comments: true
categories: 
---

When `rake generate` fails with  
```
	block in ffi_lib: Could not open library .dll
``` 

for me it was because the rubypython was looking for python27.dll in `{instal.dir}\Python27\libs\python27.dll` and it was not there. This can be fixed by copying the `python27.dll` from `C:\Windows\System32\python27.dll` if yours is a 32-bit or from `C:\Windows\SysWOW64\python27.dll` if 64-bit.

got to know about the missing dll from [Hussion's Blog](http://hussion.me/blog/2012/04/29/octopress2/), but mine was 64-bit and ofcourse copying from system32 didnt solve the issue.