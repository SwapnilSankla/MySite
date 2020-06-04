---
title: "Tail Recursion in Kotlin"
date: 2020-06-04T15:39:56+05:30
tags: [
    "Kotlin",
    "Recursion",
    "Tail recursion",
    "Tail call optimization"
]
---

<p align= "center">
    <img src="/images/Recursion.jpg" width="90%">
    <p align= "center"> 
    <a href="https://www.flickr.com/photos/torley/2361164281">Image source</a>
    </p>
</p>

Let's understand how Kotlin helps you to write Tail recursive functions. Let's take an example of a simple recursive function like calculating factorial of given number. There are various ways to implement this. Let's start with procedural approach.

{{<highlight kotlin "linenos=table">}}
fun factorial(num: Int): Int {
    var result = 1
    var counter = num

    while (counter > 1) {
        result *= counter
        counter -= 1
    }
    return result
}
{{</highlight >}}

The above method uses temp variables for computation. Can we avoid them? Yes, one way is using higher order function on the collection like below.

{{<highlight kotlin>}}
fun factorial(num: Int): Int {
    return if (num <= 1) 1 else (1..num).reduce { acc, number -> acc * number }
}{{</highlight >}}

The above method does simple computation.

- Either return 1 when number is less than or equal to 1.
- Or else create a list and reduce the list by multiplying numbers in the list.

What if we don't want to use the collection and still avoid procedural code? The solution is <b><u>Recursion</u></b>.
Recursion is basically calling same method until certain condition is met. Below is the recursive version of the factorial code.

{{<highlight kotlin>}}
fun factorial(num: Int): Int {
    return if (num <= 1) 1
    else num * factorial(num - 1)
}
{{</highlight >}}

The above code is simple. Let's see how will it work if we call it with number 7

{{<highlight kotlin>}}
(factorial 7)
(7 * factorial 6)
(7 * (6 * factorial 5))
(7 * (6 * (5 * factorial 4))))
(7 * (6 * (5 * (4 * factorial 3))))
(7 * (6 * (5 * (4 * (3 * factorial 2)))))
(7 * (6 * (5 * (4 * (3 * (2 * factorial 1))))))
(7 * (6 * (5 * (4 * (3 * (2 * 1))))))
(7 * (6 * (5 * (4 * (3 * 2)))))
(7 * (6 * (5 * (4 * 6))))
(7 * (6 * (5 * 24)))
(7 * (6 * 120))
(7 * 720)
5040
{{</highlight >}}

Do you see any issue with the above execution? The problem is the process builds a chain of <u><b>deferred operations</u></b>. The execution engine needs to keep track of all the operations to be performed later on. This amount of information linearly grows with n. Bigger the number the bigger chain needs to be remembered. Eventually the stack will throw overflow exception if it cannot hold the entire chain. This type of recursion is called as <b><u>'Linear Recursion'</u></b>. Below is the snapshot of the stack trace when above is called with number 30. 

<p align= "center">
<img src="/images/LinearRecursionStackTrace.png" width="300" height="300">
</p>

Can we avoid long chain problem? Yes we can. Following code shows how.

{{<highlight kotlin>}}
fun factorial(num: Int): Int {
    fun factorial(number: Int, count: Int): Int {
        return if (count <= 1) number else return factorial(number * (count - 1), count - 1)
    }
    return if(num <= 1) 1 else factorial(num, num)
}
{{</highlight >}}

Above code combines recursive and procedural approaches. We rely on counter but it is not stored in a variable. Rather we compute counter and number before recursive call. Notice inner factorial method, there is no computation after making recursive call. Such recursive methods are known as <b><u>Tail recursive</u></b> methods. 

Let's see how will it work if we call it with number 7

{{<highlight kotlin>}}
(factorial 7)
(factorial 42 6)
(factorial 210 5)
(factorial 840 4)
(factorial 2520 3)
(factorial 5040 2)
(factorial 5040 1)
5040
{{</highlight >}}

Main difference is now the execution is <i>not deferred</i>. It is all about computing first before recursively calling the method again. Hence execution engine does not need to remember variables state at every method call. However still the chain needs to be remembered. Meaning we have partly sovled the problem. Let's explore <b><u>Tail call optimization</u></b> to solve this further.

<h2>Tail call optimization</h2>
As per https://wiki.c2.com/?TailCallOptimization

<div style="color:dark-grey;">
<h4><b>Tail-call optimization (or tail-call merging or tail-call elimination) is a generalization of TailRecursion: If the last thing a routine does before it returns is call another routine, rather than doing a jump-and-add-stack-frame immediately followed by a pop-stack-frame-and-return-to-caller, it should be safe to simply jump to the start of the second routine, letting it re-use the first routine's stack frame (environment).</b></h4>
</div>

In simple words, smart execution engines detects the tail recursive methods and convert them to iterative code. This solves the chaining problem and also developers can enjoy writing recursive methods which are simple to understand. However <u>JVM does not natively support tail call optimization.</u> Don't worry! Kotlin compiler does it for us. Mark tail recursive method with <b><u>tailrec</u></b> annotation and rest is done by the compiler.

Below is the sample.

{{<highlight kotlin>}}
fun factorial(num: Int) = if(num <= 1) 1 else factorial(num, num)

private tailrec fun factorial(number1: Int, count: Int): Int {
    return if (count <= 1) number1 else return factorial(number1 * (count - 1), count - 1)
}
{{</highlight>}}

What if you try to apply tailrec on a method which is not tail recursive? Don't worry, compiler detects it. Below is how Intellij Idea shows it.

<p align= "center">
<img src="/images/TailrecOnNonTailRecursieMethod.png">
</p>

Can we see what changes Kotlin compiler does for supporting tail call optimization? Yes. Intellij Idea has provision to decompile kotlin bytecode. One can get the java equivalent of the Kotlin bytecode. Below is the snapshot.

<p align= "center">
<img src="/images/DecompiledTailRecursive.png">
</p>

Hope this blog helps you in understanding various recursion constructs and it's support in Kotlin. Please share if you enjoyed it.