---
title: Programmers Should Never Trust Anyone, Not Even Themselves
id: 407668f9-ad93-4150-a4c0-2bfafe038abd
createTimestamp: 2026-02-17T19:00:00+08:00
updateTimestamp: 2026-02-17T19:00:00+08:00
sortTimestamp: 2026-02-17T19:00:00+08:00
tags:
  - hidden

original-layout: post
original-title:  "Programmers Should Never Trust Anyone, Not Even Themselves"
original-date:   2024-06-19 12:00:00 -0500
original-categories: jekyll update
---

Programmers should be paranoid.

* "I double checked the code"
* "The code passes the tests"
* "The reviewer approved my code"

"So is my code correct?"

Writing code correctly is hard and verifying code correctness is impossible.
Here are some reasons why:
* **Generality**: Even if your code behaves correctly once, will it do so for
  all cases, for all machines, for all times?
* **False Pass**: Failing tests indicate the presence of bugs, but passing
  tests do not promise their absence.
* **Lack of certainty**: You could write a formal proof for your code's
  correctness but now you must wonder if the proof is correct. You would need to
  prove the proof. This chain of verifying the verification would
  never end.

It is folly to pursue certainty of your code's correctness. A bug may be hiding
in a dependency that you'll never find. Yet we should not despair. We can still
decrease the risk of bugs via greater understanding and due diligence.

# Abstractions

What is "greater understanding"? Let's focus on one facet of understanding which
comes up frequently among programmers: _abstractions_.

Abstractions are...
* mental models of how stuff works
* when we treat `thing A` **as if** it were `thing B`
* metaphorically...
  - the result of the data compression that occurs in your brain
  - seeing the forest for the trees
* used all the time in day-to-day life


> The word "abstraction" has many meanings. In programming, it can also refer
> to layers of code that hide complexity. This post will only be talking about
> abstractions in the cognitive sense.

Examples of abstraction:
* We treat our bank deposits **as if** the bank simply stores that money for us.
  - In reality, the bank does not just store the money we deposit. It loans
    away/invests most of the money that people deposit. Our money does _not_
    sit idle in a large pile in a vault.
  - The abstraction works because banks still keep enough cash on hand to
    handle most withdrawals.
* We treat time **as if** it passes at the same rate for everyone.
  - Time dilation slightly changes each person/object's flow of time based on
    their speed and how much gravity they're under.
  - GPS satellites orbiting the Earth have to adjust their clocks by ~38
    microseconds per day to account for time dilation
    ([Source](https://pilotswhoaskwhy.com/2021/03/14/gnss-vs-time-dilation-what-the/)).
  - The abstraction works because the effect of time dilation is too miniscule
    to notice unless you are doing extremely precise engineering.

---

One way to form abstractions is by removing details (creating a simplified view
of something complex). For example, most people that drive do not know much about
the inner workings of their car. Their perspective of the car can be boiled down
to:
* Ignition turns on car
* Accelerator makes car go
* Brake makes car stop
* Wheel turns car
* Car needs gas/diesel

Knowing the above abstraction makes it unnecessary to understand the inner
workings of car engines. Most drivers have only this working knowledge of cars
and they can drive where they need to go.

When we use a programming language, it provides abstractions that allow us to
operate computers without understanding their inner workings.
* **Basic language features** (such as loops, if-conditionals, functions,
  statements, and expressions) are all abstractions which hide:
  - Hardware-level details: CPU instructions, registers, flags, and details
    specific to CPU architecture, ...
  - OS-level details: call stack management, memory management, ...
* **Portability**: Languages abstract away the need to concern ourselves with the
  differences between different machines.
  - Any compiled Java program (eg, a jar file) _should be_ able to run on any
    machine that has a Java Runtime Environment (ie, JVM) on it.
  - A Python script _should be_ able to run on any machine with a Python
    interpreter.
  - A C program _should be_ able to compile and run on any machine if the machine
    has a C compiler.

# Abstractions fail

Unfortunately, abstractions fail.

* Language abstractions are not enough if you care about code performance. To
  speed up your code, you need to know hardware-level and OS-level details.
* Porting programs that have external dependencies such as dynamic libraries or
  network requirements is not as simple. They cannot simply be moved to
  another machine and run. Extra setup and knowledge are required.
* Car owners who only know the bare minimum can end up stuck in a broken down
  car. If the driver doesn't regularly change their car's lubricant/oil, they'll
  shorten the engine's lifespan.

The driver's abstraction works well in the short term (for a single drive), but
fails in the long term (many years). Joel Spolsky describes such failing
abstractions as "leaky" and put forth the [Law of Leaky Abstractions](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/):

> All non-trivial abstractions, to some degree, are leaky.

Which is similar to the maxim from statistics:

> All models are wrong, but some are useful.

When we write code, we use leaky abstractions all the time. Here are some random
examples:
* Garbage collection takes away the burden of worrying about memory management
  (unless we care about latency jitters)
* C++ smart pointers make memory safe (as long as you don't store any raw
  pointers from it)
* Hashtables are fast because they have O(1) operations (but arrays are faster
  for smaller sizes).
* Passing by reference is faster than passing by value (except for cases of copy
  elision and values which fit in CPU registers like ints)

Fortunately, many leaky abstractions crash your code when they fail, so they're
easy to address. However, some may just produce undefined behavior or
performance degradation which are harder to identify and fix.

# Press X to doubt

So if abstractions can be problematic, then should we try to understand a topic
without abstractions (to know cars as they _really_ are)? No. When you dig
beneath abstractions, you just find more abstractions. It's turtles all the way
down.

* Underpinning our abstraction of cars, there is an understanding of each
  component's purpose.
* Beneath that, the chemistry of combustion and the mechanical engineering of
  the engine
* Beneath that, the mathematics/physics that model the forces of our universe

These layers of abstractions go down until we hit our most basic axioms about
logic and reality.

As programmers, we should see our knowledge as a house of cards made of leaky
abstractions and assumptions. We should have a healthy amount of skepticism of
everything and everyone, including ourselves.

# Trust, but verify

A programmer should have a "trust, but verify" policy.

Here are some examples:
* Trust the information that people tell you, but verify it with what the
  documentation says
* Test your beliefs by trying to disprove them.
  - You wrote tests for your code change and they pass on the first try. Try
    running the tests without your changes and see if they still pass. They
    may have a bug that makes them always pass.
  - You've refactored the code which should be a no-op. All the tests still
    pass. Check to make sure there actually are any tests that run the code
    you refactored.
  - You optimized your service and you see the expected reduction in resource
    utilization. Check to make sure your service is not just handling fewer
    requests at the moment.
  - You've submitted a code change and you see no problems in the service the
    next day. Check to make sure a rollout occurred that day and your code was
    included in it.
* Always measure impact when optimizing code. Code changes that appear to be
  "theoretically" faster can end up being slower due to factors revealed in
  lower layers of abstraction.

# Beware of unknown unknowns

The scariest epistemological issue for programmers is the "unknown unknown".

There are...
* things you know (ie, "knowns")
* things you know you don't know ("known unknowns")
* things you don't even know that you don't know ("unknown unknowns")

These unknown unknowns are the root of abstraction failures (and the reason why
programmers can never accurately predict how long a project will take).

You may have never heard of...
* Sanitizing user inputs
  - If you use user-provided strings as part of a SQL query, your service can
    be hacked via SQL injections.
* Character encodings
  - Any text data your code processes must be using the character encoding
    (eg, ASCII, UTF-8, UTF-32, etc) your code expects/supports.
  - Random access of a character in a text buffer could take constant
    time (for ASCII) or linear time (for UTF-8) depending on the character
    encoding.
  - You may output incomprehensible characters if you try to read text data
    using the wrong character encoding.
* Java heap size
  - Your program may be slowed down due to a lack of heap memory.
  - You could fix this issue if you knew to configure a larger max heap size
    for your Java program.

If you have not heard of these topics before, you may not even know that you
fell into their pitfalls.

There is no sure fire way to catch unknown unknowns when they are around but we
should check under at least one layer of abstraction to look for them.
Especially when a project requires learning something new, you should always
learn more than you need to. Doing so will mitigate the risk of being
surprised by abstraction failures.

When learning/working with an unfamiliar platform/language/tool/library/technology:
* Read more documentation than just the bare minimum you need
* Watch videos
  - Conference presentations have the highest quality in my experience
* Read blog posts
* Read the source code
* Grow your understanding of abstractions you have to work with
  - Learn about features recently added in to your programming language
  - Read through all of the public functions of libraries, not just the ones
    you're using
  - Skim through all of the flags of a CLI tool's man page
* Learn at least one layer of abstraction lower than you need
  - Learn about your compiler's optimizations
  - If you're running a service, learn about your orchestration platform (ex:
    kubernetes)
  - If you're working in Java, learn about the JVM
  - If you're working in Python, learn about the Python interpreter.

# Conclusion

Abstractions are necessary as they allow us to think efficiently but they are
treacherous as they can make it appear that we know "enough". Programmers who
learn superficially will not succeed on difficult projects that come without
known solutions and involve multiple domains of expertise.

That said, the ideal put forth by this blog post needs to be balanced against
reality. Obviously, we can't take time to learn every little thing when we're
in a rush. Also, beginners can't be expected to be so thorough. Ideals should be
balanced against real-world considerations.

And real-world considerations should be balanced against ideals. We should be
willing to pay some short-term costs for thorough learning and verifications.
Not just for writing correct code, but as part of our long-term journey of
becoming capable and principled engineers.
