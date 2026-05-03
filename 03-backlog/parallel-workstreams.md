# Parallel Workstreams

How to run multiple development streams simultaneously with Claude.

---

## Principles

1. **Independent contexts** - Each workstream needs its own Claude session
2. **Different repos** - Best when workstreams touch different codebases
3. **No conflicts** - Avoid parallel edits to the same files
4. **Coordinate merges** - Rebase/merge carefully when streams converge

---

## Current Parallel Opportunities

### Stream 1: BUILD-004 (Canonical Entity Drafts)

**Repo:** `bndy-signals`
**Focus:** Backend entity resolution

```
When: Accepted claim triggers entity check
Where: New Lambda or Step Functions state
Files:
  - functions/entity-resolver/index.ts (NEW)
  - functions/shared/entities/canonical-entity.ts (NEW)
  - infrastructure/cdk/lib/workflow-stack.ts (ADD state)
```

**Blocked by:** Nothing (unblocked)

### Stream 2: Chat UI (Optional)

**Repo:** `bndy-frontstage`
**Focus:** Conversational gig submission

```
When: User wants natural language input
Where: /chat or /dropzone/chat
Files:
  - app/chat/page.tsx (NEW)
  - app/chat/components/ChatInterface.tsx (NEW)
  - hooks/useChat.ts (NEW)
```

**Blocked by:** Nothing (unblocked)

### Stream 3: Deploy/Monitor

**Repo:** `bndy-signals` (read-only)
**Focus:** Deployment, testing, monitoring

```
Tasks:
  - Run `npx cdk deploy --all`
  - Test POST /signals with real data
  - Monitor CloudWatch logs
  - Check Step Functions executions
```

---

## How to Run Parallel Sessions

### Option A: Multiple Terminal Windows

```bash
# Terminal 1: BUILD-004
cd c:\VSProjects\florence\bndy-signals
claude

# Terminal 2: Chat UI
cd c:\VSProjects\bndy-frontstage
claude
```

Each session has independent context. Reference this doc at session start.

### Option B: VS Code Split Terminals

1. Open bndy-signals in one terminal
2. Split terminal (Ctrl+Shift+5)
3. cd to bndy-frontstage in the other
4. Run `claude` in each

### Option C: Background Deployment

```bash
# Start deploy in background
cd c:\VSProjects\florence\bndy-signals
npx cdk deploy --all &

# Continue development in foreground
claude
```

---

## Coordination Checklist

Before starting parallel streams:

- [ ] Identify repos touched by each stream
- [ ] Confirm no file overlap
- [ ] Pull latest from main in all repos
- [ ] Create feature branches if needed

After parallel work:

- [ ] Run tests in each repo
- [ ] Commit changes separately
- [ ] Push to feature branches
- [ ] Create PRs for review
- [ ] Merge in dependency order

---

## Example: Running BUILD-004 + Chat UI

### Session 1 (Terminal 1)
```
cd c:\VSProjects\florence\bndy-signals
claude

> "Continue BUILD-004: entity resolution. When a claim is accepted,
   check if entity exists, link or create draft."
```

### Session 2 (Terminal 2)
```
cd c:\VSProjects\bndy-frontstage
claude

> "Add a /chat page where users can describe gigs naturally.
   Parse intent and submit as signal."
```

Both sessions can run independently because:
- Different repos
- Different concerns (backend vs frontend)
- No shared files

---

## Anti-Patterns

### Don't: Same file from multiple sessions
```
Session 1: editing functions/interpretation-runner/index.ts
Session 2: editing functions/interpretation-runner/index.ts
Result: Merge conflicts, lost work
```

### Don't: Dependent changes without coordination
```
Session 1: Changes claim schema
Session 2: Uses old claim schema
Result: Runtime errors
```

### Do: Check dependencies first
```
Session 1: "I'm changing ClaimSchema in shared/entities"
Session 2: "I'll wait for that to merge before using claims"
```

---

## Current Status

| Stream | Status | Repo | Owner |
|--------|--------|------|-------|
| BUILD-004 Entity Drafts | Ready | bndy-signals | Unassigned |
| Chat UI | Ready | bndy-frontstage | Unassigned |
| Deploy/Monitor | Complete | bndy-signals | Complete |

---

## Related

- [[now]] - Current build priorities
- [[build-plan]] - Full technical breakdown
- [[../10-brain/bndy-brain-concept]] - The agentic model

