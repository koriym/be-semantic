# Workflow: Step-by-Step Guide

## Overview

```
Story → ALPS → Fake → Agreement → Schema → Be Implementation
```

Each step has a clear input, a clear output, and a clear question it answers.

---

## Step 1: Story

**Question:** What does the user want to do, and why?

**Format:** User stories in structured natural language.

```markdown
As a [user type],
I want to [action],
because [reason].
```

**Output:** A `story/main.md` file listing all user stories, including a brief description of the application and its core entities.

**Tips:**
- Write stories from the user's perspective, not the system's
- The "because" clause is mandatory — it reveals the domain's true intent
- List your entities at the bottom: what concepts exist in this domain?

**Example:** See [`example/story/main.md`](../../example/story/main.md)

---

## Step 2: ALPS

**Question:** What can the application do, and what vocabulary does it use?

**Tool:** [ALPS](https://alps.io/) (Application-Level Profile Semantics)

**Output:** An `alps/[name].json` file defining:
- **Semantic descriptors:** the data elements (id, title, memo, isCompleted...)
- **Safe descriptors:** read operations (goTodoList, goTodoDetail...)
- **Unsafe/idempotent descriptors:** write operations (doCreateTodo, doCompleteTodo...)
- **Representations:** the compound views (TodoList, TodoDetail...)

**Tips:**
- Use `def` to link descriptors to Schema.org or other standard vocabularies when possible
- Naming convention: `go` prefix for safe (read), `do` prefix for unsafe/idempotent (write)
- Validate your ALPS file: [https://alps-io.github.io/](https://alps-io.github.io/)
- Visualize the state diagram: [app-state-diagram](https://github.com/koriym/app-state-diagram)

**Example:** See [`example/alps/todo.json`](../../example/alps/todo.json)

---

## Step 3: Fake Data

**Question:** What does this domain look like in reality?

**Goal:** Generate 50+ realistic fake data entries that look like what real users would create.

**Output:** A `fake/data-50.json` file (or `.jsonl`) with realistic sample data.

**How to generate:**
Use an AI model with a prompt like:
```
Generate 50 realistic fake Todo items. Each should have:
- todoId: ULID format (26 chars)  
- todoTitle: varied realistic titles (short, medium, long)
- todoMemo: sometimes null, sometimes a brief note
- isCompleted: mix of true/false
- createdAt: ISO 8601 timestamps over the past 3 months

Make them feel like real data from real users, not random test data.
```

**What to observe:**
- Length distribution of text fields
- Null/empty frequency for optional fields
- Value patterns and ranges
- Edge cases that naturally appear

**Example:** See [`example/fake/data-50.json`](../../example/fake/data-50.json)

---

## Step 4: Agreement

**Question:** Does this data match everyone's understanding of the domain?

**Activity:** Review the fake data with stakeholders (product owner, designer, other developers).

**Key questions to ask:**
- Do these look like real todos?
- Is anything missing that real users would need?
- Is anything here that real users would never do?
- What are the realistic limits? (max length, required vs optional...)

**Output:** A shared understanding of the domain constraints, documented as notes or directly as decisions in the schema.

**Why this step matters:**  
Misunderstandings discovered here cost nothing to fix. Misunderstandings discovered after deployment cost everything.

---

## Step 5: Schema

**Question:** What are the agreed constraints on this domain?

**Format:** JSON Schema (one file per representation)

**Output:** Files in `schema/` directory, one per major data structure.

**How constraints are derived from fake data:**

| Observation | Schema Decision |
|-------------|----------------|
| Longest title observed: 41 chars | `maxLength: 80` (real + buffer) |
| Many todos have no memo | `memo` is optional |
| todoId is always 26 chars | `minLength: 26, maxLength: 26` |
| isCompleted is always boolean | `type: boolean` |

**Tips:**
- `maxLength` should be observed-max × ~2, not a round default like 255
- Mark fields as `required` only if the fake data shows they're always present
- Use ALPS descriptor IDs as property names for consistency

**Example:** See [`example/schema/`](../../example/schema/)

---

## Step 6: Be Implementation

**Question:** How do we code this domain in Be Framework?

**Input:** The schemas from Step 5 become your `Being` definitions.

**Structure:**
```
src/
├── Being/
│   ├── TodoList.php      ← maps to schema/todo-list.json
│   └── TodoDetail.php    ← maps to schema/todo-detail.json
├── Final/
│   ├── CreateTodo.php    ← doCreateTodo
│   ├── CompleteTodo.php  ← doCompleteTodo
│   └── DeleteTodo.php    ← doDeleteTodo
└── Reason/
    ├── TodoRepository.php
    └── Todo.php          ← entity
```

**The connection:**
- ALPS `descriptor` ids → Being property names
- ALPS `safe` descriptors → Being (read)
- ALPS `unsafe/idempotent` descriptors → Final (write)
- JSON Schema constraints → Being validation rules

The code reads like the domain because the domain was designed first.

---

## The Full Loop

When you complete all six steps, you have:

1. A **story** that any stakeholder can read and confirm
2. An **ALPS profile** that documents the application's vocabulary
3. **Fake data** that makes the domain tangible and reviewable
4. **Agreement** that everyone shares the same understanding
5. A **schema** that records observed reality as constraints
6. **Implementation** that mirrors the domain

The output of each step feeds the next. The story shapes the ALPS. The ALPS shapes the fake data. The fake data informs the schema. The schema guides implementation.

No step is wasted. No step can be skipped.
