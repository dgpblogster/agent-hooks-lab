# Post tool use: audit row mapping

The audit hook is a **Post tool use** hook with **Tools** left empty, so it runs after every
successful tool call. Its workflow returns nothing; it only writes one row per call to a
Dataverse table or a SharePoint list.

| Audit row column | Hook input |
|---|---|
| Timestamp | `timestamp` |
| Agent | `metadata.agentName` |
| User (Entra ID) | `metadata.userAadObjectId` |
| Channel | `metadata.channel` |
| Conversation | `metadata.conversationId` |
| Tool | `toolName` |
| Arguments | `parameters` |

## Two details that matter

**Identify the person by `userAadObjectId`, not `userId`.** `userId` is the user's ID in the
channel they are chatting on, such as Teams. `userAadObjectId` is their Microsoft Entra object
ID, which is the identity you want in an audit record.

**Give every column a fallback.** The docs say any metadata field can be empty, and the user
fields always are when the person chatting isn't signed in. Write a placeholder such as
`(not signed in)` rather than letting the write fail. A failed hook does not stop the agent,
so a failed audit write shows up only as a gap in the table.

## Storing `parameters`

The docs do not document the internal shape of `parameters` for a given tool. Store it as text
(a multiline text column, or a SharePoint multiple lines of text column) and read a few real
rows before you build anything that parses it.
