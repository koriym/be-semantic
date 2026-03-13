# Insights: Observations and Reflections

## Domain Comes Before Pattern

There's a common approach to software design that goes something like this: learn design patterns, then apply them to your domain.

The result is code shaped by patterns, where the domain has been squeezed into pattern-shaped containers.

The better direction is the reverse: understand the domain deeply, and let the patterns emerge from that understanding.

This is true of the best frameworks. BEAR.Sunday didn't emerge from "let's implement MVC with resource orientation." It emerged from looking at how the web actually works — resources, hypermedia, state transitions — and letting that reality determine the structure.

ALPS didn't emerge from "let's create an API description format." It emerged from asking "what does an application *mean*, independent of its implementation?"

**Be-semantic embodies the same principle at the project level.** We look at the domain — user stories, realistic data, stakeholder understanding — and let the schema emerge from that observation.

Pattern follows domain. Not the other way around.

---

## The 50-Entry Discipline

Why 50 fake entries? Why not 5 or 10?

With 5 entries, you can construct them too deliberately. You think "I'll make one short title, one long title, one with a memo, one without." You're still prescribing.

With 50 entries, you have to be realistic. You can't carefully craft each one. You have to ask: what would 50 real users actually create? And when you look at 50 realistic entries, patterns emerge that you didn't put there.

The longest title is 41 characters. Not because you decided "titles up to 41 chars," but because when you imagine 50 realistic todos, nobody writes a 100-character title.

The memo is null in 35 of the 50 entries. Not because you decided "memo is optional," but because most todos don't need a note.

This is the difference between prescribing constraints and observing them. The 50-entry exercise forces you into observation mode.

---

## The Agreement Step is the Most Undervalued

In a solo project, it's tempting to skip the Agreement step. You generated the fake data, you reviewed it yourself, you know what you decided.

But the Agreement step isn't about the data. It's about shared understanding.

When a product owner looks at 50 fake todos and says "these look right to me," they are implicitly confirming:
- The vocabulary (that "title" and "memo" make sense to them)
- The structure (that a todo has these fields and not others)
- The edge cases (that it's okay for memo to be null)

When a product owner looks at 50 fake todos and says "wait, where's the due date?" — that's a discovery that would have been expensive to find in code review.

**Fake data is a cheap communication tool.** It makes the abstract concrete. It gives stakeholders something they can react to instead of a schema they can't read.

---

## Schemas as Records, Not Prescriptions

A JSON Schema written without the Fake + Agreement steps is a guess.

A JSON Schema written after those steps is a record.

The difference is significant. A "guess schema" will be wrong in subtle ways that only appear when real users send real data. A "record schema" captures what the team actually agreed the domain looks like.

`maxLength: 255` is a guess (or a habit from database defaults).  
`maxLength: 80` derived from observing that real titles top out around 41 characters is a record.

This also changes how you respond to edge cases. When someone asks "why is maxLength 80?", you can say: "because we looked at 50 realistic examples and the longest was 41 characters, so we gave double the room." That's a defensible, traceable decision.

---

## The Cost of Skipping

Every step in the workflow has a cost. It also has a cost if skipped.

| If you skip... | You risk... |
|----------------|-------------|
| Story | Implementing features nobody needs |
| ALPS | No shared vocabulary; different people mean different things by the same words |
| Fake | Constraints that don't match reality; surprising edge cases in production |
| Agreement | Discovering misunderstandings in code review or after launch |
| Schema | Implementation drift; each developer has different assumptions |
| Be | The implementation diverges from the domain model |

The workflow is designed so each step makes the next step easier and cheaper.

---

## What This Has to Do with Be Framework

Be Framework's philosophy is "Be, Don't Do." Objects should *be* something, not *do* things.

This aligns naturally with be-semantic:

- `Story` defines what users want to *be* (a person with organized todos)
- `ALPS` defines what the application *can be* (in various states)
- `Being` in code represents things that *are* (a todo list, a todo detail)
- `Final` represents the transition to a new state of being

The whole workflow is oriented around *states of being* rather than *sequences of actions*.

When the design starts from meaning — from what things *are* — the Be implementation becomes a natural expression of that meaning.
