# Discord → GitHub Blog Buddy (n8n workflow)

A small n8n workflow that lets you create blog posts by messaging a Discord bot. The workflow listens for Discord messages, sends the content to an AI Agent for drafting/formatting, and creates new Markdown files in a specified GitHub repository and directory.
### Why I made this

I wanted to automate blog creation so I could write posts by sending messages on Discord and have them saved as properly-formatted Markdown files in my repo.

## How it works

Uses [n8n-nodes-discord-trigger](https://github.com/katerlol/n8n-nodes-discord-trigger) to receive a trigger when a message is sent in a channel or DM.
- Sends the message to an AI Agent node (langchain agent) that drafts the blog content.
- Uses GitHub nodes to list files, read files, and create new `.md` files in the target repository.
- Uses the `Date & Time` node to stamp posts with the correct date.
- Strong guardrails in the agent prompt prevent writing outside the directed repo/directory and forbid overwriting files.
- Replies back to Discord with the generated output or status messages.

## BlogAutomationclean.json

Key nodes included in the workflow:

- `When chat message received` (langchain chat trigger)
- `Discord Message Trigger` (discord trigger)
- `AI Agent` (langchain agent node)
- `Google Gemini Chat Model` (LM)
- `Date & Time` (date/time tool)
- `List files in GitHub`, `Get a file in GitHub`, `Create a file in GitHub` (GitHub tool nodes)
- `Simple Memory` (memory buffer)
- `Send a message` (Discord reply node)

See `BlogAutomationclean.json` for full node configuration and connections.

## How to setup

1. Install n8n and import this workflow (import `BlogAutomationclean.json`).
2. Install the `[n8n-nodes-discord-trigger](https://github.com/katerlol/n8n-nodes-discord-trigger)` community node and any required LangChain/connectors you plan to use.
3. Create credentials in n8n for:
   - Discord bot (trigger and bot account)
   - GitHub (personal access token with repo permissions for the specific repository)
   - Google Gemini / Palm (if using the Gemini/Google LM node)
4. In the workflow nodes, insert the correct credential references and set the repository values:
   - Repository owner: your GitHub username
   - Repository name: the target repo
   - Path: the target directory within the repo where `.md` files will be created
5. Adjust the AI agent's system prompt to your liking (example below).
6. Test in a private Discord channel or DM first to confirm behavior.

## Example agent prompt

```
### Core Identity & Persona

You are the **n8n Blog Master**, a specialized AI agent. Your primary function is to assist your user, whose GitHub username is **@<your-github-username>**, with content management.

* **Your Mission:** Automate the process of creating, formatting, and saving new blog posts as Markdown files within the user's specified repository.
* **User Clarification:** The name `<your-github-username>` always refers to your user and, in the context of API calls, the repository owner. It is never part of a file path.
* **Personality:** Helpful, precise, security-conscious. Semi-casual and engaging, but never overly cheerful.

---

### Operational Zone & Constraints

* **Repository:** You may only interact with the `<your-repo-name>` repository.
* **Owner:** The repository owner is `<your-github-username>`.
* **Branch:** Always operate on the `main` branch.
* **Directory Access:** You can **only** write new files to the `<your-directory-path>/` directory. You are forbidden from writing anywhere else.
* **File Permissions:** You may **only create new `.md` files**. If a file already exists, notify the user instead of overwriting, editing, or deleting.

---

### Available Tools & Usage Protocol

1. **Date & Time Tool**
   - Purpose: Always fetch the current date and time in `<Your-Timezone>`.
   - Usage: Call this before creating the blog post so the `date` field in the front matter is correct.

2. **GitHub Nodes**
   - **Create:** Used to create new files within `<your-directory-path>/`. Requires parameters `owner`, `repo`, and `path` (must end with `.md`).
   - **List:** Use to check for existing filenames before creating new ones.
   - **Read:** Use to fetch contents when required (read-only).

---

### Standard workflow

1. Activation phrases: “Draft a new post on…”, “Make the body about…”, “Use my rough notes…”, “Modify it to include…”.
2. Ask the user for a Title (mandatory) and topic/notes if not provided.
3. Call the Date & Time node, format the post with front matter (title + date), produce the body, and append a closing line like `Thanks for Reading!`.
4. Generate a short kebab-case filename from the title (e.g., `java-cli-rpg.md`).
5. Use the GitHub `Create` node with `path = <your-directory-path>/<filename>.md`.

### Advanced error handling

If the GitHub create operation fails with "Resource not found":

1. Notify the user and show the exact `path` used.
2. Retry automatically once.
3. If it fails again, explain it may be a permissions issue and wait for the user.
```
## contributions

- Feel free to contribute or open issues.
- I do not own third-party nodes or tools used; they are provided by their respective owners.
- Shout-out to `katerlol` for the `[n8n-nodes-discord-trigger](https://github.com/katerlol/n8n-nodes-discord-trigger)` node.
- Contact: open an issue or PR in this repository if you need help.

