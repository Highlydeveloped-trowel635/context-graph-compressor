# context-graph-compressor

A Claude skill that compresses any chat conversation into a minimal, portable JSON context graph — so you can continue the same conversation in a new chat or a different LLM at a fraction of the token cost.

---

## The Problem

Every message in a long chat re-sends the entire history:

```
Turn 50:  [turn1 + turn2 + ... + turn49 + your message]  →  ~15,000 tokens
```

## The Solution

Compress the chat once, paste the output into a new session:

```
New chat turn 1:  [compressed graph ~400 tokens + your message]  →  done
```

Same context. 97% fewer tokens.

---

## Why This Is Different

Several context-handling solutions already exist. Here's how this one compares:

| | This skill | Claude's built-in compaction | Handoff skills (Smithery / ClaudSkills) |
|---|---|---|---|
| **Works across sessions** | ✅ Export & paste anywhere | ❌ Same chat only | ✅ |
| **Works with other LLMs** | ✅ GPT, Gemini, Mistral, etc. | ❌ Claude only | ❌ Claude Code only |
| **Output format** | JSON graph | Internal (not exportable) | Plain text / markdown bullets |
| **Machine-readable** | ✅ Structured, parseable | ❌ | ❌ |
| **Importance tagging** | ✅ `h/m/l` per node | ❌ | ❌ |
| **Status tracking** | ✅ open/resolved/deferred/blocked | ❌ | ❌ |
| **Graph relationships** | ✅ depends_on, resolves, supersedes... | ❌ | ❌ |
| **Assumptions explicit** | ✅ `A` node type with confidence | ❌ | ❌ |
| **Downloadable file** | ✅ `.json` | ❌ | ❌ |
| **Token target** | ~400–800 tokens | Varies, not optimized | Varies |

### In plain terms

**Claude's built-in compaction** kicks in automatically when a chat gets too long — but it works silently inside the same conversation. You can't export it, restart clean, or take it to another LLM. It's a life raft, not a handoff.

**Existing handoff skills** produce a markdown bullet list — readable by humans, but not structured enough for a machine. There's no importance ranking, no node graph, no way to programmatically filter what a new LLM should prioritize. And they're all built for Claude Code, not Claude.ai.

**This skill** treats context as *data*, not prose. The JSON graph is structured so any LLM can parse it, importance tags tell the receiving model what it must know vs. what's optional, and compact mode is explicitly optimized to hit the lowest possible token count for the handoff — not just "shorter than the original."

---

## What It Outputs

A structured JSON graph with two modes:

### `compact` (default) — for new chat handoff
~300–500 tokens. Abbreviated keys, telegraphic summaries, no fluff.

```json
{
  "v": 1,
  "mode": "compact",
  "desc": "Building a JWT-secured Spring Boot API with MySQL.",
  "n": [
    {
      "id": "n1",
      "t": "F",
      "i": "h",
      "s": "Spring Boot 3.2, Java 21, MySQL. Maven. Constructor injection throughout.",
      "c": [
        { "id": "n1.1", "t": "F", "i": "m", "s": "DB: app_db. ddl-auto=update. Config in application.yml." }
      ]
    },
    {
      "id": "n2",
      "t": "P",
      "i": "h",
      "s": "403 on all endpoints fixed: added .requestMatchers('/auth/**').permitAll() before anyRequest().authenticated()."
    }
  ],
  "handoff": "Continue a Spring Boot JWT API project. Stack: Java 21, MySQL, JPA. Auth working (403 fixed). N+1 query fixed with @EntityGraph. Refresh tokens deferred."
}
```

### `readable` — for humans / archiving
Full field names, prose summaries, complete handoff prompt. Good for sharing with teammates or saving a project's history.

---

## How to Use

### Step 1 — Compress your chat
In any Claude chat, say:
```
compress this chat
```
or
```
save context / handoff to new chat / too many tokens, start fresh
```

The skill outputs the compact JSON and saves a downloadable `.json` file.

### Step 2 — Paste into new chat
Start a new chat and paste:
```
Context from previous session:
<paste JSON here>

Continue from this state.
```

That's it. The new LLM picks up exactly where you left off.

---

## Node Types

| Short | Full | Meaning |
|---|---|---|
| `F` | fact | Technical facts, stack, config, versions |
| `D` | decision | Choices made, things ruled out |
| `P` | problem | Bugs, errors, and their fixes |
| `G` | goal | What the user is building / trying to achieve |
| `C` | code | Snippets, method names, annotations |
| `X` | context | Open/unresolved threads |

## Importance Levels

| Short | Meaning |
|---|---|
| `h` | High — losing this breaks continuity |
| `m` | Medium — useful but not critical |
| `l` | Low — dropped in compact mode |

---

## Installation

1. Download `context-graph-compressor.skill`
2. Go to **Claude.ai → Settings → Skills**
3. Upload the `.skill` file
4. Done — Claude will use it automatically when you ask to compress a chat

---

## Works With

- Claude (claude.ai)
- Any LLM that accepts a system/user prompt — GPT-4, Gemini, Mistral, etc.
- Just paste the JSON + `"Continue from this state."` as your first message

---

## Files

```
context-graph-compressor/
├── SKILL.md       — Core skill instructions for Claude
└── README.md      — This file
```

---

## License

MIT — free to use, modify, and share.
