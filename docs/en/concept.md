# Concept: Design from Meaning

## The Core Idea

Software development has a habit of starting in the wrong place.

We reach for frameworks, choose databases, design tables — all before we deeply understand what we're actually building. The result is code that speaks the language of technology, not the language of the domain.

**be-semantic** proposes a different starting point: *meaning*.

Before asking "how do we store this?" or "what does the endpoint look like?", we ask:

> What does the user want to do?  
> What does that action *mean* in the real world?  
> What are the actual constraints on that meaning?

## Semantics Before Structure

The word "semantic" comes from the Greek *sēmantikós* — "significant, having meaning."

In software, semantics refers to *what something means*, as opposed to *how it is implemented*.

Consider a field called `title`:
- A database designer might say: `VARCHAR(255)` — because that's the default
- A frontend developer might say: `string, required` — because it's in the form
- A domain expert might say: "the title someone would actually write" — which, when you look at 50 examples, is never more than 80 characters

The last answer comes from *semantics*. The others come from habits.

## The Role of Each Step

### Story: Grounding in Intent

User stories in the format "As a user, I want to... because..." force us to articulate the *why* behind every feature. This is the foundation.

A story like "I want to filter tasks by completion status because I need to focus on what's left to do" tells you:
- There is a concept of "completion status"
- It is binary (done / not done)
- It exists to serve focus, not just record-keeping

This shapes everything downstream.

### ALPS: Vocabulary Before Implementation

ALPS (Application-Level Profile Semantics) describes what an application *can do* and the vocabulary it uses — independent of any technology.

It answers: "What are the actions? What are the data elements? How do they relate?"

Not: "What are the REST endpoints?" or "What are the database tables?"

ALPS is the shared language between designers, developers, and stakeholders. When everyone agrees on what `doCompleteTodo` means, implementation becomes a translation task.

### Fake Data: Observation Before Prescription

This is the most counterintuitive step, and perhaps the most valuable.

Before defining any constraints, we generate 50+ realistic fake data entries. Not random garbage — realistic data that looks like what real users would actually create.

Then we *look at them*.

What's the actual range of title lengths? Are there any null memos? What does a "completed" todo look like at 2 AM vs. 2 PM (does `createdAt` matter)?

This observation changes the questions we ask. We stop prescribing constraints from abstract rules and start *discovering* them from observed reality.

### Agreement: Shared Understanding

The fake data review isn't just a solo exercise. It's a conversation with stakeholders.

"Look at these 50 todos. Do they look right? Is anything missing? Is anything wrong?"

This step aligns everyone before a single line of real code is written. It's far cheaper to discover a misunderstanding here than after implementation.

### Schema: Recording Reality

By this point, the JSON Schema almost writes itself.

`maxLength: 80` because the longest title we observed was 41 characters, and 80 gives room for the unusual case.

`"memo": { "type": "string" }` is optional, not required — because many todos in the fake data had no memo, and that's realistic.

The schema is not a prescription. It's a record of what we agreed is true.

### Be Implementation: Code That Mirrors the Domain

With the schema as a contract, Be Framework implementation becomes straightforward. Each `Being` represents a domain concept. Each `Final` represents an outcome.

The code reads like the domain, not like the database.

## Why This Matters

The alternative — jumping straight to implementation — produces code that is technically correct but semantically wrong. It passes tests but doesn't quite fit the domain. It requires constant translation between "what the code does" and "what the business means."

be-semantic is a discipline for closing that gap before it opens.

## The Deeper Philosophy

There is a parallel to how good frameworks are designed.

The best frameworks — BEAR.Sunday, Be, ALPS itself — didn't emerge from "what patterns should we implement?" They emerged from looking at domains, understanding them deeply, and letting the patterns arise from that understanding.

Pattern follows domain. Not the other way around.

This is the spirit of be-semantic: let the constraints emerge from reality. Let the design follow the meaning.
