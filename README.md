# Agent Hooks Lab

## A Companion Lab for The Workbench

The specification and test harness around **Copilot Studio hooks (preview)**: workflows that
run every time a lifecycle event fires in an agent powered by the GitHub Copilot harness,
including the one event that can block a tool call.

Companion to the post *The Workflow That Runs Every Time* on
[The Workbench](https://www.theworkbench.blog).

Source of truth: [Hooks (GitHub Copilot) (preview)](https://learn.microsoft.com/microsoft-copilot-studio/agents-experience/hooks-overview).
Hooks are a preview feature, and agents on the GitHub Copilot harness consume Copilot Credits.

---

## Why a Repo for Something That Lives in the Environment

Hook workflows are built in the Copilot Studio workflow designer and live in your environment.
What does not live anywhere convenient is the contract: which inputs each event sends, which
outputs it reads back, which event can block, and what the test cases are. That is what this
repo holds.

---

## Event Reference

Every event also sends `event`, `timestamp` and a `metadata` object
([`contracts/_common.json`](contracts/_common.json)). All outputs are optional; an empty output
means the agent carries on with its default behavior.

| Event type | Runs when | Adds inputs | Can return | Can block |
|---|---|---|---|---|
| **Start (pre-loop)** | A new conversation begins | none | `additionalContext` | No |
| **User prompt submitted** | Before each user message is processed | `prompt` | `modifiedPrompt`, `suppressOutput` | No |
| **Error** | An error outside a tool call | `error`, `errorContext`, `recoverable` | `errorHandling` (`retry` / `skip` / `abort`), `retryCount`, `userNotification`, `suppressOutput` | No |
| **Pre tool use** | Before a tool call | `toolName`, `parameters` | `permissionDecision` (`deny`), `permissionDecisionReason`, `modifiedParameters`, `additionalContext`, `suppressOutput` | **Yes** |
| **Post tool use** | After a tool succeeds | `toolName`, `parameters`, `result` | `modifiedResult`, `additionalContext`, `suppressOutput` | No |
| **After tool failure** | A tool returns an error | `toolName`, `parameters`, `error` | `additionalContext` | No |

The three tool events can be scoped to specific tools in the hook's **Tools** field, or left
empty to apply to **All tools**.

---

## Repo Structure

```
agent-hooks-lab/
├── README.md                              # This file
├── contracts/                             # One file per event, field names exactly as documented
│   ├── _common.json                       # event, timestamp, metadata (and its eight fields)
│   ├── start.json
│   ├── user-prompt-submitted.json
│   ├── error.json
│   ├── pre-tool-use.json
│   ├── post-tool-use.json
│   └── after-tool-failure.json
├── payloads/
│   ├── pre-tool-use-deny.json             # The deny response from the post
│   └── post-tool-use-audit-mapping.md     # Hook inputs to audit-row columns
└── tests/
    └── test-plan.md                       # Three prompts with expected outcomes
```

---

## Things That Will Bite You

From the docs, in the order you are likely to hit them:

- **Hooks are on the agent's command bar, not the Build tab.** Three dots (**...**) on the top
  menu bar > **Hooks**.
- **Keep the starter workflow's field names.** Copilot Studio matches workflows to events by
  their input and output names; rename one and the workflow disappears from the list.
- **Publish the workflow, save the agent, publish the agent.** A saved but unpublished workflow
  doesn't run, and selecting **Done** only adds the hook to the draft agent.
- **Hooks fail open.** If the workflow fails, times out or returns something unreadable, the
  agent continues as though the hook returned nothing. The docs advise against relying on a hook
  as the only safeguard for a business-critical rule.
- **Inputs are untrusted.** Prompts, tool results and error messages can carry content the
  agent didn't produce.
- **Metadata can be empty.** The user fields always are when nobody is signed in.
- **Editing a workflow changes every hook that uses it.** A hook stores a reference, not a copy.

---

## Not Here (Yet)

**An exported solution with the agent and both workflows.** It goes in once it has been built
and run in a real environment, not before. Until then, the contracts and the test plan are
enough to build both hooks by hand in a few minutes.

**The shape of `parameters` for a given tool** is not documented. Run the agent once against a
harmless input and read the workflow's run before you write conditions against it.
