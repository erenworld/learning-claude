## Claude learning

#### The context window is the most important resource to manage. 
- When the context window is getting full, Claude may start “forgetting” earlier instructions or making more mistakes.

#### Give Claude a way to verify its work 
A test, a build, a screenshot to compare. It’s the difference between a session you watch and one you walk away from.

- **Provide verification criteria** - "write a validateEmail function. example test cases: user@example.com is true, invalid is false, user@.com is false. run the tests after implementing”
- **Verify UI changes visually** - "[paste screenshot] implement this design. take a screenshot of the result and compare it to the original. list differences and fix them”
- **Address root causes, not symptoms** - "the build fails with this error: [paste error]. fix it and verify the build succeeds. address the root cause, don’t suppress the error”

- In one prompt: ask Claude to run the check and iterate in the same message, as in the table above.
- Across a session: set the check as a /goal condition. A separate evaluator re-checks it after every turn and Claude keeps working until it holds.
- As a deterministic gate: a Stop hook runs your check as a script and blocks the turn from ending until it passes. Claude Code overrides the hook and ends the turn after 8 consecutive blocks.
- By a second opinion: a verification subagent or a dynamic workflow that checks its own findings has a fresh model try to refute the result, so the agent doing the work isn’t the one grading it.

#### Explore first, then plan, then code

- Separate research and planning from implementation to avoid solving the wrong problem.
- Use plan mode to separate exploration from execution.

##### Example 

1. Explore 

Enter plan mode by pressing Shift+Tab until the status bar shows ⏸ plan mode on, or start the session with claude --permission-mode plan. Claude reads files and answers questions without making changes.

```
read /src/auth and understand how we handle sessions and login.
also look at how we manage environment variables for secrets.
```

2. Plan 

Ask Claude to create a detailed implementation plan.

```
I want to add Google OAuth. What files need to change?
What's the session flow? Create a plan.
```

- Press `Ctrl+G` to open the plan in your text editor for direct editing before Claude proceeds.

3. Implement 

Switch out of plan mode by approving the plan or pressing Shift+Tab, then let Claude code, verifying against its plan.

```
implement the OAuth flow from your plan. write tests for the
callback handler, run the test suite and fix any failures.
```

4. Commit 

commit with a descriptive message and open a PR


#### Plan mode is useful, but also adds overhead.

For tasks where the scope is clear and the fix is small (like fixing a typo, adding a log line, or renaming a variable) ask Claude to do it directly.

Planning is most useful when you’re uncertain about the approach, when the change modifies multiple files, or when you’re unfamiliar with the code being modified. If you could describe the diff in one sentence, skip the plan.

#### Provide specific context in your prompts

- The more precise your instructions, the fewer corrections you’ll need.
- Reference specific files, mention constraints, and point to example patterns.

##### Scope the task. Specify which file, what scenario, and testing preferences.

- "write a test for foo.py covering the edge case where the user is logged out. avoid mocks.”

##### Point to sources. Direct Claude to the source that can answer a question.

- "look through ExecutionFactory’s git history and summarize how its api came to be”

##### Reference existing patterns. Point Claude to patterns in your codebase.

- "look at how existing widgets are implemented on the home page to understand the patterns. HotDogWidget.php is a good example. follow the pattern to implement a new calendar widget that lets the user select a month and paginate forwards/backwards to pick a year. build from scratch without libraries other than the ones already used in the codebase.”

##### Describe the symptom. Provide the symptom, the likely location, and what “fixed” looks like.

- "users report that login fails after session timeout. check the auth flow in src/auth/, especially token refresh. write a failing test that reproduces the issue, then fix it”
- Vague prompts can be useful when you’re exploring and can afford to course-correct. A prompt like "what would you improve in this file?" can surface things you wouldn’t have thought to ask about.
