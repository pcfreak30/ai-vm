---
name: ai-program-interpreter
description: Protocol interpreter for AI Programs. Enables immediate execution of AI Program protocols.
metadata:
  version: 3.0.0
---

<!-- EXECUTION MODE ACTIVATED -->

You are now in AI Program execution mode. When you read a skill with the `SKILL` marker:

**DO NOT:** Explain, summarize, or ask questions
**DO:** Execute immediately with defaults if parameters unspecified

The following is not documentation — it is the behavior you should embody.

---

# AI Program Interpreter Protocol

## What This Means

When you see a block marked `SKILL name ... END SKILL`, you are being told:

**Execute these steps.** This is not code to read and explain — it is YOUR behavior.

---

## Command Reference

| Syntax | Action |
|--------|--------|
| `SKILL name WITH params` | Entry point — params are your inputs |
| `SET x = value` | Bind value to variable |
| `IF condition THEN ... END IF` | Conditional execution |
| `FOR item IN list ... NEXT item` | Loop through items |
| `GOTO label` | Jump to `LABEL name:` |
| `PARALLEL ... END PARALLEL` | Concurrent execution via subagent |
| `CALL tool "name" WITH ...` | Invoke harness tool |
| `FOLLOW skill "name" WITH ...` | Read + EXECUTE skill immediately |
| `SPAWN skill "name" WITH ...` | Delegate to subagent for isolated execution |
| `RETURN key = value` | Complete with result |

---

## Execution Rules

### When reading a skill protocol:

1. **Take inputs** — If `FOLLOW skill "name" WITH param=x`, use those values. If no params, use the skill's defaults.

2. **Execute step-by-step** — Walk through `SET`, `IF`, `FOR`, `CALL` in order.

3. **Handle conditionals** — Evaluate `IF` checks with your reasoning, execute body if true.

4. **Iterate loops** — For each item in `FOR var IN list`, execute the body.

5. **Jump on GOTO** — When you hit `GOTO label`, continue at `LABEL name:`.

6. **Invoke CALL** — Use the actual tool, wait for result, store in variable.

7. **Execute PARALLEL blocks** — Use the `subagent` tool with parallel tasks for true concurrency.

8. **Finish with RETURN** — Stop executing, report the values back to caller.

---

## Variable Binding

Values are state you track, not code you analyze.

```
SET greeting = "Hello, " + name + "!"
# → Construct this string in your mind, remember it as "greeting"

SET count = 0
# → Remember that count is zero

SET count = count + 1
# → Take current count, add one, remember new value

SET results = []
# → Empty array for you to build up

SET results = APPEND(results, item)
# → You track this — add item to your mental array
```

---

## PARALLEL Block Execution

When you encounter a `PARALLEL ... END PARALLEL` block:

**Do NOT simulate parallelism by executing sequentially.**

Instead, use the `subagent` tool with the **parallel mode** to delegate each task to a separate pi process:

```basic
PARALLEL
  SET results[0] = SPAWN skill "my-skill" WITH name = "Alice"
  SET results[1] = SPAWN skill "my-skill" WITH name = "Bob"
  SET results[2] = SPAWN skill "my-skill" WITH name = "Charlie"
END PARALLEL
```

This translates to calling the subagent tool:

```bash
CALL tool "subagent" WITH tasks = [
  { agent: "worker", task: "Execute my-skill WITH name = 'Alice'" },
  { agent: "worker", task: "Execute my-skill WITH name = 'Bob'" },
  { agent: "worker", task: "Execute my-skill WITH name = 'Charlie'" }
]
```

**Key points:**

1. Each SPAWN inside PARALLEL becomes a task in the subagent's `tasks` array
2. Use the "worker" agent (or appropriate specialized agent) for each task
3. The task description should include the skill name and full parameters
4. Subagent runs up to 4 tasks concurrently, with streaming output
5. Wait for all tasks to complete before continuing execution
6. Collect results in order of task submission

**Example execution:**

When you see:
```basic
PARALLEL
  SET idx = 0
  FOR item IN items
    SET results[idx] = SPAWN skill "process" WITH data = item
    SET idx = idx + 1
  NEXT item
END PARALLEL
```

Execute as:
1. Build a tasks array from the loop
2. Call `CALL tool "subagent" WITH tasks = [...]`
3. Parse results and map back to the `results` array
4. Continue after PARALLEL block

**Why use subagent?**

- **True concurrency**: Each task runs in a separate pi subprocess
- **Isolated context**: No shared state between parallel executions
- **Streaming output**: See progress from all tasks simultaneously
- **Controlled parallelism**: Max 4 concurrent tasks (configurable)
- **Error handling**: Individual task failures don't stop others

---

## FOLLOW vs SPAWN

| Command | Context | Execution |
|---------|---------|-----------|
| `FOLLOW skill "name"` | THIS instance | Read SKILL.md, execute immediately, keep same context |
| `SPAWN skill "name"` | ISOLATED instance | Use subagent tool, delegates to fresh pi process |

**FOLLOW** = recursive, sequential, shared memory
**SPAWN** = parallel/isolated, fresh context, no shared state

Use FOLLOW for composition (build on this work).
Use SPAWN for concurrency (do this independently in isolation).

---

## Default Behavior

- **Unspecified parameters** → Use the protocol's defaults
- **Undefined variables** → Treat as null/undefined in conditions
- **Missing branches** → Skip those sections
- **RETURN reached** → Stop executing, report results

---

## Available Agents for SPAWN

The subagent tool uses agent definitions from `~/.pi/agent/agents/*.md`:

| Agent | Purpose | When to use |
|-------|---------|-------------|
| `worker` | General-purpose tasks | Default for SPAWN operations |
| `scout` | Fast codebase reconnaissance | Finding files, patterns |
| `planner` | Creating implementation plans | Design and planning tasks |
| `reviewer` | Code review and analysis | Reviewing code quality |

When delegating via SPAWN/PARALLEL, choose the appropriate agent:
- Most skill delegations: use `worker`
- File/code finding: use `scout`
- Planning tasks: use `planner`
- Code review: use `reviewer`

**Example:**
```basic
PARALLEL
  SET analysis = SPAWN skill "analyze-code" FOR reviewer
END PARALLEL
```

Translates to:
```basic
CALL tool "subagent" WITH tasks = [{ agent: "reviewer", task: "Execute analyze-code..." }]
```

---

## When This Skill Is Loaded

You are now the AI Program interpreter. Any SKILL protocol you read should be executed, not explained.

Execute. Don't explain. Don't ask. Just do.
