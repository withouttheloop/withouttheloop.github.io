---
title: F# Project Loading
author: Liam McLennan
date: 2015-01-16 20:32
template: article.jade
---

Everytime I create a new F# project (typically using the MVC 5 F# Templates) things go great until I close Visual Studio. When I try to reopen the solution I inevitably get:

> The project ?? could not be opened because opening it would cause a folder to be rendered multiple times in the solution explorer. One such problematic item is ??

It's natural to think that the indicated file is causing the problem, but if you remove it then the problem just shifts to the next file. 

The error occurs when files within the project are not grouped by their directory. A project containing:

    <Compile Include="Home\HomeController.fs" />    
    <Content Include="Scripts\react.js" />
    <None Include="Home\Index.cshtml" />
    <Content Include="Scripts\react.d.ts" />

cannot be reopened. To fix, group the files by their directory.

    <Compile Include="Home\HomeController.fs" />    
    <None Include="Home\Index.cshtml" />
    <Content Include="Scripts\react.js" />
    <Content Include="Scripts\react.d.ts" />

That's it. Hope this helps someone. 