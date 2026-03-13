# Tutorial: Build a Be App from Scratch

This tutorial walks through the be-semantic workflow to build a Todo app from zero.
The `example/` directory shows the finished result.

## Prerequisites

- PHP 8.3+
- [Composer](https://getcomposer.org/)
- [asd (app-state-diagram)](https://github.com/alps-asd/app-state-diagram)

### Install asd

```bash
# Homebrew (recommended)
brew install alps-asd/asd/asd

# Or with Composer
composer global require koriym/app-state-diagram
```

Verify installation:
```bash
asd --version
```

---

## Step 1: Write User Stories

Create your project directory and start with user stories.

```bash
mkdir my-todo
cd my-todo
mkdir -p story alps fake schema src/{Being,Final,Reason}
```

Create `story/main.md`:

```markdown
# App Overview
A simple Todo app. Create, manage, and complete tasks.

## User Stories

### View task list
As a user,
I want to see all my todos in a list,
because I need to know what I should be doing.

### Create a task
As a user,
I want to create a new task with a title and optional note,
because I need to record things I have to do.

...（see example/story/main.md for the full list）

## Entities
- **Todo**: id, title, memo (optional), isCompleted, createdAt
```

**Checkpoint:** Does each story have a "because" clause? Are entities and attributes clear?

---

## Step 2: Create an ALPS Profile

Use your stories to define the application's vocabulary and state transitions.

Create `alps/todo.json` (see `example/alps/todo.json` for the full version).

Basic ALPS structure:
```json
{
  "$schema": "https://alps-io.github.io/schemas/alps.json",
  "alps": {
    "title": "Todo App",
    "descriptor": [
      // Semantic descriptors (data elements)
      {"id": "todoId", "title": "Todo ID"},
      {"id": "todoTitle", "title": "Title"},

      // Representations (compound views)
      {"id": "TodoList", "descriptor": [...]},

      // Operations (transitions)
      {"id": "goTodoList", "type": "safe", "rt": "#TodoList"},
      {"id": "doCreateTodo", "type": "unsafe", "rt": "#TodoList"}
    ]
  }
}
```

### Visualize the state diagram

```bash
# Validate and generate HTML
asd alps/todo.json

# Open in browser
open alps/todo.html
```

The state diagram shows the full application at a glance. Use it directly for stakeholder discussions.

**Checkpoint:** Are all story actions reflected in the ALPS profile?

---

## Step 3: Generate Fake Data

With your ALPS profile in hand, ask an AI to generate 50 realistic fake entries.

**Example prompt:**
```
Based on the following ALPS profile, generate 50 realistic fake Todo items.
Make them feel like real data from real users in daily life.

[paste alps/todo.json here]

Output format: JSON array
```

Save the output to `fake/data-50.json`.

You can also reference `example/fake/data-50.json`.

**Checkpoint:** Does the data feel realistic? Not random test data, but what real users would actually create?

---

## Step 4: Observe and Reach Agreement

Study the 50 entries and discover constraints.

**What to observe:**

```bash
# Check title length distribution
cat fake/data-50.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
lengths = [len(d['todoTitle']) for d in data]
print(f'min: {min(lengths)}, max: {max(lengths)}, avg: {sum(lengths)/len(lengths):.1f}')
"

# Count null memos
cat fake/data-50.json | python3 -c "
import json, sys
data = json.load(sys.stdin)
null_count = sum(1 for d in data if d.get('todoMemo') is None)
print(f'memo=null: {null_count}/{len(data)} entries')
"
```

**Record your findings:**
```
Observations:
- todoTitle: max 41 chars observed → set maxLength: 80
- todoMemo: 35/50 entries are null → optional
- isCompleted: mix of true/false → boolean
- createdAt: always present → required
```

Review the data with stakeholders and reach explicit agreement.

---

## Step 5: Write the Schema

Translate observations and agreements into JSON Schema.

`schema/todo-list.json`:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["items"],
  "properties": {
    "items": {
      "type": "array",
      "items": {
        "required": ["todoId", "todoTitle", "isCompleted", "createdAt"],
        "properties": {
          "todoTitle": {
            "type": "string",
            "maxLength": 80
          }
        }
      }
    }
  }
}
```

See `example/schema/` for the complete schemas.

**Remember:** `maxLength: 80` because you *observed* it, not because VARCHAR defaults to 255.

---

## Step 6: Implement in Be Framework

### Project setup

```bash
composer init \
  --name="yourname/my-todo" \
  --require="be-framework/be:0.x-dev" \
  --require="ray/di:^2.18" \
  --require-dev="phpunit/phpunit:^10.0"

composer install
```

### Implement Beings (states)

Schema `required` → Being constructor arguments  
Schema `properties` → types and validation

```php
// src/Being/TodoList.php
namespace MyTodo\Being;

use Be\Framework\Being;

#[Being]
class TodoList
{
    public function __construct(
        /** @var TodoItem[] */
        public readonly array $items
    ) {}
}
```

See `be-framework-demos/demos/` for working implementation examples.

### Run tests

```bash
./vendor/bin/phpunit
```

---

## Completion Checklist

- [ ] `story/main.md` — All features as user stories (with "because" clauses)
- [ ] `alps/todo.json` — Validated ALPS profile
- [ ] `alps/todo.html` — State diagram generated by `asd`
- [ ] `fake/data-50.json` — 50 realistic fake entries
- [ ] Observation notes — Rationale for each constraint
- [ ] `schema/todo-list.json` — Schema derived from observations
- [ ] `schema/todo-detail.json` — Schema derived from observations
- [ ] `src/` — Be Framework implementation
- [ ] All tests passing

---

## References

- [Be Framework](https://be-framework.github.io/)
- [ALPS Specification](https://alps.io/)
- [app-state-diagram](https://www.app-state-diagram.com/)
- [be-framework-demos](https://github.com/be-framework/demos)
- [Semantic-Ex Method](https://koriym.github.io/blog/2025/08/10/semantic-method-en)
