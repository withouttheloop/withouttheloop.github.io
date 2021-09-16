---
title: Connascence of methods
author: Liam McLennan
date: 2012-02-10 20:32
template: article.jade
---

From wikipedia:

    Two software components are connascent if a change in one would require the other to be modified in order to maintain the overall correctness of the system.

Connascence is a useful tool for discussing different types of coupling that occur within software. 

Again from wikipedia, "connascence of name is when multiple components must agree on the name of an entity". Connascence of name one is of the list harmful and most unavoidable types of connascence because it is simple to reason about and change. While connascence of name is still connascence, and therefore bad, it is among the most amenable to refactoring. If the name of something changes then everything that refers to that thing by name must also change. This is a simple change that can be mostly statically determined and applied by tools. 

Connascence of position is present when changing the order of something necessitates a change elsewhere. An example is the order of function arguments. If the order of a function's arguments is changed then every client of that function must also change. Connascence of position is worse than connascence of name (usually) because it is less explicit and therefore more difficult to reason about and successfully change. 

From connascence of position to connascence of name
---------------------------------------

Some modern languages have slowly been migrating to connascence of name from connascence of position. As well as converting to a less harmful form of coupling this has the additional benefit of documenting the client-side of a method call. 

### Ruby

Ruby supports positional method arguments.

    def full_name(first_name, middle_name, last_name)
        "#{first_name} #{middle_name} #{last_name}"
    end

    p full_name 'Eric', 'Arthur', 'Blair'
    => "Eric Arthur Blair"

but many ruby apis prefer the use of a single hash containing the method arguments:

    def full_name(name)
        "#{name[:first]} #{name[:middle]} #{name[:last]}"
    end

    p full_name :first=>'Eric', :middle=>'Arthur', :last=>'Blair'
    => "Eric Arthur Blair"

this has the benefit of converting positional parameters to named parameters and therefore connascence of position to connascence of name.

### JavaScript

JavaScript has adopted the same convention. While positional function arguments work:

    function fullName(first, middle, last) {
        return first + ' ' + middle + ' ' + last;
    }

    fullName('Eric', 'Arthur', 'Blair');
    => "Eric Arthur Blair"

most apis favour named arguments by passing a single object containing the method arguments:

    function fullName(name) {
        return name.first + ' ' + name.middle + ' ' + name.last;
    }

    fullName({ first: 'Eric', middle: 'Arthur', last: 'Blair'});
    => "Eric Arthur Blair"

### C sharp

C# now supports named arguments, but there usage is not popular. C# and .net has a dependable history of adopting, after a delay, the technology and techniques that have become dominant on other platforms. Therefore, I predict that usage of named method arguments in C# will increase over the next few years. Interestingly, C# is the only one of the three languages discussed that has a specific language feature for named arguments and it is also the only one that does not require a change to the definition of the method to take advantage of named arguments. The decision to use or not use named arguments is entirely at the client's discretion.

    string FullName(string first, string middle, string last) {
        return String.Format("{0} {1} {2}", first, middle, last);
    }

    FullName(first: "Eric", middle: "Arthur", last: "Blair");
    => "Eric Arthur Blair"

