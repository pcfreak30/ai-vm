# AI VM - AI Program Virtual Machine

A lightweight virtual machine for executing AI Program protocols. This project provides a protocol interpreter and a comprehensive example skill demonstrating the system's capabilities.

## Prerequisites

This project is built for **[pi](https://github.com/mariozechner/pi)** — the AI coding agent. Before using this:

1. **Install pi** — Follow the installation at [pi.dev](https://pi.dev)
2. **Ensure pi agents are installed** — The pi agents example (worker, scout, planner, reviewer) must be present in `~/.pi/agent/agents/`
   - These agents are required for the `SPAWN` and `PARALLEL` operations in the VM
   - If missing, install the agents from the pi examples:
   ```bash
   # Copy agents to your pi directory (if available from pi examples)
   mkdir -p ~/.pi/agent/agents
   # Follow pi documentation to install the agents bundle
   ```

## Components

### ai-program-interpreter (The VM)

The core protocol interpreter that enables immediate execution of AI Program protocols. When loaded, it transforms the AI agent into an execution engine that:

- **Executes protocols** - Runs SKILL protocols step-by-step without explanation
- **Handles variables** - Manages state binding and transformations
- **Control flow** - Supports IF/THEN, FOR loops, and GOTO
- **Parallel execution** - Uses subagent delegations for true concurrency
- **Tool invocation** - Calls harness tools (read, write, edit, bash, etc.)
- **Recursive composition** - FOLLOW for sequential recursion, SPAWN for isolated delegation

### hello-world

An advanced demonstration skill that stress-tests the VM with:

- **Nested tree structures** - Recursive tree building with configurable depth and branch factor
- **Mixed dispatch** - Both sequential and parallel execution modes
- **Multi-level aggregation** - Results aggregated across depth levels
- **Complex patterns** - Demonstrates composition, isolation, and true concurrency

## Installation

To use these skills with pi, copy them to your pi skills directory:

```bash
# Copy skills to pi's skill directory
mkdir -p ~/.pi/agent/skills
cp -r ai-program-interpreter ~/.pi/agent/skills/
cp -r hello-world ~/.pi/agent/skills/
```

The `/skill` command in pi will automatically discover skills from `~/.pi/agent/skills/`.

## Usage

The AI VM skills are designed for use with the **pi coding agent**. You must use the `/skill` command from within the pi CLI to load these skills.

### Getting Started

1. **Open pi**
   ```bash
   pi
   ```

2. **Load the AI Program Interpreter skill** — This enables protocol execution mode
   ```
   /skill ai-program-interpreter
   ```

3. **Execute hello-world** — Test the system
   ```
   Execute hello-world WITH name = "Alice", output_path = "greeting.txt"
   ```

> **Important:** Always load `ai-program-interpreter` first using `/skill ai-program-interpreter`. The interpreter transforms pi into an execution engine that understands the AI Program protocol syntax.

### Advanced Tree Generation (inside pi)

```
# Generate a nested tree of greetings
Execute hello-world WITH nested_mode = true, depth = 3, branch_factor = 3, parallel = true
```

### Multi-Mode Processing (inside pi)

```
# Process multiple names
Execute hello-world WITH names = ["Alice", "Bob", "Charlie"], output_dir = "./greetings", parallel = true, depth = 2
```

## Protocol Syntax

The VM understands the following commands:

| Syntax | Action |
|--------|--------|
| `SKILL name WITH params` | Entry point with inputs |
| `SET x = value` | Bind value to variable |
| `IF condition THEN ... END IF` | Conditional execution |
| `FOR item IN list ... NEXT item` | Iterate through items |
| `GOTO label` | Jump to label |
| `PARALLEL ... END PARALLEL` | Concurrent execution via subagent |
| `CALL tool "name" WITH ...` | Invoke harness tool |
| `FOLLOW skill "name" WITH ...` | Recursive execution (same context) |
| `SPAWN skill "name" WITH ...` | Isolated execution (fresh context) |
| `RETURN key = value` | Complete with result |

## About

This project is based on and designed for **[pi](https://pi.dev)** — the AI coding agent. The protocol interpreter and skills demonstrate pi's skill system and agent delegation capabilities.

### Related pi Concepts

- **Skills** — Modular capabilities loaded via `/skill` command ([docs/skills.md](https://github.com/mariozechner/pi/blob/main/docs/skills.md))
- **Agents** — Specialized sub-contexts for task delegation (worker, scout, planner, reviewer)
- **Subagent tool** — Enables parallel task execution and isolated contexts

For more information on pi's capabilities, visit [pi.dev](https://pi.dev) or explore the [pi documentation](https://github.com/mariozechner/pi#readme).

## License

MIT License - See [LICENSE](LICENSE) file
