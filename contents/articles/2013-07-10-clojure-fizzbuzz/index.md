---
title: Clojure FizzBuzz
author: Liam McLennan
date: 2013-07-10 20:32
template: article.jade
---

Here is my clojure fizzbuzz:

    (defn fizzbuzz []
      (let [divisibleBy3 (fn [x]
                           (= 0 (mod x 3)))
            divisibleBy5 (fn [x]
                           (= 0 (mod x 5)))
            calcz (fn [i]
                   (cond
                     (and (divisibleBy3 i) (divisibleBy5 i)) "FizzBuzz"
                     (divisibleBy5 i) "Buzz"
                     (divisibleBy3 i) "Fizz"
                     :else i))
            ]
         (map calcz (range 1 101))))
    
    (fizzbuzz)

