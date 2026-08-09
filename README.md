# Best Practices


## 1. The context window is your most important resource

As the context window fills up, Claude may start "forgetting" earlier instructions or making more mistakes. Manage it deliberately — don't let a session run indefinitely without a reset.

## 2. Give Claude a way to verify its work

A test, a build, a screenshot to compare — this is the difference between a session you have to babysit and one you can walk away from.

- **Provide verification criteria** — *"Write a `validateEmail` function. Example test cases: `user@example.com` is true, `invalid` is false, `user@.com` is false. Run the tests after implementing."*
- **Verify UI changes visually** — *"[paste screenshot] Implement this design. Take a screenshot of the result and compare it to the original. List differences and fix them."*
- **Address root causes, not symptoms** — *"The build fails with this error: [paste error]. Fix it and verify the build succeeds. Address the root cause, don't suppress the error."*

Verification can happen at different levels:
- **In one prompt** — ask Claude to run the check and iterate in the same message.
- **Across a session** — set the check as a `/goal` condition; a separate evaluator re-checks it after every turn and Claude keeps working until it holds.
- **As a deterministic gate** — a Stop hook runs your check as a script and blocks the turn from ending until it passes (Claude Code overrides the hook and ends the turn after 8 consecutive blocks).
- **By a second opinion** — a verification subagent, or a workflow that checks its own findings, has a fresh model try to refute the result — so the agent doing the work isn't the one grading it.


## 3. Explore first, then plan, then code

Separate research and planning from implementation to avoid solving the wrong problem. Use Plan mode to keep exploration and execution distinct.

### Example workflow

**1. Explore**
Enter Plan mode by pressing `Shift+Tab` until the status bar shows `⏸ plan mode on`, or start the session with `claude --permission-mode plan`. In this mode, Claude reads files and answers questions without making changes.

```
read /src/auth and understand how we handle sessions and login.
also look at how we manage environment variables for secrets.
```

**2. Plan**
Ask Claude to create a detailed implementation plan.

```
I want to add Google OAuth. What files need to change?
What's the session flow? Create a plan.
```

Press `Ctrl+G` to open the plan in your text editor for direct editing before Claude proceeds.

**3. Implement**
Switch out of Plan mode by approving the plan or pressing `Shift+Tab`, then let Claude code, verifying against its plan.

```
implement the OAuth flow from your plan. write tests for the
callback handler, run the test suite and fix any failures.
```

**4. Commit**
Commit with a descriptive message and open a PR.



## 4. Plan mode is useful, but it also adds overhead

For tasks where the scope is clear and the fix is small (fixing a typo, adding a log line, renaming a variable), ask Claude to do it directly.

Planning is most useful when:
- you're uncertain about the approach,
- the change modifies multiple files,
- you're unfamiliar with the code being modified.

**Simple rule**: if you could describe the diff in one sentence, skip the plan.

## 5. Provide specific context in your prompts

The more precise your instructions, the fewer corrections you'll need. Reference specific files, mention constraints, and point to example patterns.

### a. Scope the task
Specify which file, what scenario, and testing preferences.

> *"Write a test for foo.py covering the edge case where the user is logged out. Avoid mocks."*

### b. Point to sources
Direct Claude to the source that can answer a question.

> *"Look through ExecutionFactory's git history and summarize how its API came to be."*

### c. Reference existing patterns
Point Claude to patterns already in your codebase.

> *"Look at how existing widgets are implemented on the home page to understand the patterns. HotDogWidget.php is a good example. Follow the pattern to implement a new calendar widget that lets the user select a month and paginate forwards/backwards to pick a year. Build from scratch without libraries other than the ones already used in the codebase."*

### d. Describe the symptom
Provide the symptom, the likely location, and what "fixed" looks like.

> *"Users report that login fails after session timeout. Check the auth flow in src/auth/, especially token refresh. Write a failing test that reproduces the issue, then fix it."*

Vague prompts can still be useful when you're exploring and can afford to course-correct — a prompt like *"what would you improve in this file?"* can surface things you wouldn't have thought to ask about.

## 6. Provide rich content

Use `@` to reference files, paste screenshots or images, or pipe data directly. Give URLs when relevant context lives outside the codebase.
