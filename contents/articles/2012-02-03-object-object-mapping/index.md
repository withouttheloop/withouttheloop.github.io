---
title: Object-object mapping
author: Liam McLennan
date: 2012-02-03 20:32
template: article.jade
---

Object-object Mapping - Why I don't use Automapper
=================================================

At the boundaries of an application it is often nice to map objects from one form to another. The most common example is mapping domain entities to view models, or mapping to dtos for network transfer. 

[Automapper](https://github.com/AutoMapper/AutoMapper/wiki/Getting-started) is a library for removing the duplication from object-object mapping code by mapping by convention. When the conventions don't map the way you want you can explicitly map properties. 

After using AutoMapper on a couple of projects I have stopped using it. The problem is that the convention based mappings introduce annoying bugs when property names change  or the object graph changes. When a change breaks the convention it results in a runtime exception. Very annoying. 

What I Do Instead
-----------------

My observations from using automapper are:

1. If you use the convention-based mapping and a property is later renamed that becomes a runtime error and a common source of annoying bugs. 
2. If you don't use convention-based mapping (ie you explicitly map each property) then you are just using automapper to do your projection, which is unnecessary complexity.

For larger projects I like to implement explicit type mappers and register them in an IoC container. The advantages are:
mappers are explicit. Easy to identify and test mappings
the mappers can have their own dependencies. The consumers of the mappers don't have to depend on the mappers dependencies.

=======================================================================================================================================


public interface IMapper<T1,T2> 
{
    T2 Map(T1 input);
}

public class MapAppleToOrange : IMapper<Apple, Orange> 
{
   public MapAppleToOrange(/* opportunity for constructor injection here. Useful for mapping VMs -> persisted models */ ISomeDependency dependency) 
   {} 
   
   Orange Map(Apple input) 
   {
       // map things here
       // make use of dependency
   }
}

// usage

public class HomeController
{
    // let the IoC container resolve the required mappers
    // note that HomeController does not depend on ISomeDependency 
    public HomeController(IMapper<Apple, Orange> mapper) 
    {}

    public ActionResult Index(Apple apple) 
    {
       // do some mapping
       return View(mapper.Map(apple));
    }
}