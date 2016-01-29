State Synchronization
#####################

:date: 2016-01-28 17:08
:tags: software-architecture, distributed-systems
:category: software-architecture
:slug: state-synchronization
:authors: Tom Burdick
:summary: State Synchronization, Or Why Distribution and Microservices are Difficult without you Knowing It.
:status: draft

Microservices are all the rage these days. The pros are all there, widely known.

Pros
----
 * Independent Book Keeping (Tied to the resources point below)
 * Independent Code Base
 * Independent Team (Huge)
 * Independent Deployments
 * Independent Computing Resources (Huge)
 * Independence Day

Cons
----
 * The spaghetti moves from Code to Networking
 * Networking means a lack of common state knowledge
 * A lack of common state knowledge means lots of little notes
 * If notes get lost, displaced, rewritten, or garbled, all hell breaks loose.

So assuming microservices, ahem, distributed computing state knowledge needs
to be moved around somehow so we can actually do anything. Pushing or pulling,
the entire state of the universe, functional updates, differential updates.
Those are really the only big choices we have to make then. Maybe a combination.

Depending on the scenario sometimes one method or another becomes the
clear answer. If the state of the universe is tiny, then its easy. We just
copy that around whenever we need it from where ever we need it. Done.

Lets just skip all the easy stuff though. Lets choose the hardest possible
situation to deal with. You know the kind, the one where we lose sanity and hair.
The kind that almost inevitably always shows up, every, damn, time.

* State is large
* Updates are frequent
* Latency is important
* f(g(n)) != g(f(n)), state change ordering matters

I mean, basically I just described how the universe itself works I suppose, but
the state of the stock market and orders might also be a close second.

Timetraveling magical aliens might be able to save the state of the universe (dr manhattan?),
mail it to their buddy, and have them take a laugh at the ridiculous scenarios
going on around the planet at that very moment easily. But I sure as hell don't
know how to do that.

So forgetting the impossible, whats the next best thing.

We want...

* fewest messages possible, more messages means more potential fudge ups.
* smallest messages possible, small messages arrive faster and are less likely
  to be ruined by accident prone package handlers.

Sending Functional messages

Alice comes up with a pseudo random number using a seed 0, then adds 2 and multiplies by 2.
mult(add(pseudo_random(0),2),2)

Bob wants to know what alices number is, so Alice tells sends him 3 letters, one for each operation

0: random 0
1: add 2
2: mult 2

Looks fine, Bob should end up with the same number as Alice.

Assumptions Made
* Letters may not arrive in order, but so long as they arrive reliably thats ok, ordering is known
* Letters sit around and wait for Bob to open them, they have assured delivery.
* Condensing letters really isn't all that possible usingly solely functional operations. Unless the sender knows for certain f(g(j(n))) = x(n)

Wait a second... that second one seems awfully presumptious. I mean, what happens that one day when it rains down acid
rain or the postman quits in a rage and tosses that days mail in the furnace or Bob's mailbox fills up or... a million other things?

Right, so we can assure ordering is correct, but not necessarily consistent delivery, for many many reasons.

So what happens then, when Bob misses a letter. Well wait, how would Bob even know he definitely missed a letter and it wasn't just
arriving late? He could put a time limit on these things. Lets say he was willing to wait a Day.

Day 2 arrives, he's still missing letter #1, in the meantime alice has been sending more letters.

3: sub 2
4: div 2
5: pow 9
6: fact

Great...

We've got a pile of Alice letters, and have no clue what state alice's number actually is.


Option #1, we just ask for what her number is right now.
Option #2, we ask for a redelivery of letter 1.

All things are terrible, the least terrible probably being Option #1.


Lessons learned...

* State change differentials are useless typically without a known and easily understood ordering (incremental integers) where missing state
  changes can be easily verified and determined
* There must be a way to determine the state at any given state increment.

My naive integer ordering above assumes only one party is updating state at a given time. It also naively assumes there's actually only one party at all.

So how can we replace 0 -> 6 above with something meaningful. First thought, hashes, duh.

hash(random(0)): random 0
hash(add(random(0), 2)): add 2

so on...

Ok, looks promising, until we introduce Jane.

Jane throws a wrench into the already seriously complicated situation, she decides to take
hash(add(random(0), 2)): add 2 and do...
hash(multi(add(random(0), 2), 4)): mult 4

Rageface. Time to google the solution.

Googling... bigtable nope, 
