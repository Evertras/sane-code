# What are you optimizing for?

The best PM I've ever worked with asked this question constantly,
even in casual conversation.  It was practically a meme.  But it's
also the best question I've ever heard when it comes to starting
a conversation about tradeoffs.

Everything is a tradeoff.  Everything.

Writing code at all is a tradeoff.  Was there something off the shelf
you could've used instead?  Why are you trying to rewrite Elasticsearch?
Stahp.  ...or maybe keep going, if that off the shelf thing doesn't fully
suit your needs and you really do need something custom after all.

If you write quick and dirty code to get something out the door,
you're optimizing for shipping fast.  It hopefully at least works in the
majority of cases.  As time goes on it will be a nightmare to maintain,
to onboard new people, to track down bugs, and even looking at it will
make you want to throw various objects out of windows.  But it did ship fast!

If you take three weeks to ship a simple feature but make sure it's covered
by tests, you're optimizing for long term stability. When it ships you know
it works, you're confident in your ability to maintain it, and you would feel
proud showing it to a new hire.  But your competitor shipped two weeks ago
and has all the users, so everything you just wrote is useless.  Oof.

Neither of these is objectively better in a vacuum. So... what are you
optimizing for?

## Three examples

To me, there are three targets to optimize for that are generally at odds with
each other.  Each of these has a time and a place.  Unfortunately I've seen
many developers think of one or more of these as "bad"; they're not! They just
have to be used appropriately.

### Ship it fast

While giant functions and slapped-together APIs make babies cry, sometimes
it's what you gotta do to get something out the door fast.  Prototypes!

When you optimize for shipping it fast, you're potentially getting something
out to users faster than your competition.  Or maybe you're getting it into
someone's hands so you can get feedback earlier and make a better end result.
That's actually pretty cool!

What you're losing out on is your sanity and well-being if you have to deal
with this code for any length of time.

If the code is literally not going to be used ever again, this isn't a problem.
The problem is that the code is *always* going to be used again unless you're
willing to draw the line.

*Throw away prototype code.*  At the start of the prototype, make it very, *very*
clear to everyone involved that this code will die.  None of it will be salvaged.
It will be, at most, used as a brief reference that school children walk by
in a museum to feel a deep sense of mourning.  Maybe they'll have to write reports
about it.  Poor kids.

If this code must live in a larger project, maybe as an experimental feature,
section it off.  Fence it with comments that scream at any future developer
that here be dragons, and not to let them out of their cage.

At the same time, don't be afraid to do this when it's appropriate.  I've
seen developers frozen in paralysis because they're writing 'bad' code and
end up taking way too long to actually get something working.  They didn't
realize that they should have been optimizing for shipping fast.

### High performance

This is code that has to run *fast*.  This is assembly in-lined into C++ because
C++ wasn't fast enough.  This is meticulously crafted contiguous memory blocks
for L2 cache hits in order to efficiently process millions of operations per second.

At a C++ conference I was struck by a point that one presenter gave.  At a certain
very large tech organization, a 2% optimization to their compiler fork was done.
This number meant that builds were now 2% faster.  This saved them *millions of dollars*.
Sometimes speed really does matter!

Shaving milliseconds or even microseconds off a tight loop can be important.

But often it's not.

Is the user actually going to notice if this button click takes two more milliseconds?
Does it matter if you shaved off a few instructions to make an API call that has
to hop over the internet?

On the other hand, there was another excellent presentation that I remember mostly
for the Q&A.  The presentation itself was about improving performance in C++.  The
presentation was interesting, but I couldn't tell you what he actually talked about
there.  What stuck with me was when someone started to ask a question, which I will
butcher in a paraphrasing here.

> Question: These are great and all, but we value maintainability and don't care about performance as-

> Answer, cutting off question: That's great that you don't care about performance, but it's people not caring about performance that makes me wait 30 seconds to start Word.

Guy's got a point.  Sometimes you do need to make things a little more complicated
to make your hot spots burn quick.  And if you're not careful, your code will die
a death by a thousand cuts with every little slow piece adding up to a slog.

### Long term stability

This is code that's going to be around for a while.  Admittedly, this is where I
tend to default to, so I may be a little biased here.  I enjoy doing TDD even for
prototype code.  I will argue vehemently with you about adding tests first and
why hearing the phrase "All that's left are the tests" makes me physically froth
at the mouth.  But that'll be for another day.

Test first.  Test often.  Automate things.  Build rock solid CI pipelines.  Keep
functions short and simple.  Don't do anything 'clever'.  Simple, simple, simple.

And that's why you performance and stability can often be at odds.  It's easy when
the fast thing is also simple, like using std::sort instead of using some random
custom bubble sort someone before you left behind.  But keeping things highly
performant can come at the cost of having to be clever.

### An example

I think a great example of this is in some logging libraries for Go.  Let's play
pretend!  You're a Go developer who is starting a new project.  Stick with me.
You need... logging.  Fantasyland.

#### Ship it fast

Just use [the standard log package](https://golang.org/pkg/log/).  Easy.  Doesn't
have any fancy bells and whistles, but prints stuff to the screen and you're done.

There's more to it, though, that I think is interesting.  Using this simple package
means that other devs don't need to learn a new package.  They don't need to learn
new interfaces, quirks, function signatures, and keep that all in their head.  They
can just hit the ground running with you and print stuff to the screen.

#### Performance

While `log` is pretty quick, you might find yourself wanting more power in terms of
what you're printing out.  Maybe you don't want to print "Hello3" 50,000,000 times
per minute because someone left a debug message on in production.

So you want some structured logging.  But it's gotta go *fast*.

[Take a look at Zap](https://github.com/uber-go/zap#performance) from Uber.  This bad boy can
spew out logs *fast*.  It's even faster than the standard library!  Daaaaaaamn.

This is, pretty clearly, optimizing for performance.  So why wouldn't you use this?
Why would any other logging library ever exist when this thing just destroys everything
else?

Well...

https://godoc.org/go.uber.org/zap

This isn't unreasonable.  This isn't a bad design.  But if you can tell me with a straight
face that this isn't going to scare your junior devs at first glance, you've forgotten
what it's actually like being a junior dev.

## Living in Harmony

A single project should probably not optimize for only one of these things.
A project should *optimize for solving a problem.*  This will involve many things,
but asking yourself "Is what I'm doing helping solve the problem?" happens less
often than you might hope.

The key is clearly noting what's being optimized for and where.  This can be a general
"In this project we favor simplicity over performance" note in a README.  Then maybe
"While we favor simplicity over performance, this area of the code is a tight loop
so we're going to get more complicated here" lives in a comment above a gnarly piece of
highly optimized madness.  The point is that as a developer that's coming in will know
what's being optimized for and know what kinds of modifications are appropriate.

Importantly, these areas should be fenced off.  Saying "Once you go into this function,
things are going to get crazy"

## Optimize for teamwork

Whatever you're optimizing for, make sure the entire team is in on it.
If one developer is optimizing for "ship it fast", another is optimizing
for "long term stability", another is optimizing for "high performance",
and another is optimizing for "looking at cats on the internet", this team
is going to be a disaster.  Also you should talk to that last dev and block
CatHub on your corporate firewall.

As above, this will likely shift a bit between various pieces of your code base.
It's still valuable to set general guidelines up front and make them required
reading for new devs.  Just keep them short and readable; optimize for understanding,
if you will!

> Prefer simple and explicit. (Optimize for stability)

> Keep memory allocations to a minimum. (Optimize for performance)

> Move fast and break things. (Optimize for shipping it)

## Stupid things to optimize for

### Feeling clever

**IF YOU TAKE AWAY ANYTHING FROM ANY OF THIS, THIS IS IT.  DO NOT OPTIMIZE FOR FEELING CLEVER.**

Please.

Pleeeeaaaaase.

Please don't optimize for feeling clever.
