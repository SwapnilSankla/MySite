---
title: "Lexical Scoping in Swift"
date: 2017-12-03T21:08:05+05:30
tags: [
    "Swift",
    "iOS",
    "Lexical scoping",
]
---

<p align= "center">
    <img src="/images/LexicalScoping_Banner.png" width="90%">
    <p align= "center"> 
    <a href="https://swift.org/">Image source</a>
    </p>
</p>

While going through lexical scoping concept in <a href="https://en.wikipedia.org/wiki/Lisp_%28programming_language%29">LISP</a>; I wondered whether such support is available in Swift or not. And it does. Yay…

I took a simple function — ‘square root of a number' to try out the lexical scoping. I used <a href="https://en.wikipedia.org/wiki/Newton%27s_method">Newton's method</a> for calculating square roots. Below is the first version.

<p><i><font size="3" color="purple">sqRt is a recursive function. The code terminates when the guess made in earlier step is good enough. Else guess is improved and function is called again.</font></i></p>

{{< highlight swift "linenos=table">}}
func square(_ x: Double) -> Double {
    return x * x
}
func avg(_ x: Double, _ y: Double) -> Double {
    return (x + y) / 2
}

func goodEnough(x: Double, guess: Double) -> Bool {
    return abs(square(guess) - x) < 0.001
}

func improveGuess(x: Double, guess: Double) -> Double {
    return avg((x / guess), guess)
}

func sqRt(x: Double, guess: Double) -> Double {
    if goodEnough(x: x, guess: guess) {
        return guess
    }
    return sqRt(x: x, guess: improveGuess(x: x, guess: guess))
}

sqRt(x: 2, guess: 1)
sqRt(x: 9, guess: 1)
sqRt(x: 16, guess: 2)
{{< / highlight >}}

But then functions like <i>goodEnough</i> and <i>improveGuess</i> does not make any sense outside squareRoot function scope. Thankfully in Swift, functions are <i>first class citizens</i>. We can declare function just like declaring variable. This brings me to the second version of sqRt function.

{{< highlight swift "linenos=table">}}
func sqRt(x: Double, guess: Double) -> Double {

    let square = { (x: Double) in return x * x }
    let avg = { (x: Double, y: Double) in return (x + y) / 2 }

    func goodEnough(x: Double, guess: Double) -> Bool {
        return abs(square(guess) - x) < 0.001
    }

    func improveGuess(x: Double, guess: Double) -> Double {
        return avg((x / guess), guess)
    }

    if goodEnough(x: x, guess: guess) {
        return guess
    }
    return sqRt(x: x, guess: improveGuess(x: x, guess: guess))
}

sqRt(x: 2, guess: 1)
sqRt(x: 9, guess: 1)
sqRt(x: 16, guess: 2)
{{< / highlight >}}

Functions like <i>improveGuess, goodEnough</i> are operating on the function parameters of <i>sqRt</i>. As these functions are already in the scope of <i>sqRt</i> function, one can argue that passing parameters to these inner functions is redundant. Which is indeed true and that’s what is addressed with lexical scoping. Parameters/ variables of the enclosing function are in scope for these functions.

A language supporting <a href="http://wiki.c2.com/?LexicalScoping">LexicalScoping</a> has following characteristics:
<ul>
<li>Functions (and procedures, etc.) can be defined within other functions</li>
<li>Such inner functions have access to the local variables defined in the enclosing scope.</li>
<li>These inner functions also have an access to the function parameters of the enclosing function.</li>
</ul>
Version 3 of the <i>sqRt</i> function by leveraging lexical scoping support is as follows.

{{< highlight swift "linenos=table">}}
func sqRt(x: Double, guess: Double) -> Double {

    let square = { () in return guess * guess }
    let avg = { (y: Double) in return (guess + y) / 2 }

    func goodEnough() -> Bool {
        return abs(square() - x) < 0.001
    }

    func improveGuess() -> Double {
        return avg(x / guess)
    }

    return goodEnough() ? guess : sqRt(x: x, guess: improveGuess())
}

sqRt(x: 2, guess: 1)
sqRt(x: 9, guess: 1)
sqRt(x: 16, guess: 2)
{{< / highlight >}}

Hey wait! Why can’t we leverage Swift’s support further to concise the code. Square, <i>goodEnough</i> and <i>improveGuess</i> look like variables. So let’s simplify those.

{{< highlight swift "linenos=table">}}
func sqRt(x: Double, guess: Double) -> Double {
    let guessSquare = guess * guess
    let guessGoodEnough = abs(guessSquare - x) < 0.001
    let avg = { (y: Double) in return (guess + y) / 2 }
    let improvedGuess = avg(x / guess)
    return guessGoodEnough ? guess : sqRt(x: x, guess: improvedGuess)
}

sqRt(x: 2, guess: 1)
sqRt(x: 9, guess: 1)
sqRt(x: 16, guess: 2)
{{< / highlight >}}

I hope you understood lexical scoping support in Swift. Let me know your thoughts on this.

This blog is originally published <a href="https://medium.com/dev-data/lexical-scoping-in-swift-ba95ab949b0c">here</a>.