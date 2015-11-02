---
layout: post
title: "Effective Code Reviews"
date: 2014-09-29 12:12
comments: true
categories:
author: "David Beveridge"
---

Code reviews are a great way to improve code quality, catch bugs, and keep your
whole team in sync with the latest changes.  My favorite tool for doing a code
review is still Github's [Pull Request][pullrequests] feature, because of the
easily-accessible diffs, inline comments, and commit-level and PR-level views of
the file changes.

So, how do we make the code review process effective? I offer the following
tips:

* Actually review PR's; telling an author to "just ship it" is detrimental.
* Senior developers *want* you to review their code; don't be scared.
* Close PR's that don't have specs
* If spec coverage is sparse, discuss it with the author (nicely).
* Spec descriptions should accurately reflect the behavior of the module/method
  they describe.
* Ask questions instead of making statements; the author likely put more thought
  into the code than you are aware of, and they've already solved the problem
  once.
* Look for design problems:
  - Does this change solve a real problem? is it worth solving?
  - Given the constraints, is the overall approach correct and reasonable?
* Look for [code smells][codesmells] and suggest refactorings.
* Ask for help with reviewing code that is outside your area of expertise.

<!--more-->

--------------------------------------------------------------------------------

Ever seen a "code review" that looks like this?

> LGTM :ship:

This is not a helpful code review. Rather than communicating the message, "good
job," it tells the author that you can't or won't take the time and effort
required to properly review their work.

If you've ever done this, you're not alone, and I'm not going to yell at
you—I've done it too.

I often hear people say they aren't qualified to review some portion of a pull
request; generally, this is because of a lack of familiarity with the language
used, or of the frameworks.  While this is an understandable view, I argue that
**you are**, in fact, **qualified to review the code**.  That's true even if it's
not your favorite language, or you're not particularly good at programming in
that language, or you're not familiar with the framework.  How do you review
code you're not well-versed in?

## Remember that code reviews are a service, not an insult.

Please, please, review my code! I think everyone who has a title like "Senior
X", "Director of X", "X Architect," or whatever else knows the importance of
having someone look over their code, ask questions, and point out mistakes.  You
don't need to be afraid of reviewing someone's code, no matter their position.
It's ok.  I am not my code.  Neither is anyone else.  I also still write bad
code sometimes.  So does everyone else.  We all need peer reviews, and we are
all peers here.

Moreover, if you are reviewing code from a senior developer, **don't be a
yes-man!** Point out mistakes you see, ask about why specs are written a certain
way, ask about architecture decisions, point out weird styling issues...
"Senior" people need help too.  If someone gets mad at you for nicely
pointing out oversights or for asking questions, that's a sign of immaturity,
not seniority.

## Ensure that there is only one feature in the pull request

Each organization does things a bit differently, but ideally, your pull requests
should correspond to exactly one user story.  If you can achieve this level of
granularity, you gain a number of benefits:

* A feature spec can accompany the PR, and can be verified by your CI server.
* The code will be easier to reason about.
* If you use the [nvie branching model][nvie] (and you squash on merge), you can
  easily define different releases and cherry-pick features as you please.

## Check that (nearly) all code has an accompanying specification.

If someone writes code without writing specs, they've introduced an unspecified,
undocumented landmine into the code base.  Your organization may have some
policy about test-first, coverage level, etc.; I'm not going to get into that.

If you review code and there are no specs, talk to the author about it first.
Discuss the importance of specifications, and offer to help them with writing
some.  I often suggest that the PR be closed and the code be rewritten using
TDD.

Why do I recommend rewriting the code? Because if code was not driven by a spec,
it is difficult to specify ex post facto.  Bolting unit tests onto untested code
results in brittle tests, encourages kludgy design, increases technical debt,
and will make refactoring difficult in the future.  If tests must be added on
afterwards, they should be intergration tests.

I don't say any of this to encourage dogmatic, or bully-like behavior.  You and
the author should discuss the specs, and their importance, and can come to a
decision about the code together.

P.S.: Don't close PR's for purely cosmetic (i.e., only CSS/HTML) changes
on this basis.

### What about spikes? Aren't they the exception?

Nope.  Spikes are a really great tool, but their purpose is to provide a
short-term proof-of-concept.  They shouldn't be merged; in fact, they shouldn't
even be reviewed.  After a spike, should do a quick rewrite with specs.

## Look for specs, and ask about the coverage level.

The first thing I do when I look at a PR is check the list of files changed (the
github email digests are useful for this).  If you see `generic_modal.coffee`
was changed, then you should also see that `generic_modal_spec.coffee` (or
something else that matches your naming scheme) was changed in the same PR.

Ideally, they should be changed in the same commit, or the spec should be
changed first.  This is not a rule, just a guideline; everyone commits a bit
differently.

As mentioned above, keep an eye out for a feature spec (aka integration test).
Optimally, there should be a one-to-one correspondence between a user story, a
feature spec, and a pull request.

There should be relevant specs for all the code, both frontend and backend.  If
you don't see something close to a 1-1 mapping between specs and source code,
ask the author what's going on.  Some source code doesn't need its own spec
file: it may have been extracted from a spec'd module during a refactor; that's
ok.

If the author just neglected to write the specs, the PR should not be merged.
This PR may be used as a reference for a spec-driven rewrite of the feature.  If
I see a PR without tests, I will close it. Specs are what give us the confidence
we need to make changes later; we can't afford to neglect them.

## Read through the specs; they should make sense.

When you read through the specs, you should be able to quickly understand what
each module does and what its public interface looks like.  You should see
relevant `describe` blocks, `context`s, and `it` descriptions (or whatever your
testing framework provides); they should describe, in English, what the
module/API/UI does, and the code inside each should reasonably correspond to
that description.

Ask the author to change non-obvious descriptions and tests; suggest a better
description.  For example:

```coffeescript
describe '#fullName', ->
  it "works", ->
    person = new Person("John", "Doe")
    expect(person.fullName).to.equal("John Doe");
```

The above spec, "it works" makes no sense.  The code inside is fine, but the
description should be more like:


```coffeescript
describe '#fullName', ->
  it "yields the full name of the person, space-separated", ->
    person = new Person("John", "Doe")
    expect(person.fullName).to.equal("John Doe");
```

## If there is backend and frontend code, mentally separate them and review both.

In a web project, many times the backend and frontend components are written in
different languages.  When you're only working on one part of the stack, it's
easy to just ignore code from the other parts.  Instead, use PR's as a learning
opportunity; the other parts of the stack will affect the maintainability of the
project just as much as the parts you're working with, so make sure you look at
both.

I find that it helps me to treat the backend and frontend code in a PR as if they
were separate PR's.  It's just a mental separation for me.

This works best when a pull request is for only one feature.  The more features
that are lumped together, the harder it is to review and, as a result, the
largest PR's end up with the worst reviews.

## Be encouraging

The purpose of code reviews is to build up, not to tear down.  That goes for both
the code and the team.  If you see something that was done well, say so! For
example, "I like the way you were able to refactor that giant for-loop into a
couple of `map` and `reduce` calls! I'll have to keep an eye out for that in my
own code!"

Everyone likes to receive positive feedback about their work.  Plus, it will
make you seem more appealing to work with, and it improves cohesiveness with the
rest of the team.

Also, try adding some [emoji][emoji], or some [funny gifs][gifs] to lighten the
mood.

## Have you tried asking questions, rather than making statements?

According to Dr. Albert Mehrabian, only [7% of our face-to-face communication is
accounted for][nonverbal] by the words we say.  In the faceless, voiceless world
of a text-based code review, it can be easy to offend someone, especially since
people in our line of work tend to put so much care into their code.  An easy
way to diffuse this is to ask questions.  For example, you see the following
code:

```coffeescript
rgba = hex.split(/\d{2}/).map (x) parseInt(x)
```

You could say,

> This is wrong.  Your split returns ["", "", ""], and parseInt needs a radix of
> 16.

Or, you could ask,

> This doesn't make sense to me; I think this will always yield `[NaN, NaN,
> Nan]`.  Did I miss something?

Which would you rather someone post on your PR? I prefer the latter.

Asking questions is a powerful thing.  When someone questions my code, sometimes
that's the first spark of thought about an alternative approach.  Sometimes,
it's a gentle reminder that I left out a use case, or forgot to remove my debug
lines, or left a swath of commented code.

My favorite is when someone new to my language of choice asks a question about
my code.  Sometimes it's a chance for them to learn something new.  Other times
it's a chance for me to break an age-old habit; you wouldn't believe how many
IE5/6 hacks I use out of habit...

If you're not sure whether something is right, just ask the author.  If it's a
mistake, they'll recognize it and provide a fix.  If it was intentional, they
can justify whatever it is; you may even learn something.

## Suggest refactoring in both the specs and the source.

Sometimes others point out code that I could refactor better; it's easy to
think I'm done refactoring when my head has been stuck in the same code for the
last 6+ hours.

In the specs, point out where the testing tools could be better used.  For
example, I often forget about some of the features of a stubbing library.  I
sometimes write verbose, repetetive tests that could be extracted into a test
helper.  I also find that sometimes other developers can help make my tests less
fragile.  If you have an idea about improving the specs, share it!  It may be
helpful to add custom matchers to your spec suite; most tools support this in
some fashion.

Do the same with the code! If you see where a specific refactoring can apply,
say so.  Look for code smells; you can refer to [Cunningham & Cunningham's wiki
page on Code Smells][codesmells].

We need to strike a balance here, too.  The point of refactoring is to make
the code easier to reason about, not to push indirection to its limits.

## Ask for help.

It never hurts to have another pair of eyes on the code.  Ask a senior person in
your organization (or another open-source developer) to take a look; most will
be glad to.  If you know someone else with relevant experience, ask them!  Most
developers will gladly look over a PR with you.

If the PR contains a lot of code in a language you're not great at, be proactive
and find someone who is, and ask them to review with you!  For example, if
there's a lot of CSS, and you're not great at it, ask a frontend developer to
help.

## The Code Checklist

When it comes to most code (obviously this doesn't apply to markup or styling
code), try the following checklist:

- Is every piece of code in the right place, i.e. model code in a model,
  controller logic in a controller, app-specific helper code in a helper,
  generic helper code in a library?
- Do all classes have only one responsibility?
- Do all methods do only one thing?
- If a method has a side-effect, is it clear from the name, and otherwise
  documented?
- Are all classes/methods/variables named properly so that the code is
  self-describing?
- Is everything as private as possible, i.e. only the intended public interface
  is public?
- Are too many things private?  This could be an indication that you should
  extract an object.
- Are all files within a reasonable size (e.g., less than 100 lines of code)?
- Are all methods less than 10 lines of code?
- No law of demeter violations (providing whole objects to methods when all
  that's needed is the value of one attribute of them)?
- Is everything tested? Is each thing tested enough?
- Are there tests for private methods? This is a code smell.
- Every class should have a small comment describing what it represents/does.
  Public methods should have comments describing what they are for, or when to
  call them, if this isn't obvious from the code. Comments shouldn't describe what
  the method does (this is visible from looking at the code).
- Are there any obvious performance-stupidities, like making a database query
  for each loop iteration, rather than using a more optimized query that loads
  all data at once?
- Spacing errors like no empty line between methods, or too many empty lines
- There shouldn't be any commented-out code.
- There should be no debug statements like "console.log" or the likes.


*Note: This article was first posted at [blog.originate.com][original-post]*

[pullrequests]:https://help.github.com/articles/using-pull-requests
[codesmells]:http://c2.com/cgi/wiki?CodeSmell
[nvie]:http://nvie.com/posts/a-successful-git-branching-model/
[emoji]:http://www.emoji-cheat-sheet.com/
[gifs]:http://giphy.com/
[nonverbal]:http://www.kaaj.com/psych/smorder.html
[original-post]:http://blog.originate.com/blog/2014/09/29/effective-code-reviews/
