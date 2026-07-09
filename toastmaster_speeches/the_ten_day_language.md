# The Ten-Day Language
### A speech on JavaScript, evolution, and why almost nothing ever gets a clean rewrite

---

**[COLD OPEN]**

In May of 1995, a programmer named Brendan Eich sat down and, in ten days, wrote the first version of a programming language.

Ten days.

That language now runs on almost every device with a screen, on this planet, right now. Your phone. Your laptop. The screen at the DMV. The display on some gas pumps. It is, by a wide margin, the most widely run programming language in human history.

And here's the part I love: its name is a lie. And some of the strangest bugs in modern software are direct, permanent fossils of that ten-day sprint — bugs that were never fixed, because by the time anyone noticed, it was too late to fix them. Tonight I want to tell you that story, because I think it's actually a story about something much bigger than code.

---

**[SETTING THE SCENE — 1995]**

Rewind to 1995. Netscape Navigator is the browser everyone uses. The web exists, but it is *static*. A web page just... sits there. No animations. No form that tells you "that's not a valid email" before you hit submit. Every single interaction meant reloading the entire page from a server, over a phone line, making that screeching modem noise while you wait.

Netscape needed the web to feel *alive* — and they needed it fast, because a company you may have heard of, Microsoft, was about to enter the browser business with Internet Explorer.

---

**[THE JAVA CONNECTION]**

Now, Netscape had a partnership with Sun Microsystems, the company behind a serious, heavyweight programming language called **Java** — built for professional developers writing full applications, the kind of thing you'd compile overnight and deploy to a bank.

But Netscape didn't need girders and steel beams. They needed duct tape. They needed a *scripting language* — something lightweight, something you could write directly into a web page and have it just... run. Instantly. No compiling, no waiting.

So they brought in Brendan Eich, and gave him ten days to build it.

He built it. First they called it Mocha. Then LiveScript. And then, right before launch, in a move that was pure marketing, Netscape renamed it **JavaScript** — purely to ride the buzz around Java. The two languages share almost nothing structurally. It's a little like naming a moped "CarLite" because cars were popular that year.

---

**[WHY HOBBYISTS RAN WITH IT]**

But here's the thing that actually made JavaScript take over the world — and it wasn't the technology. It was that JavaScript lived right inside a web page. Which meant anyone, anywhere, could hit "View Source" in their browser and just... read the code. Copy it. Tweak it. Break it. Fix it.

No installation. No computer science degree. No compiler. Just a text editor and curiosity.

That single fact turned an entire generation of hobbyists and tinkerers into programmers, almost by accident. And it's a huge part of why the interactive web exploded the way it did — blinking text, popups, dropdown menus, all of it — in a way Java's more locked-down, professional model never came close to matching.

---

**[THE SCARS OF A TEN-DAY LANGUAGE]**

But you don't write a language in ten days without leaving scars. And here's where it gets fun — a few of my favorites:

In JavaScript, if you check the type of "nothing" — of `null` — it tells you it's an **object**. That's just... wrong. It's been wrong since 1995. It's a known bug with a name and everything, and it can *never* be fixed, because too much of the internet now depends on that wrong answer.

JavaScript will also tell you that `0.1 plus 0.2` does not equal `0.3`. It's very close, but not quite — a side effect of how it stores decimal numbers under the hood.

And my personal favorite: JavaScript has a special value that literally means "Not a Number" — and if you ask it whether Not-a-Number is equal to itself, it says no. Not a Number is not even equal to Not a Number.

These aren't edge cases nobody hits. Developers run into this stuff constantly. And because JavaScript now runs the *entire web*, nobody can just go in and fix it. The unbreakable rule of the web is: **never break the old websites.**

So instead of fixing the language, we built entire layers of tooling *on top of it* — things like TypeScript, which checks your code before it runs, specifically to catch these thirty-year-old landmines before you step on one. Modern web development is, in a very real sense, an elaborate set of scaffolding built to protect us from decisions made in a single week in 1995.

---

**[THE ZOOM OUT — EVOLUTION AS TINKERER]**

Now here's where I want to zoom all the way out, because I don't think this is really a story about software.

There's a great line from the biologist François Jacob: **evolution is not an engineer, it's a tinkerer.** An engineer gets to start with a blank sheet of paper. A tinkerer only gets to work with whatever's already there, alive, and running — and it can never be taken offline for repairs.

Look at a giraffe. The nerve that runs from its brain to its voice box takes a bizarre detour — down the entire length of its neck, loops around a major blood vessel near the heart, and comes all the way back up. In a fish, that nerve took the short, direct path. But as necks got longer over millions of years, evolution couldn't reroute it — rerouting would mean breaking something that had to keep working at every single step along the way. So the giraffe is stuck, forever, with a fifteen-foot detour to cover a distance of about four inches.

That's not a design. That's a patch. On top of a patch. On top of a patch, going back millions of years. Same with the blind spot in your eye, from a retina that's wired in backwards. Evolution didn't fix it. It just made every one of us live with it.

---

**[BRINGING IT HOME — SEATTLE]**

And you don't have to look at giraffes to see this. You can look out the window.

For years, this city had a structurally compromised elevated highway running right through downtown — the Alaskan Way Viaduct — and Seattle couldn't just delete it. It had to keep carrying ninety thousand cars a day while an entire replacement tunnel was built *underneath* it, live traffic running overhead the whole time. That's not an engineering problem. That's a tinkering problem. We patched the city while it stayed running, because there was never a version where we got to shut Seattle off for a few years and start clean.

---

**[GENERALIZING — WHEN DO REWRITES ACTUALLY WORK?]**

So — when *does* a clean rewrite actually work? Because it does happen, sometimes.

In 1971, the United Kingdom switched its entire currency system in a single day. Decimal Day. Every bank, every shop, every price tag, all at once. That's a real rewrite. But notice what made it possible: a central government with the authority to force everyone to switch at exactly the same moment.

Compare that to something like Python — a major programming language that, a number of years back, planned and announced a clean rewrite, version 2 to version 3, run by the very people who controlled the language. Total authority. Years of advance notice. Good intentions all around.

It still took over a decade to fully happen. Because even with total authority, you still had millions of independent developers, each with their own reasons to wait just one more year.

So here's the real rule: rewrites don't fail or succeed because of the technology. They live or die on **coordination** — on whether you can get everyone to move at the same moment. That was never really the hard part with JavaScript. Anyone could write a better language in a weekend. The impossible part was getting a billion independent websites to agree to switch over on the same day.

---

**[THE AI QUESTION]**

Which brings me to the question I actually want to leave you with.

We're told, constantly right now, that we're entering an age where AI lets us finally rewrite things — rebuild our systems clean, shed decades of legacy code and legacy rules we've all been quietly dragging behind us.

And there's something real there. AI tools are already being used to take ancient, brittle banking software — code nobody currently at the company even understands anymore — and translate it into something modern. That's not hype, that's happening today.

But I'd ask you to notice something. The thing that always made rewrites hard was never really the coding. It was getting a million stakeholders to agree to flip the switch at the same time. And AI, so far, has shown no particular gift for making people agree with each other. It can write beautiful new code. It cannot make Congress, or a homeowners' association, or a giraffe's nervous system, hold a vote and agree to change all at once.

So maybe the honest promise of this next age isn't "we finally get to start over." Maybe it's something a little less dramatic, and a little more useful: we get *much* better at tinkering. Better at patching, adapting, and managing decades of accumulated complexity — in our code, in our institutions, maybe even in how we talk to each other across genuine disagreements — without ever needing to shut the whole thing down first.

---

**[CLOSE]**

JavaScript never got its clean rewrite. It just kept getting patched, year after year, by people making the best of a language built in ten days.

Neither did this city. Neither has most of what we depend on.

Maybe the real skill was never demanding a rewrite.

Maybe it's getting good — really good — at patching things together, while the lights stay on.

Thank you.

---

*[Estimated speaking time: 6–7 minutes at a moderate pace. Trim the "SETTING THE SCENE" or "SEATTLE" section by a sentence or two if running long.]*