# Saying "there's no right way to do it" is a lazy cop-out

subjective != arbitrary
The arbitrary is a proper subset of the subjective
Proof: Grading climbing routes is subjective but not arbitrary (there is reason and a system involved)

The phrase is of course true (noSQL databases aren't inherently superior or inferior to relational databases in every way) but given the appropriate conditions, one solution will show itself to be superior to the other.

1. It's a waste of breath - It's like saying the world isn't perfect. True, but uninteresting and unproductive.
2. It's often used to mask sloppy thinking or esckew critical analysis of the pros and cons of competing approaches in favor of choosing the one which promises the least amount of effort.

If you really find the difference between two approaches negligable, then say so. That's at least a claim which can be countered. The moment you do, I bet you people will speak up

Would a civil engineer say that when designing a bridge? Who about a chemist while deceloping a new drug to treat dementia?

Admittedly, with some exceptions, people's lives aren't at stake when our code crashes, but what of their privacy, personal info, finaces, etc?

Other variations include: "don't take this article/talk/pattern as gospel"

Again, this is something we as engineers shouldn't need to be told. In fact, if the presentation is cogent, it shouldn't self-evident. We don't need to take it as gospel, because you've provided us with data and arguments which are able to be verified and criticised.

Again, this often stops development of a concept short of a quite comprehensive and robust aproach or best-practice. It's unhelpful to have a concept presented only to hear "don't take this as gospel. There are cases where this may not apply." You're not much better off than when you started. What are those cases? Can you give an example When you go to implement a feature and you reach for the presented pattern, now you're wondering, "is this one of those aforementioned cases?" So you either use it or you don't, not feeling confident either way.

(Admittedly, this is easier said then done and can often require some level or sorcery to achieve which I can't in good conscience endorse.) However, it's worth breaking down the known corner-cases, because in most cases, they are finite, if not elusive. Then, down the line when a new pracice or language-feature changes the game (e.g. proper tail-recusion in ES6 WHOOO) you can keep that list up to date (hooray for Medium edits)

The best analogy I can think of is the difference between saying:
You shall not lie (but don't take this as gospel, use your best judgement!)
You shall not bear false witness against your neighbor

Or, purhaps more comprehensible:
You shall not kill (unless it's better to kill than not to, then yeah, do that)
You shall not murder (and then proceed to explain what kind of killing qualifies as murder, e.g. when you or someone else isn't in mortal danger, when it is premeditated, etc.)

All we did is add some conditions to the general rule which helps clarify when it is appropriate to apply. (I actually think we do a pretty good job of this as an industry, but I still see instances where we stop short of even attempting to enumerate PROs and CONs or discussing under which conditions one approach is superior to another.)
