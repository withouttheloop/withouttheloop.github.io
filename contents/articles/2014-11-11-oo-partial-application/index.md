---
title: Object-Orientation is Just Grouping Partially Applied Functions
author: Liam McLennan
date: 2014-11-11 20:32
template: article.jade
---

I've been wondering if perhaps there is not much to this object orientation business. Could it be that an 'object' is nothing more than a collection of partially applied functions?

An Example of Object-Orientation
-----------------------

Here is a simple object oriented class, representing a person with a couple of operations.

    void Main()
    {
        var vlad = new Person("Vladimir", 62);
        var tony = new Person("Tony", 57);
                
        tony.Greet().Dump();
        vlad.OlderThan(tony).Dump();
    }

    public class Person {
        public string _name;
        public int _age;
                                    
        public Person(string name, int age) {
            _name = name;
            _age = age;
        }

        public string Greet() {
            return String.Format("Hello, my name is {0}", _name);
        }
                                                                                
        public bool OlderThan(Person other) {
            return _age > other._age;
        }
    }

The data is a person's name and age. The operations are Greeting and comparing the ages of two instances.

The output of the above Linqpad script is:

<blockquote>Hello, my name is Tony </blockquote>
<blockquote>True</blockquote>

An Example With Functions
---------------

An equivalent functional implementation is:

    type Person = {name:string;age:int}

    let greet p = p.name |> sprintf "Hello my name is %s"
    let olderThan p1 p2 = p1.age > p2.age

    let vlad = {name= "Vladimir"; age= 62}
    let tony = {name= "Tony"; age= 59}

    greet vlad
    olderThan vlad tony

output

<blockquote>
val it : string = "Hello my name is Vladimir"
val it : bool = true
</blockquote>

Objects Are Just Partially Applied Functions
------------

If I didn't care about sensible solutions I might implement a functional program that is more directly equivalent to the object-oriented code.

    type PersonOO = { greet: string; olderThan: string -> int -> bool}

    let greet' name age = name |> sprintf "Hello my name is %s" 
    let olderThan' name age name' age' = age > age'

    let constructPerson name age = { greet = greet' name age; olderThan = olderThan' name age}

    let vlad' = constructPerson "Vladimir" 62
    let tony' = constructPerson "Tony" 59

    vlad' 
    vlad'.olderThan "Tony" 59

output

    val tony' : PersonOO = {greet = "Hello my name is Tony"; olderThan = <fun:tony'@9>;}
    val it : bool = true

When you look at it this way it is easier to see the problem. In the signature of `greet'` the age argument is redundant. In the signature of `olderThan'` only the age arguments are used.  

Some Problems With Objects
-----------

Both of the previous examples do the same thing. The trouble with the object version is that both methods are coupled to the name and age fields, even though they don't need to be. The other side of the same coin is a cohesion problem. 

So both implementations are pretty much equivalent, but to me the functions are simpler and more obvious.
