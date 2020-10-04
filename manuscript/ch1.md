# Chapter 1: Why Functional Programming?

## At a Glance

Let's briefly illustrate the notion of "Functional-Light JavaScript" with a before-and-after snapshot of code. Consider:

```js
var numbers = [4,10,0,27,42,17,15,-6,58];
var faves = [];
var magicNumber = 0;

pickFavoriteNumbers();
calculateMagicNumber();
outputMsg();                // The magic number is: 42

// ***************

function calculateMagicNumber() {
    for (let fave of faves) {
        magicNumber = magicNumber + fave;
    }
}

function pickFavoriteNumbers() {
    for (let num of numbers) {
        if (num >= 10 && num <= 20) {
            faves.push( num );
        }
    }
}

function outputMsg() {
    var msg = `The magic number is: ${magicNumber}`;
    console.log( msg );
}
```

Now consider a very different style that accomplishes exactly the same outcome:

```js
var sumOnlyFavorites = FP.compose( [
    FP.filterReducer( FP.gte( 10 ) ),
    FP.filterReducer( FP.lte( 20 ) )
] )( sum );

var printMagicNumber = FP.pipe( [
    FP.reduce( sumOnlyFavorites, 0 ),
    constructMsg,
    console.log
] );

var numbers = [4,10,0,27,42,17,15,-6,58];

printMagicNumber( numbers );        // The magic number is: 42

// ***************

function sum(x,y) { return x + y; }
function constructMsg(v) { return `The magic number is: ${v}`; }
```

Once you understand FP and Functional-Light, this is likely how you'd *read* and mentally process that second snippet:

> We're first creating a function called `sumOnlyFavorites(..)` that's a combination of three other functions. We combine two filters, one checking if a value is greater-than-or-equal to 10 and one for less-than-or-equal to 20. Then we include the `sum(..)` reducer in the transducer composition. The resulting `sumOnlyFavorites(..)` function is a reducer that checks if a value passes both filters, and if so, adds the value to an accumulator value.
>
> Then we make another function called `printMagicNumber(..)` which first reduces a list of numbers using that `sumOnlyFavorites(..)` reducer we just defined, resulting in a sum of only numbers that passed the *favorite* checks. Then `printMagicNumber(..)` pipes that final sum into `constructMsg(..)`, which creates a string value that finally goes into `console.log(..)`.

All those moving pieces *speak* to an FP developer in ways that likely seem highly unfamiliar to you right now. This book will help you *speak* that same kind of reasoning so that it's as readable to you as any other code, if not more so!

A few other quick remarks about this code comparison:

* It's likely that for many readers, the former snippet feels closer to comfortable/readable/maintainable than the latter snippet. It's entirely OK if that's the case. You're in exactly the right spot. I'm confident that if you stick it out through the whole book, and practice everything we talk about, that second snippet will eventually become a lot more natural, maybe even preferable!

* You might have done the task significantly or entirely different from either snippet presented. That's OK, too. This book won't be prescriptive in dictating that you should do something a specific way. The goal is to illustrate the pros/cons of various patterns and enable you to make those decisions. By the end of this book, how you would approach the task may fall a little closer to the second snippet than it does right now.

* It's also possible that you're already a seasoned FP developer who's scanning through the start of this book to see if it has anything useful for you to read. That second snippet certainly has some bits that are quite familiar. But I'm also betting that you thought, "Hmmm, I wouldn't have done it *that* way..." a couple of times. That's OK, and entirely reasonable.

    This is not a traditional, canonical FP book. We'll at times seem quite heretical in our approaches. We're seeking to strike a pragmatic balance between the clear undeniable benefits of FP, and the need to ship workable, maintainable JS without having to tackle a daunting mountain of math/notation/terminology. This is not *your* FP, it's "Functional-Light JavaScript".

Whatever your reasons for reading this book, welcome!


## Readability

<p align="center">
    <img src="images/fig17.png" width="50%">
</p>

*Imperative* describes the code most of us probably already write naturally; it's focused on precisely instructing the computer *how* to do something. Declarative code -- the kind we'll be learning to write, which adheres to FP principles -- is code that's more focused on describing the *what* outcome.

The first snippet is imperative, focused almost entirely on *how* to do the tasks; it's littered with `if` statements, `for` loops, temporary variables, reassignments, value mutations, function calls with side effects, and implicit data flow between functions. You certainly *can* trace through its logic to see how the numbers flow and change to the end state, but it's not at all clear or straightforward.

The second snippet is more declarative; it does away with most of those aforementioned imperative techniques. Notice there's no explicit conditionals, loops, side effects, reassignments, or mutations; instead, it employs well-known (to the FP world, anyway!) and trustable patterns like filtering, reduction, transducing, and composition. The focus shifts from low-level *how* to higher level *what* outcomes.

Instead of messing with an `if` statement to test a number, we delegate that to a well-known FP utility like `gte(..)` (greater-than-or-equal-to), and then focus on the more important task of combining that filter with another filter and a summation function.

Moreover, the flow of data through the second program is explicit:

1. A list of numbers goes into `printMagicNumber(..)`.
2. One at a time those numbers are processed by `sumOnlyFavorites(..)`, resulting in a single number total of only our favorite kinds of numbers.
3. That total is converted to a message string with `constructMsg(..)`.
4. The message string is printed to the console with `console.log(..)`.

You may still feel this approach is convoluted, and that the imperative snippet was easier to understand. You're much more accustomed to it; familiarity has a profound influence on our judgments of readability. By the end of this book, though, you will have internalized the benefits of the second snippet's declarative approach, and that familiarity will spring its readability to life.

I know asking you to believe that at this point is a leap of faith.

It takes a lot more effort, and sometimes more code, to improve its readability as I'm suggesting, and to minimize or eliminate many of the mistakes that lead to bugs. Quite honestly, when I started writing this book, I could never have written (or even fully understood!) that second snippet. As I'm now further along on my journey of learning, it's more natural and comfortable.

If you're hoping that FP refactoring, like a magic silver bullet, will quickly transform your code to be more graceful, elegant, clever, resilient, and concise -- that it will come easy in the short term -- unfortunately that's just not a realistic expectation.

FP is a very different way of thinking about how code should be structured, to make the flow of data much more obvious and to help your reader follow your thinking. It will take time. This effort is eminently worthwhile, but it can be an arduous journey.

It still often takes me multiple attempts at refactoring a snippet of imperative code into more declarative FP, before I end up with something that's clear enough for me to understand later. I've found converting to FP is a slow iterative process rather than a quick binary flip from one paradigm to another.

I also apply the "teach it later" test to every piece of code I write. After I've written a piece of code, I leave it alone for a few hours or days, then come back and try to read it with fresh eyes, and pretend as if I need to teach or explain it to someone else. Usually, it's jumbled and confusing the first few passes, so I tweak it and repeat!

I'm not trying to dampen your spirits. I really want you to hack through these weeds. I am glad I did it. I can finally start to see the curve bending upward toward improved readability. The effort has been worth it. It will be for you, too.

### Resourcees

Blogs/sites

* [Awesome FP JS](https://github.com/stoeffel/awesome-fp-js)
* [Functional Programming Jargon](https://github.com/hemanth/functional-programming-jargon#functional-programming-jargon)
* [Functional Programming Exercises](https://github.com/InceptionCode/Functional-Programming-Exercises)

popular FP libraries:

* [Ramda](http://ramdajs.com)
* [lodash/fp](https://github.com/lodash/lodash/wiki/FP-Guide)
* [functional.js](http://functionaljs.com/)
* [Immutable.js](https://github.com/facebook/immutable-js)
[Appendix C takes a deeper look at these libraries](apC.md/#stuff-to-investigate) and others.
