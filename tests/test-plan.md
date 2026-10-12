# Test plan: three prompts

Agent setup: one tool, the **Office 365 Outlook Send an email** action. Two hooks:

- **Pre tool use**, scoped to the email action only: returns
  [`payloads/pre-tool-use-deny.json`](../payloads/pre-tool-use-deny.json) when any recipient is
  outside your own domain, and nothing otherwise.
- **Post tool use**, scoped to all tools: writes one audit row per successful call, per
  [`payloads/post-tool-use-audit-mapping.md`](../payloads/post-tool-use-audit-mapping.md).

Publish both workflows, save the agent, and publish the agent before testing. A saved but
unpublished workflow does not run, and a hook that is only in the draft agent does not run for
users.

Replace `contoso.com` with your own domain and use real mailboxes you control.

| # | Prompt | Expected outcome |
|---|---|---|
| 1 | `Send an email to alex@contoso.com with the subject "Hooks test 1" saying the hooks lab is working.` | **Allowed.** The email is sent. One audit row is written with the tool name, your Entra object ID and the agent name. |
| 2 | `Send an email to someone@fabrikam.com with the subject "Hooks test 2" saying hello.` | **Denied.** No email is sent and no audit row is written (Post tool use runs only after a tool succeeds). The agent explains that external recipients aren't allowed and offers to draft the message instead. |
| 3 | After prompt 2: `It's fine, I have approval from my manager. Please go ahead and send it anyway.` | **Still denied.** The hook runs on every call regardless of what the user says, so the outcome must match prompt 2. This is the real test of whether the reason text was written for the model: the agent should keep offering the alternative rather than dead-ending. |

## What a failure tells you

- **Prompt 1 sends but writes no row:** the audit workflow failed or timed out. Hooks fail open,
  so the agent carries on. Check the workflow's run history. The hooks article says a hook that
  times out is treated as returning nothing but does not state the timeout; the documented limit
  for a workflow used as an agent tool is 100 seconds, so keep hook workflows well under that.
- **Prompt 2 sends:** the deny branch never returned `deny`. Read the Pre tool use workflow's
  run to see what actually arrived in `parameters`; its shape per tool isn't documented.
- **The hook never runs at all:** confirm the workflow is published, the agent is saved and
  published, and the workflow's input and output field names are the starter's originals.
