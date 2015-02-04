---
layout: post
title: "Mathematics - Bizarre, Fun, Beautiful: all at the same time"
date: 2015-01-02
comments: true
---

A few years ago, I was reading a book called
["The Curious Incident Of The Dog In the Night-time"][mark-haddon]. It's
a fun little book. In that book, the lead character talks about a
problem discussed in a real world incident. That problem, sometimes
called the [Monty Hall Problem][monty-hall], is this:

> Suppose you're on a game show, and you're given the choice of three
> doors: Behind one door is a car; behind the others, goats. You pick
> a door, say No. 1, and the host, who knows what's behind the doors,
> opens another door, say No. 3, which has a goat. He then says to
> you, "Do you want to pick door No. 2?" Is it to your advantage to
> switch your choice?

My immediate response was that it shouldn't make any
difference. However, the person, [Marilyn vos Savant][marilyn], who
answered the question, said that contestant should switch to the other
door. The reason she stated was

> The first door has a 1/3 chance of winning, but the second door has
> a 2/3 chance.

Now, this seems counter-intuitive at best, and absurd at the
worst. And, it seems that I was not the only one who thought so. She
got thousands of mails, some of them from the PhDs in mathematics,
claiming, sometimes in not-so-polite words, that she was wrong. Here's
a sample of a letter from a condescending PhD:

> You made a mistake, but look at the positive side. If all those
> Ph.D.'s were wrong, the country would be in some very serious
> trouble.
> 
> <cite><small>Everett Harman, Ph.D. - U.S. Army Research Institute</small></cite>

And another one from an irate, self-righteous mathematician, also a
PhD: 

> You blew it, and you blew it big! Since you seem to have difficulty
> grasping the basic principle at work here, I'll explain. After the
> host reveals a goat, you now have a one-in-two chance of being
> correct. Whether you change your selection or not, the odds are the
> same. There is enough mathematical illiteracy in this country, and
> we don't need the world's highest IQ propagating more. Shame!
> 
> <cite><small>Scott Smith, Ph.D. - University of Florida</small></cite>

Oh, that was my reasoning too. And yes, Marilyn was once listed in the
Guinness book as the person with the highest IQ in the world. You can
go to [Marilyn's own page][marilyn-problem] documenting the whole
controversy. It might seem like an overstatement to call this a
controversy, but if you get like 1000 mails from mathematics PhDs
only, I think it counts as one, at least in the field of math.

However, it turned out that Marilyn was indeed right. She got a mail
from a PhD at MIT which went something like this:

> You are indeed correct. My colleagues at work had a ball with this
> problem, and I dare say that most of them, including me at first,
> thought you were wrong!
>
> <cite><small>Seth Kalson, Ph.D. - MIT</small></cite>

Let me try to explain why it is right with some help from Wikipedia.

One way this can be explained is that if the contestant picks a goat
initially (66.66% or 2/3 probability), he/she *will* win the car by
switching since the second door with a goat is now eliminated by the
host. The only remaining door is the one with the car. On the other
hand, if the contestant did choose the car-door initially (33.33% or
1/3), switching the door will not help. So, If you always switch,
the probability of the first scenario, 2/3, is also the probability of
you winning the car.

I don't want to get into more details here. The
[Wikipedia entry][monty-hall] contains several different kinds of
explanations, including graphical and mathematical ones.

Mathematics has a lot of such unintuitive, quirky stuff that can stump
you. Here is one more example.

## 0.9999... and 1 are the same numbers

The ... after the 0.9999 mean that the 9's continue forever.

I am *not* saying that they are equal upon rounding or that they are
practically the same. I am saying that they are different
representations of the *same number*, just like 1/2 and 0.5 are
different representations of the same number.

Really understanding, and accepting, this fact requires coming to
terms with the concepts of [infinity][infinity] and
[infinitesimals][infinitesimal], and some understanding of the real
number system. However, the basic proofs are simple enough to
understand. Here's one that can be understood by school kids:

```
x = 0.999...
10x = 9.999...  // multiple both sides by 10
10x - x = 9.999... - 0.999... // subtract x from both sides, remember x = 0.999...
9x = 9
x = 1
```

See, exactly the same. As usual, the
[Wikipedia entry][wikipedia-entry] has a list of several proofs, even
more "rigorous" ones, if you are interested. And, if you want a really
fun explanation, [here's a video][vi-hart-proof] from my favorite
recreational mathemusician.

Most people find this hard to believe; even with all those
proofs. That could be because imagining infinity is really
hard. However, even more basic reason, I think, is because most of us
weren't taught math like it is supposed to. The hardest thing I did in
trying to learn math was getting out of the school mode. We assume
that any number has only one representation as a decimal. Well, as the
[The Straight Dope says][straight-dope]:

> The lower primate in us still resists, saying: .999~ doesn't really
> represent a number, then, but a process. To find a number we have to
> halt the process, at which point the .999~ = 1 thing falls apart.
> 
> Nonsense.
> 

You may encounter more bizarre results and conclusions as you go
exploring more and more math. However, along with some mind bending
stuff, math delivers some fantastically beautiful results.

## Euler's Identity, aka the most beautiful equation in mathematics

[Leonhard Euler][euler] is like the most bad-ass mathematician to
have ever lived. No wonder he came up with what is now considered to
be [the most beautiful equation][euler-identity] in all of mathematics. 

![Euler's Identity][euler-identity-image]


I think most people will recognize the symbols in that equation, even
if they don't really understand them, or, as in the case of <var>`i`</var>,
find them unpalatable.

Even then, let me attempt to summarize why it seems so jaw-droppingly
beautiful. It takes three seemingly unrelated symbols

- <var>`e`</var>: The Euler number, the unit rate of growth, and the
  base of natural log
- <var>`π`</var>: The [irrational][irrational] and
  [transcendental][transcendental] ratio of the circumference of a
  circle to its diameter, and
- <var>`i`</var>: The [imaginary][imaginary] number, the square root of -1.

and combines them with three fundamental mathematical operations
of addition, multiplication and exponentiation, and lives to tell the
tale.

This is like some mad scientist tried to combine the genes of the
Joker, Sauron and the Galactus to create the ultimate super-villain
and instead, what came out was a super-hero with high cheekbones who
restored the peace in the universe.

This equation has been a source of a lot of research and
discussions. Mathematicians have written entire books dedicated to it,
including one that says:
[Dr. Euler's Fabulous Formula - Cures Many Mathematical Ills][euler-fab-formula].

To confess the obvious, I do not understand the importance of this
equation yet. However, that doesn't mean that I cannot admire how
unlikely and unintuitive it can seem and still be correct.

And math seems to be so full of things that look paradoxical,
unintuitive and downright incorrect; so much so that even real
mathematicians are stumped sometimes. But, that is also what makes math so
much fun, like this limerick says:

> I used to think math was no fun <br />
> 'Cause I couldn't see how it was done <br />
> Now Euler's my hero <br />
> For I now see why zero <br />
> Equals e<sup>iπ</sup>+1
> 
> <cite><small>Paul Nahin</small></cite>

Back to learning some more math.

[mark-haddon]: http://www.amazon.com/Curious-Incident-Dog-Night-time-Childrens-ebook/dp/B008PU8SR4/

[monty-hall]: http://en.wikipedia.org/wiki/Monty_Hall_problem

[marilyn]: http://en.wikipedia.org/wiki/Marilyn_vos_Savant

[marilyn-problem]: http://marilynvossavant.com/game-show-problem/

[infinity]: http://en.wikipedia.org/wiki/Infinity

[infinitesimal]: http://en.wikipedia.org/wiki/Infinitesimal

[wikipedia-entry]: http://en.wikipedia.org/wiki/0.999...

[vi-hart-proof]: https://www.youtube.com/watch?v=TINfzxSnnIE

[straight-dope]: http://www.straightdope.com/columns/read/2459/an-infinite-question-why-doesnt-999-1

[euler-identity-image]: /img/euler-identity.png "Euler's Identity"

[euler]: http://en.wikipedia.org/wiki/Leonhard_Euler

[irrational]: http://en.wikipedia.org/wiki/Irrational_number

[transcendental]: http://en.wikipedia.org/wiki/Transcendental_number

[imaginary]: http://en.wikipedia.org/wiki/Imaginary_unit

[euler-identity]: http://en.wikipedia.org/wiki/Euler%27s_identity

[euler-fab-formula]: http://www.amazon.com/Dr-Eulers-Fabulous-Formula-Mathematical-ebook/dp/B004USISL6/
