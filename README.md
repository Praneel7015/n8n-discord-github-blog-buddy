<img width="1919" height="1067" alt="flow_b36b3d8336" src="https://github.com/user-attachments/assets/f1f8aada-dd91-496e-b13a-2e02d2ba2b71" /># Discord → GitHub Blog Buddy (n8n workflow)

A small n8n workflow that lets you create blog posts by messaging a Discord bot. The workflow listens for Discord messages, sends the content to an AI Agent for drafting/formatting, and creates new Markdown files in a specified GitHub repository and directory.

![Workflow preview](https://n8niostorageaccount.blob.core.windows.net/n8nio-strapi-blobs-prod/assets/flow_b36b3d8336.png)  
*(Add your exported workflow image here — recommended for clarity)*  

> ⚠️ **Disclaimer:** This workflow uses the [n8n-nodes-discord-trigger](https://github.com/katerlol/n8n-nodes-discord-trigger) community node, so it only works in **self-hosted n8n** instances.

---

### Why I made this

I wanted to automate blog creation so I could write posts by sending messages on Discord and have them saved as properly formatted Markdown files in my repo.

---

## How it works

- Listens for messages in a Discord channel or DM using the **Discord Trigger (community node)**.  
- Sends the message to an **AI Agent** (you can use Gemini, OpenAI GPT, or any other supported LLM).  
- Uses GitHub nodes to list files, read files, and create new `.md` posts in your target repository.  
- Uses the **Date & Time** node to stamp posts with the correct date.  
- Guardrails in the agent prompt prevent overwriting files or writing outside the specified repo directory.  
- Replies back to Discord with the generated output or status messages.  

---

## BlogAutomationclean.json

Key nodes included in the workflow:

- `Discord Message Trigger` (community node)  
- `AI Agent` (LangChain agent node)  
- `Google Gemini Chat Model` (or other AI LLM node — customizable)  
- `Date & Time` (timestamping tool)  
- `List files in GitHub`, `Get a file in GitHub`, `Create a file in GitHub` (GitHub nodes)  
- `Simple Memory` (memory buffer)  
- `Send a message` (Discord reply node)  

See `BlogAutomationclean.json` for the full node configuration and connections.

---

## How to set up

1. Install n8n and import this workflow (`BlogAutomationclean.json`).  
2. Install the [n8n-nodes-discord-trigger](https://github.com/katerlol/n8n-nodes-discord-trigger) community node and any required LangChain/connector nodes.  
3. Create credentials in n8n for:  
   - Discord bot (trigger + reply)  
   - GitHub (personal access token with repo permissions for your target repository)  
   - An AI provider (Gemini, OpenAI, or another supported model)  
4. Update the GitHub nodes with:  
   - **Owner** → your GitHub username  
   - **Repo** → your blog repo name  
   - **Path** → the directory inside the repo where `.md` posts should be created  
5. Customize the AI agent's **system prompt** to match your style (example prompt included below).  
6. Test the workflow in a private Discord channel or DM before going live.  

---

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
---

## How to customize

- Swap the AI model node for any provider (Gemini, OpenAI, local LLM, etc.).  
- Edit the prompt to enforce your blog’s style guide.  
- Add new workflow steps (Slack notifications, auto-publish, Notion sync).  
- Change the directory path or file naming scheme to fit your repo.  
```
---

## Contributions

- Feel free to contribute or open issues.  
- Third-party nodes and tools belong to their respective owners.  
- Shout-out to [katerlol](https://github.com/katerlol) for the [n8n-nodes-discord-trigger](https://github.com/katerlol/n8n-nodes-discord-trigger) node.  
- Contact: open an issue or PR in this repository if you need help.  

